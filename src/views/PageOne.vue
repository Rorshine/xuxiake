<template>
  <div class="page-container">
    <!-- 左侧章节目录（完全保留原样） -->
    <div class="chapter-list" ref="chapterList">
      <div class="panel-header">
        <div class="panel-title">章节目录</div>
        <div class="panel-subtitle">选择篇目查看路线与时间轴</div>
      </div>
      <div
        v-for="(chapter, index) in chapters"
        :key="index"
        class="chapter-item"
        :class="{ active: selectedChapters.includes(index) }"
        :data-index="index"
        @click="selectChapter(index)"
      >
        {{ chapter }}
      </div>
    </div>

    <!-- 右侧内容区：上下两栏（地图 + 情感图） -->
    <div class="right-panel" :class="{ 'detail-mode': currentDetail !== null }">
      <div class="map-wrapper" :class="{ 'detail-active': currentDetail === 'map' }">
        <MapView
          ref="mapView"
          :active-chapter="primaryActiveChapter"
          :selected-chapters="mapSelectedChapters"
          :is-detail-mode="currentDetail === 'map'"
          :locations="locations"
          :time-data="timeData"
          :chapters="chapters"
          @chapter-change="handleChapterChange"
        />
        <button v-if="currentDetail === 'map'" class="detail-back-btn" @click="exitDetailMode">返回</button>
        <button v-else class="detail-enter-btn" @click="enterDetailMode('map')">⛶ 放大地图</button>
      </div>

      <div class="sentiment-wrapper" :class="{ 'detail-active': currentDetail === 'sentiment' }">
        <SentimentChart
          ref="sentimentChart"
          :active-chapter="primaryActiveChapter"
          :selected-chapters="selectedChapters"
          :is-detail-mode="currentDetail === 'sentiment'"
          :chapters="chapters"
          :sentiment-data="sentimentData"
          @chapter-change="handleChapterChange"
        />
        <button v-if="currentDetail === 'sentiment'" class="detail-back-btn" @click="exitDetailMode">返回</button>
        <button v-else class="detail-enter-btn" @click="enterDetailMode('sentiment')">⛶ 放大情感图</button>
      </div>
    </div>
  </div>
</template>

<script>
import MapView from "../components/MapView.vue";
import SentimentChart from "../components/SentimentChart.vue";

export default {
  name: "PageOne",
  components: { MapView, SentimentChart },
  data() {
    return {
      chapters: [],
      activeChapter: null,
      selectedChapters: [],
      locations: [],
      timeData: [],
      sentimentData: [],
      currentDetail: null,
      mapSelectionBackup: [],
    };
  },
  computed: {
    primaryActiveChapter() {
      return this.selectedChapters.length ? this.selectedChapters[0] : null;
    },
    mapSelectedChapters() {
      if (this.currentDetail === "map") {
        return this.primaryActiveChapter !== null ? [this.primaryActiveChapter] : [];
      }
      return this.selectedChapters.slice(0, 3);
    },
  },
  async mounted() {
    await this.loadAllData();
    this.$nextTick(() => {
      setTimeout(() => {
        if (this.$parent && this.$parent.fullpageInstance) {
          this.$parent.fullpageInstance.reBuild();
        }
      }, 300);
    });
  },
  methods: {
    async loadAllData() {
      try {
        const chapterRes = await fetch("/data/chapters.json");
        const chapterData = await chapterRes.json();
        this.chapters = chapterData.chapters;

        const locationRes = await fetch("/data/dataset_total.json");
        const locationData = await locationRes.json();
        this.locations = locationData.map(item => ({
          name: item["地名"],
          lon: item["地点经度（默认东经）"],
          lat: item["地点纬度（默认北纬）"],
          chapter: item["所属篇目"],
          route: item["详细路线"],
        }));

        const timeRes = await fetch("/data/dataset_time.json");
        const timeDataRaw = await timeRes.json();
        this.timeData = timeDataRaw.map(item => ({
          name: item["地名"],
          lon: item["地点经度（默认东经）"],
          lat: item["地点纬度（默认北纬）"],
          chapter: item["所属篇目"],
          route: item["详细路线"],
          time: this.extractDate(item["游历时间"]),
          info: item["地点信息补充（来自百科）"],
          impo: item["重要事件"],
        }));

        await this.loadSentimentData();
      } catch (error) {
        console.error("数据加载失败:", error);
      }
    },

    extractDate(timeString) {
      if (!timeString) return new Date(1940, 0, 1);
      const yearRegex = /[（(](\d{4})[）)]/;
      const yearMatch = timeString.match(yearRegex);
      const year = yearMatch ? parseInt(yearMatch[1], 10) : 1940;
      const monthRegex = /(\d+)(?=月)/;
      const monthMatch = timeString.match(monthRegex);
      const month = monthMatch ? parseInt(monthMatch[1], 10) - 1 : 0;
      const dayRegex = /(\d+)(?=日)/;
      const dayMatch = timeString.match(dayRegex);
      const day = dayMatch ? parseInt(dayMatch[1], 10) : 1;
      return new Date(year, month, day);
    },

    normalizeChapterName(name) {
      return String(name || "")
        .replace(/[《》]/g, "")
        .replace(/\s+/g, "")
        .replace(/后/g, "后")
        .trim();
    },

    async loadSentimentData() {
      try {
        const [res1, res2] = await Promise.all([
          fetch("/data/sentiment1.json"),
          fetch("/data/sentiment2.json"),
        ]);
        const [data1, data2] = await Promise.all([res1.json(), res2.json()]);
        const merged = [...data1, ...data2];
        const sentimentMap = new Map();
        merged.forEach(item => {
          const key = this.normalizeChapterName(item["篇目"]);
          sentimentMap.set(key, {
            chapter: item["篇目"],
            positiveCount: Number(item["积极词数"] || 0),
            negativeCount: Number(item["消极词数"] || 0),
            positiveWords: (item["情感词"] || []).filter(w => Number(w["情感值"]) > 0).map(w => w["词语"]),
            negativeWords: (item["情感词"] || []).filter(w => Number(w["情感值"]) < 0).map(w => w["词语"]),
          });
        });

        this.sentimentData = this.chapters.map(ch => {
          const found = sentimentMap.get(this.normalizeChapterName(ch));
          return found || {
            chapter: ch,
            positiveCount: 0,
            negativeCount: 0,
            positiveWords: [],
            negativeWords: [],
          };
        });
      } catch (error) {
        console.error("情感数据加载失败:", error);
        this.sentimentData = this.chapters.map(ch => ({
          chapter: ch,
          positiveCount: 0,
          negativeCount: 0,
          positiveWords: [],
          negativeWords: [],
        }));
      }
    },

    selectChapter(index) {
      const exists = this.selectedChapters.includes(index);
      if (exists) {
        this.selectedChapters = this.selectedChapters.filter(i => i !== index);
      } else {
        if (this.selectedChapters.length >= 3) return;
        this.selectedChapters = [...this.selectedChapters, index];
      }
      this.activeChapter = this.primaryActiveChapter;
      this.$nextTick(() => {
        const container = this.$refs.chapterList;
        const el = container?.querySelector(`[data-index="${index}"]`);
        if (el && typeof el.scrollIntoView === "function") {
          el.scrollIntoView({ block: "nearest", behavior: "smooth" });
        }
      });
    },

    handleChapterChange(chapterName) {
      const idx = this.chapters.findIndex(c => c === chapterName);
      if (idx === -1) return;
      if (this.selectedChapters.includes(idx)) {
        this.selectChapter(idx);
        return;
      }
      if (this.selectedChapters.length >= 3) {
        this.selectedChapters = [idx];
        this.activeChapter = idx;
        return;
      }
      this.selectChapter(idx);
    },

    enterDetailMode(type) {
      if (type === "map") {
        // 进入地图详情模式前，备份当前总览模式下的多选状态
        this.mapSelectionBackup = [...this.selectedChapters];

        if (this.selectedChapters.length > 1) {
          const first = this.selectedChapters[0];
          this.selectedChapters = first !== undefined ? [first] : [];
          this.activeChapter = first ?? null;
        }
      }

      this.currentDetail = type;
    },

    exitDetailMode() {
      if (this.currentDetail === "map" && this.mapSelectionBackup.length > 0) {
        this.selectedChapters = [...this.mapSelectionBackup];
        this.activeChapter = this.mapSelectionBackup[0] ?? null;
      }

      this.currentDetail = null;
      this.mapSelectionBackup = [];
    },
  },
};
</script>

<style scoped>
/* ========== 全局变量（完全保留原样） ========== */
.page-container {
  --paper-0: #f6f1e2;
  --paper-1: #efe6cf;
  --paper-2: #e4dbc1;
  --ink-0: #1f2424;
  --ink-1: rgba(31, 36, 36, 0.78);
  --ink-2: rgba(31, 36, 36, 0.58);
  --seal: #b33a2b;
  --seal-weak: rgba(179, 58, 43, 0.22);
  --jade: #597b7a;
  --gold: #b7923a;
  --border: rgba(31, 36, 36, 0.14);
  --hairline: rgba(31, 36, 36, 0.10);
  --shadow: 0 18px 40px rgba(0, 0, 0, 0.12);
  --shadow-soft: 0 12px 26px rgba(0, 0, 0, 0.10);
}

.page-container * {
  box-sizing: border-box;
}

/* 主容器布局 */
.page-container {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  /* padding: 70px 24px 16px; */
  gap: 16px;
  width: 100%;
  max-width: 100%;
  height: 100%;
  min-height: 0;
  margin-top: 50px;
  overflow: hidden;
  font-family: "Source Han Serif SC", "Noto Serif CJK SC", "Songti SC", "STSong", "SimSun", serif;
  color: var(--ink-0);
  background:
    radial-gradient(1100px 700px at 18% 15%, rgba(0, 0, 0, 0.06), transparent 60%),
    radial-gradient(1000px 650px at 86% 28%, rgba(0, 0, 0, 0.05), transparent 62%),
    repeating-linear-gradient(90deg, rgba(31, 36, 36, 0.025) 0px, rgba(31, 36, 36, 0.025) 1px, transparent 1px, transparent 7px),
    repeating-linear-gradient(0deg, rgba(31, 36, 36, 0.018) 0px, rgba(31, 36, 36, 0.018) 1px, transparent 1px, transparent 9px),
    radial-gradient(900px 520px at 20% 35%, rgba(179, 58, 43, 0.10), transparent 62%),
    radial-gradient(900px 520px at 80% 40%, rgba(89, 123, 122, 0.10), transparent 64%),
    linear-gradient(180deg, var(--paper-0), var(--paper-1));
}

/* ========== 左侧章节目录（完全保留原样） ========== */
.chapter-list {
  display: flex;
  flex-direction: column;
  width: 20%;
  max-width: 250px;
  min-width: 150px;
  padding: 14px 10px;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.58), rgba(255, 255, 255, 0.38)),
    radial-gradient(500px 260px at 40% 20%, rgba(179, 58, 43, 0.06), transparent 70%),
    linear-gradient(180deg, rgba(246, 241, 226, 0.92), rgba(239, 230, 207, 0.88));
  border: 1px solid var(--border);
  flex-shrink: 0;
  position: relative;
  min-height: 0;           /* 允许收缩 */
}
.chapter-list::before {
  content: "";
  position: absolute;
  inset: 10px;
  border: 1px dashed rgba(31, 36, 36, 0.16);
  pointer-events: none;
}
.panel-header {
  padding: 8px 10px 12px;
  border-bottom: 1px solid var(--hairline);
  margin-bottom: 8px;
}
.panel-title {
  font-size: 18px;
  font-weight: 800;
  color: var(--ink-0);
  letter-spacing: 0.5px;
  position: relative;
  padding-left: 10px;
}
.panel-title::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  width: 6px;
  height: 6px;
  transform: translateY(-50%) rotate(12deg);
  background: var(--seal);
  border-radius: 2px;
  box-shadow: 0 6px 14px rgba(179, 58, 43, 0.25);
}
.panel-subtitle {
  margin-top: 4px;
  font-size: 12px;
  color: var(--ink-2);
}
.chapter-item {
  font-family: "KaiTi", "Kaiti SC", "STKaiti", "DFKai-SB", "Source Han Serif SC", serif;
  padding: 10px 12px;
  margin: 6px 6px;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.70), rgba(255, 255, 255, 0.50)),
    radial-gradient(260px 120px at 30% 30%, rgba(31, 36, 36, 0.04), transparent 60%);
  color: var(--ink-0);
  border: 1px solid var(--hairline);
  cursor: pointer;
  transition: background-color 0.2s ease, color 0.2s ease, transform 0.2s ease, box-shadow 0.2s ease;
  line-height: 1.25;
}
.chapter-item:hover {
  background:
    linear-gradient(180deg, rgba(246, 241, 226, 0.82), rgba(239, 230, 207, 0.78)),
    radial-gradient(260px 120px at 30% 30%, rgba(179, 58, 43, 0.08), transparent 60%);
  transform: translateY(-1px);
  box-shadow: 0 10px 18px rgba(0, 0, 0, 0.08);
}
.chapter-item.active {
  background: linear-gradient(135deg, rgba(179, 58, 43, 0.96), rgba(183, 146, 58, 0.86));
  color: #ffffff;
  border-color: rgba(255, 255, 255, 0.25);
  box-shadow: 0 14px 26px rgba(0, 0, 0, 0.14);
}

/* ========== 右侧面板（新增） ========== */
.right-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
  transition: all 0.3s ease;
  overflow: hidden;
  min-width: 0;
  min-height: 0;
  overflow-x: hidden; /* 新增：防止横向溢出 */
}

.map-wrapper,
.sentiment-wrapper {
  position: relative;

  min-height: 0;
  transition: flex 0.3s ease, opacity 0.2s ease;
  border: 1px solid var(--border);
  background: #f6f1e2;
  overflow: hidden;
  box-sizing: border-box;
  /*flex-direction: column;  */
}
.map-wrapper {
  flex: 1.2;   /* 地图占比稍大 */
}

.sentiment-wrapper {
  flex: 1;     /* 情感图占比稍小 */
}
.map-wrapper {
  padding-right: 12px;
  padding-bottom: 10px;
}

/* 详情模式：选中的组件放大，另一个缩小隐藏 */
.right-panel.detail-mode .map-wrapper,
.right-panel.detail-mode .sentiment-wrapper {
  flex: 0.2;
  opacity: 0.6;
}
.right-panel.detail-mode .map-wrapper.detail-active,
.right-panel.detail-mode .sentiment-wrapper.detail-active {
  flex: 5;
  opacity: 1;
}

/* 放大/返回按钮（留在面板内边距内，避免贴齐外缘被裁切） */
.detail-enter-btn,
.detail-back-btn {
  position: absolute;
  bottom: 14px;
  right: 14px;
  left: auto;
  z-index: 10;
  max-width: calc(100% - 28px);
  box-sizing: border-box;
  background: rgba(31, 36, 36, 0.7);
  color: white;
  border: none;
  border-radius: 20px;
  padding: 6px 12px;
  font-size: 12px;
  cursor: pointer;
  backdrop-filter: blur(4px);
  transition: 0.2s;
  white-space: nowrap;
}
.detail-enter-btn:hover,
.detail-back-btn:hover {
  background: var(--seal);
}

/* 响应式（保留原章节目录的响应式，右侧面板继承即可） */
@media (max-width: 1100px) {
  .page-container {
    padding: 66px 16px 16px;
  }
  .chapter-list {
    min-width: 140px;
  }
}
@media (max-width: 920px) {
  .page-container {
    flex-direction: column;
    height: 100vh;
    overflow: auto;
  }
  .chapter-list {
    width: 100%;
    max-width: none;
    height: auto;
    max-height: 240px;
  }
  .right-panel {
    min-height: 60vh;
  }
}
</style>