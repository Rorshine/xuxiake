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
        '可按篇目查看叙事地图和情感词统计图词云图',
        '可查看跨省游历阶段的时间结构',
        '可查看山水名胜省份关联结构与名胜详情',
        '可查看景观类型划分和数量统计'
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
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 70px 24px 30px; /* 底部留白略增，整体更舒展 */
  overflow: hidden;
  box-sizing: border-box;
}

.home-center {
  flex: 1;
  overflow-y: auto;
  min-height: 0;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.title-block {
  margin-left: 24px;
}

.title-big {
  font-family: var(--font-kaiti);
  font-weight: 900;
  font-size: 66px; /* 原来 60 */
  letter-spacing: 7px;
  color: var(--ink-0);
  line-height: 1.05;
  position: relative;
}

.title-big::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: 6px;
  height: 14px;
  background: linear-gradient(90deg, rgba(31,36,36,0), rgba(31,36,36,0.14), rgba(31,36,36,0));
  pointer-events: none;
}

.title-big.sub {
  margin-top: 12px;
  font-size: 58px; /* 原来 52 */
  letter-spacing: 11px;
  color: rgba(31,36,36,0.86);
}

.home-bottom {
  flex-shrink: 0;
  height: auto;
  min-height: 128px;   /* 原来 92，整体更高 */
  padding-top: 0;
  margin-bottom: 18px; /* 上移一点：通过减少贴底感实现 */
  overflow-x: hidden;
}

.nav-row {
  width: 100%;
  display: flex;
  gap: 14px; /* 原来 10 */
  flex-wrap: wrap;
  overflow-x: hidden;
}

.nav-item {
  flex: 1;
  min-height: 104px; /* 新增：让导航标签更高 */
  border: 1px solid var(--border);
  background: rgba(255,255,255,0.20);
  padding: 18px 18px; /* 原来 12px 14px */
  cursor: pointer;
  text-align: left;
  font-family: var(--font-kaiti);
  transition: background-color 0.15s ease, border-color 0.15s ease, transform 0.15s ease;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.nav-item-title {
  display: block;
  font-weight: 900;
  letter-spacing: 1px;
  color: var(--ink-0);
  font-size: 24px; /* 新增：主标题更大 */
  line-height: 1.25;
}

.nav-item-desc {
  display: block;
  margin-top: 10px;
  color: rgba(31,36,36,0.0);
  font-size: 15px; /* 原来 12 */
  line-height: 1.45;
  transition: color 0.15s ease;
  min-height: 20px;
}

.nav-item:hover {
  background: rgba(179,58,43,0.06);
  border-color: rgba(179,58,43,0.28);
  transform: translateY(-2px);
}

.nav-item:hover .nav-item-desc {
  color: rgba(31,36,36,0.72);
  font-size: 15px;
}

@media (max-width: 980px) {
  .title-big { font-size: 48px; }
  .title-big.sub { font-size: 40px; }
  .nav-row { flex-wrap: wrap; }
  .nav-item { 
    flex: 1 1 calc(50% - 14px);
    min-height: 90px;
  }
  .nav-item-title {
    font-size: 20px;
  }
  .nav-item-desc {
    font-size: 15px;
  }
  .home-bottom {
    min-height: 110px;
    margin-bottom: 12px;
  }
}
</style>