<template>
  <div ref="chartContainer" class="sentiment-chart-container"></div>
</template>

<script>
import * as echarts from "echarts";

export default {
  name: "SentimentChart",
  props: {
    activeChapter: { type: Number, default: null },
    isDetailMode: { type: Boolean, default: false },
    sentimentData: { type: Array, required: true }, // [{ chapter, positiveCount, negativeCount }]
  },
  data() {
    return {
      chart: null,
    };
  },
  watch: {
    sentimentData: {
      handler() { this.renderChart(); },
      deep: true,
    },
    activeChapter: {
      handler() { this.highlightCurrentChapter(); },
      immediate: true,
    },
    isDetailMode() {
      this.renderChart(); // 简单重新渲染
    },
  },
  mounted() {
    this.renderChart();
    window.addEventListener("resize", this.handleResize);
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.handleResize);
    if (this.chart) this.chart.dispose();
  },
  methods: {
    renderChart() {
      if (!this.$refs.chartContainer) return;
      if (this.chart) this.chart.dispose();
      this.chart = echarts.init(this.$refs.chartContainer);
      const chapters = this.sentimentData.map(d => d.chapter);
      const positive = this.sentimentData.map(d => d.positiveCount);
      const negative = this.sentimentData.map(d => d.negativeCount);

      const option = {
        title: { text: "情感词数量统计", left: "center", top: 0, textStyle: { fontSize: 14 } },
        tooltip: { trigger: "axis", axisPointer: { type: "shadow" } },
        legend: { data: ["积极情感词", "消极情感词"], left: "left", top: 30 },
        grid: { top: 80, bottom: 30, containLabel: true },
        xAxis: { type: "category", data: chapters, axisLabel: { rotate: this.isDetailMode ? 30 : 45, interval: 0, fontSize: 10 } },
        yAxis: { type: "value", name: "词频数量" },
        series: [
          { name: "积极情感词", type: "bar", data: positive, itemStyle: { color: "#b33a2b" } },
          { name: "消极情感词", type: "bar", data: negative, itemStyle: { color: "#597b7a" } },
        ],
      };
      this.chart.setOption(option);
      this.highlightCurrentChapter();
      // 点击柱子触发章节联动
      this.chart.off("click");
      this.chart.on("click", (params) => {
        if (params.componentType === "series") {
          const chapterName = chapters[params.dataIndex];
          this.$emit("chapter-change", chapterName);
        }
      });
    },
    highlightCurrentChapter() {
      if (!this.chart || this.activeChapter === null) return;
      // 高亮对应的柱子（通过dispatchAction）
      this.chart.dispatchAction({
        type: "highlight",
        seriesIndex: 0,
        dataIndex: this.activeChapter,
      });
      this.chart.dispatchAction({
        type: "highlight",
        seriesIndex: 1,
        dataIndex: this.activeChapter,
      });
    },
    handleResize() {
      if (this.chart) this.chart.resize();
    },
  },
};
</script>

<style scoped>
.sentiment-chart-container {
  width: 100%;
  height: 100%;
  min-height: 200px;
}
</style>