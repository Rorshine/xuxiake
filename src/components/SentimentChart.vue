<template>
  <div class="sentiment-chart-shell">
    <div v-if="showThumbnailWords" class="thumbnail-wordclouds">
      <div
        v-for="item in selectedSentimentItems"
        :key="item.chapter"
        class="thumbnail-chapter-block"
      >
        <div class="thumbnail-chapter-title" style="font-size: 16px; font-weight: 600;">{{ item.chapter }}</div>
        <div class="thumbnail-count-row">
          <span style="font-size: 16px; font-weight: 600;">积极词 {{ item.positiveCount }}</span>
          <span style="font-size: 16px; font-weight: 600;">消极词 {{ item.negativeCount }}</span>
        </div>
        <div class="thumbnail-cloud-row">
          <div class="thumbnail-cloud-panel">
            <div class="thumbnail-cloud-title" style="font-size: 16px; font-weight: 600;">积极情感词</div>
            <div :ref="setPositiveThumbRef(item.chapter)" class="cloud-chart"></div>
          </div>
          <div class="thumbnail-cloud-panel">
            <div class="thumbnail-cloud-title" style="font-size: 16px; font-weight: 600;">消极情感词</div>
            <div :ref="setNegativeThumbRef(item.chapter)" class="cloud-chart"></div>
          </div>
        </div>
      </div>
    </div>
    <template v-else>
      <div ref="barContainer" class="sentiment-chart-container" :class="{ 'detail-bar': isDetailMode, 'thumb-bar': !isDetailMode }"></div>
      <div v-if="isDetailMode" class="detail-wordclouds">
        <div v-if="selectedSentimentItems.length" class="detail-wordclouds-inner">
          <div
            v-for="item in selectedSentimentItems"
            :key="item.chapter"
            class="detail-chapter-block"
          >
            <div class="thumbnail-chapter-title" style="font-size: 16px; font-weight: 600;">{{ item.chapter }}</div>
            <div class="thumbnail-count-row">
              <span style="font-size: 16px; font-weight: 600;">积极词 {{ item.positiveCount }}</span>
              <span style="font-size: 16px; font-weight: 600;">消极词 {{ item.negativeCount }}</span>
            </div>
            <div class="thumbnail-cloud-row">
              <div class="thumbnail-cloud-panel">
                <div class="thumbnail-cloud-title" style="font-size: 16px; font-weight: 600;">积极情感词</div>
                <div :ref="setPositiveDetailRef(item.chapter)" class="cloud-chart"></div>
              </div>
              <div class="thumbnail-cloud-panel">
                <div class="thumbnail-cloud-title" style="font-size: 16px; font-weight: 600;">消极情感词</div>
                <div :ref="setNegativeDetailRef(item.chapter)" class="cloud-chart"></div>
              </div>
            </div>
          </div>
        </div>
        <div v-else class="detail-empty-tip" style="font-size: 16px; font-weight: 600;">选择章节后，可在此查看积极词与消极词词云</div>
      </div>
    </template>
  </div>
</template>

<script>
import * as echarts from "echarts";
import "echarts-wordcloud";
let barChart;
export default {
  name: "SentimentChart",
  props: {
    activeChapter: { type: Number, default: null },
    selectedChapters: { type: Array, default: () => [] },
    chapters: { type: Array, default: () => [] },
    isDetailMode: { type: Boolean, default: false },
    sentimentData: { type: Array, required: true },
  },
  data() {
    return {
      barChart: null,
      thumbPositiveRefs: {},
      thumbNegativeRefs: {},
      detailPositiveRefs: {},
      detailNegativeRefs: {},
      cloudCharts: new Map(),
      resizeHandler: null,
      wordCloudDataCache: {},
    };
  },
  computed: {
    showThumbnailWords() {
      return !this.isDetailMode && this.selectedChapters.length > 0;
    },
    selectedSentimentItems() {
      return this.selectedChapters
        .map(index => this.sentimentData[index])
        .filter(Boolean)
        .slice(0, 3);
    },
  },
  watch: {
    sentimentData: {
      handler() {
        this.refreshAll();
      },
      deep: true,
    },
    selectedChapters: {
      handler(newVal, oldVal) {
        const newList = Array.isArray(newVal) ? newVal : [];
        const oldList = Array.isArray(oldVal) ? oldVal : [];

        const added = newList.filter(index => !oldList.includes(index));

        added.forEach((index) => {
          const item = this.sentimentData[index];
          if (!item || !item.chapter) return;

          delete this.wordCloudDataCache[this.getCacheKey(item.chapter, "positive")];
          delete this.wordCloudDataCache[this.getCacheKey(item.chapter, "negative")];
        });

        this.refreshAll();
      },
      deep: true,
    },
    activeChapter() {
      this.renderBarChart();
    },
    isDetailMode() {
      setTimeout(()=>{
        this.refreshAll();
      },300)
    },
  },
  mounted() {
    this.refreshAll();
    this.resizeHandler = () => {
      if (barChart) barChart.resize();
      this.cloudCharts.forEach(chart => chart && chart.resize());
    };
    window.addEventListener("resize", this.resizeHandler);
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.resizeHandler);
    this.disposeAllCharts();
  },
  methods: {
    setPositiveThumbRef(chapter) {
      return (el) => {
        if (el) this.thumbPositiveRefs[chapter] = el;
        else delete this.thumbPositiveRefs[chapter];
      };
    },
    setNegativeThumbRef(chapter) {
      return (el) => {
        if (el) this.thumbNegativeRefs[chapter] = el;
        else delete this.thumbNegativeRefs[chapter];
      };
    },
    setPositiveDetailRef(chapter) {
      return (el) => {
        if (el) this.detailPositiveRefs[chapter] = el;
        else delete this.detailNegativeRefs[chapter];
      };
    },
    setNegativeDetailRef(chapter) {
      return (el) => {
        if (el) this.detailNegativeRefs[chapter] = el;
        else delete this.detailNegativeRefs[chapter];
      };
    },
    seededValue(seed) {
      let h = 2166136261;
      const str = String(seed || "");
      for (let i = 0; i < str.length; i++) {
        h ^= str.charCodeAt(i);
        h = Math.imul(h, 16777619);
      }
      h += h << 13;
      h ^= h >>> 7;
      h += h << 3;
      h ^= h >>> 17;
      h += h << 5;
      return (h >>> 0) / 4294967295;
    },
    getCacheKey(chapter, type) {
      return `${chapter}__${type}`;
    },

    buildRandomWordCloudData(words, chapter, type) {
      const picked = this.pickWords(words);
      return picked.map((word) => ({
        name: word,
        value: 22 + Math.round(Math.random() * 10),
        textStyle: {
          color: type === "positive" ? "#b33a2b" : "#5e7987",
        },
      }));
    },

    getWordCloudData(words, chapter, type) {
      const key = this.getCacheKey(chapter, type);
      if (!this.wordCloudDataCache[key]) {
        this.wordCloudDataCache[key] = this.buildRandomWordCloudData(words, chapter, type);
      }
      return this.wordCloudDataCache[key];
    },
    getPickedWords(words, chapter, type) {
      const key = this.getCacheKey(chapter, type);

      if (!this.pickedWordsCache[key]) {
        this.pickedWordsCache[key] = this.pickWords(words);
      }

      return this.pickedWordsCache[key];
    },
    pickWords(words) {
      const uniqueWords = [...new Set((words || []).filter(Boolean))];
      if (uniqueWords.length <= 10) return uniqueWords;

      const shuffled = [...uniqueWords];
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }

      return shuffled.slice(0, 10);
    },
    buildWordCloudData(words, chapter, type) {
      return this.getWordCloudData(words, chapter, type);
    },
    getBarSeriesColor(seriesType, dataIndex) {
      const selected = this.selectedChapters || [];
      const isSelected = selected.includes(dataIndex);
      if (selected.length === 0) {
        return seriesType === "positive" ? "#b33a2b" : "#5e7987";//此处调柱子颜色
      }
      if (isSelected) {
        return seriesType === "positive" ? "#b33a2b" : "#5e7987";//此处调柱子颜色
      }
      return seriesType === "positive" ? "rgba(179, 58, 43, 0.28)" : "rgba(89, 123, 122, 0.28)";
    },
    renderBarChart() {
      console.log('renderBarChart')
      if (this.showThumbnailWords || !this.$refs.barContainer) {
        if (barChart) {
          barChart.dispose();
          barChart = null;
        }
        return;
      }
      if (barChart) barChart.dispose();
      barChart = echarts.init(this.$refs.barContainer);
      const chapters = this.sentimentData.map(d => d.chapter);
      const positive = this.sentimentData.map(d => d.positiveCount || 0);
      const negative = this.sentimentData.map(d => d.negativeCount || 0);
      const option = {
        animationDuration: 500,
        title: { text: "情感词数量统计", left: "center", top: 4, textStyle: { fontSize: 16, fontWeight: 600 } },
        tooltip: {
          show: true,
          trigger: "axis",
          triggerOn: "mousemove",
          axisPointer: { type: "shadow" },
          confine: true,
          formatter: (params) => {
            if (!params || !params.length) return "";
            const chapterName = params[0].axisValue || "";
            const lines = [chapterName];
            params.forEach((item) => {
              lines.push(`${item.marker}${item.seriesName}：${item.value}`);
            });
            return lines.join("<br/>");
          },
        },
        legend: { data: ["积极情感词", "消极情感词"], left: 'right', top: 28, selectedMode: false },
        grid: {
          top: this.isDetailMode ? 68 : 60,
          left: 42,
          right: 16,
          bottom: this.isDetailMode ? 70 : 52,
          containLabel: true,
        },
        xAxis: {
          type: "category",
          data: chapters,
          axisLabel: {
            rotate: this.isDetailMode ? 28 : 38,
            interval: 0,
            fontSize: this.isDetailMode ? 11 : 10,
          },
        },
        yAxis: { type: "value", name: "词频数量" },
        series: [
          {
            name: "积极情感词",
            type: "bar",
            barMaxWidth: 26,
            color: "#b33a2b",
            data: positive.map((value, index) => ({
              value,
              itemStyle: { color: this.getBarSeriesColor("positive", index) },
            })),
          },
          {
            name: "消极情感词",
            type: "bar",
            barMaxWidth: 26,
            color: "#5e7987",
            data: negative.map((value, index) => ({
              value,
              itemStyle: { color: this.getBarSeriesColor("negative", index) },
            })),
          },
        ],
      };
      console.log(option, 'test option')
      barChart.setOption(option,true);
      barChart.off("click");
      barChart.on("click", (params) => {
        if (params.componentType === "series") {
          const chapterName = chapters[params.dataIndex];
          this.$emit("chapter-change", chapterName);
        }
      });
    },
    renderOneCloud(el, chapter, words, type) {
      if (!el) return;
      const key = `${chapter}-${type}-${this.isDetailMode ? 'detail' : 'thumb'}`;
      const oldChart = this.cloudCharts.get(key);
      if (oldChart) oldChart.dispose();
      const chart = echarts.init(el);
      this.cloudCharts.set(key, chart);
      const seriesData = this.buildWordCloudData(words, chapter, type);
      chart.setOption({
        tooltip: { show: false },
        series: [{
          type: 'wordCloud',
          shape: 'diamond',
          left: 0,
          top: 0,
          width: '100%',
          height: '100%',
          sizeRange: this.isDetailMode ? [16, 34] : [12, 24],
          rotationRange: [0, 0],
          rotationStep: 45,
          gridSize: 8,
          drawOutOfBound: false,
          textStyle: {
            fontFamily: 'STKaiti',
            fontWeight: 600,
          },
          /*emphasis: {
            textStyle: {
              shadowBlur: 8,
              shadowColor: 'rgba(0,0,0,0.18)',
            },
          },*/
          data: seriesData,
        }],
      });
    },
    renderCloudCharts() {
      this.disposeCloudCharts();
      this.$nextTick(() => {
        this.selectedSentimentItems.forEach(item => {
          const positiveWords = item.positiveWords || [];
          const negativeWords = item.negativeWords || [];
          const positiveEl = this.isDetailMode
            ? this.detailPositiveRefs[item.chapter]
            : this.thumbPositiveRefs[item.chapter];
          const negativeEl = this.isDetailMode
            ? this.detailNegativeRefs[item.chapter]
            : this.thumbNegativeRefs[item.chapter];
          this.renderOneCloud(positiveEl, item.chapter, positiveWords, 'positive');
          this.renderOneCloud(negativeEl, item.chapter, negativeWords, 'negative');
        });
      });
    },
    disposeCloudCharts() {
      this.cloudCharts.forEach(chart => chart && chart.dispose());
      this.cloudCharts.clear();
    },
    disposeAllCharts() {
      if (barChart) {
        barChart.dispose();
        barChart = null;
      }
      this.disposeCloudCharts();
    },
    refreshAll() {
      this.$nextTick(() => {
        this.renderBarChart();
        this.renderCloudCharts();
      });
    },
  },
};
</script>

<style scoped>
.sentiment-chart-shell {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.sentiment-chart-container {
  width: 100%;
  min-height: 200px;
}

.thumb-bar {
  flex: 1;
  height: 100%;
}

.detail-bar {
  height: 46%;
  min-height: 240px;
  flex: 0 0 46%;
}

.thumbnail-wordclouds,
.detail-wordclouds {
  width: 100%;
  height: 100%;
  min-height: 0;
}

.thumbnail-wordclouds {
  display: flex;
  flex-direction: row;
  gap: 10px;
  align-items: stretch;
}

.detail-wordclouds {
  flex: 1;
  min-height: 0;
  padding-top: 8px;
}

.detail-wordclouds-inner {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
  gap: 10px;
}

.thumbnail-chapter-block,
.detail-chapter-block {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  border: 1px solid rgba(31, 36, 36, 0.08);
  background: rgba(255, 255, 255, 0.18);
  padding: 8px;
}

.thumbnail-chapter-title {
  font-size: 14px;
  font-weight: 700;
  line-height: 1.4;
  margin-bottom: 4px;
  text-align: center;
}

.thumbnail-count-row {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  margin-bottom: 6px;
  color: rgba(31, 36, 36, 0.82);
}

.thumbnail-cloud-row {
  flex: 1;
  min-height: 0;
  display: flex;
  gap: 8px;
}

.thumbnail-cloud-panel {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.thumbnail-cloud-title {
  font-size: 12px;
  text-align: center;
  margin-bottom: 4px;
  color: rgba(31, 36, 36, 0.78);
}

.cloud-chart {
  flex: 1;
  min-height: 140px;
}

.detail-empty-tip {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  color: rgba(31, 36, 36, 0.58);
}
</style>
