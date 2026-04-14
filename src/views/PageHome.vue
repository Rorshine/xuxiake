<template>
  <div class="home x-theme">
    <div class="home-center">
      <div class="title-block">
        <div class="title-big">《徐霞客游记》</div>
        <div class="title-big sub">数据可视化系统</div>
      </div>
    </div>

    <div class="home-bottom">
      <div class="nav-row">
        <button
          v-for="(page, idx) in otherPages"
          :key="idx"
          class="nav-item"
          @click="goToPage(idx + 2)"
        >
          <span class="nav-item-title">{{ page.title }}</span>
          <span class="nav-item-desc">{{ pageSubtitles[idx] }}</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['pageConfigs'],
  computed: {
    otherPages() {
      return this.pageConfigs.slice(1);
    },
    pageSubtitles() {
      return [
        '按篇目筛选路线与时间轴高亮',
        '各省份情感词对比与分布',
        '跨省游历阶段的时间结构',
        '山水名胜关联与空间分布',
        '景观类型层级与词项结构'
      ];
    }
  },
  methods: {
    goToPage(pageIndex) {
      this.$emit('navigate', pageIndex);
    }
  }
};
</script>

<style scoped>
.home {
  width: 100%;
  height: 100vh;           /* 占满父容器 */
  display: flex;          /* 启用 flex 列布局 */
  flex-direction: column;
  justify-content: space-between; /* 两端对齐：上部分和下部分 */
  padding: 70px 24px 18px;
  overflow: hidden;       /* 整体不产生滚动条（滚动由内部区域负责） */
  box-sizing: border-box;
}

/* 上部分：可滚动区域（当内容过多时） */
.home-center {
  flex: 1;                /* 占据剩余空间 */
  overflow-y: auto;       /* 内容过多时内部滚动 */
  min-height: 0;          /* 允许 flex 子项收缩 */
  display: flex;
  align-items: center;    /* 垂直居中（原来效果） */
  justify-content: flex-start;
}
.title-block{
  margin-left: 24px;
}
.title-big{
  font-family: var(--font-kaiti);
  font-weight: 900;
  font-size: 60px;
  letter-spacing: 6px;
  color: var(--ink-0);
  line-height: 1.05;
  position: relative;
}
.title-big::after{
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: 6px;
  height: 14px;
  background: linear-gradient(90deg, rgba(31,36,36,0), rgba(31,36,36,0.14), rgba(31,36,36,0));
  pointer-events: none;
}
.title-big.sub{
  margin-top: 10px;
  font-size: 52px;
  letter-spacing: 10px;
  color: rgba(31,36,36,0.86);
}

/* 下部分：固定高度（或由内容撑开） */
.home-bottom {
  flex-shrink: 0;         /* 不缩小 */
  height: auto;           /* 自适应高度，不再固定 92px */
  min-height: 92px;
  padding-top: 10px;
  overflow-x: hidden;
}
/* 导航按钮行 */
.nav-row {
  width: 100%;
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  overflow-x: hidden;
}
.nav-item{
  flex: 1;
  border: 1px solid var(--border);
  background: rgba(255,255,255,0.20);
  padding: 12px 14px;
  cursor: pointer;
  text-align: left;
  font-family: var(--font-kaiti);
  transition: background-color 0.15s ease, border-color 0.15s ease;
  position: relative;
}
.nav-item-title{
  display: block;
  font-weight: 900;
  letter-spacing: 1px;
  color: var(--ink-0);
}
.nav-item-desc{
  display: block;
  margin-top: 8px;
  color: rgba(31,36,36,0.0);
  font-size: 12px;
  line-height: 1.4;
  transition: color 0.15s ease;
  min-height: 16px;
}
.nav-item:hover{
  background: rgba(179,58,43,0.06);
  border-color: rgba(179,58,43,0.28);
}
.nav-item:hover .nav-item-desc{
  color: rgba(31,36,36,0.72);
}

@media (max-width: 980px){
  .title-big{ font-size: 44px; }
  .title-big.sub{ font-size: 38px; }
  .nav-row{ flex-wrap: wrap; }
  .nav-item{ flex: 1 1 calc(50% - 10px); }
  .home-bottom{ height: auto; padding-top: 10px; }
}
</style>