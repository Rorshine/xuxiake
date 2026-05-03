<template>
  <div style="height: 100%;">
    <nav class="navbar x-theme">
      <div class="logo-text" @click="goToSection(0)">徐霞客游记可视化</div>
      <ul>
        <li
          v-for="(page, index) in pages"
          :key="index"
          :class="{ active: currentPage === index }"
          @click="goToSection(index)"
        >
          {{ page }}
        </li>
      </ul>
    </nav>

    <div id="fullpage">
      <div
        v-for="(component, index) in pageComponents"
        :key="index"
        class="section"
        :class="'section' + (index + 1)"
      >
        <component :is="component" :pageConfigs="pageConfigs" @navigate="navigateToPage" />
      </div>
    </div>
  </div>
</template>

<script>
import fullpage from 'fullpage.js';
import 'fullpage.js/dist/fullpage.css';
import PageHome from './views/PageHome.vue';
import PageOne from './views/PageOne.vue';
// import PageTwo from './views/PageTwo.vue';
import PageThree from './views/PageThree.vue';
import PageFour from './views/PageFour.vue';
import PageFive from './views/PageFive.vue';

export default {
  name: 'App',
  components: {
    PageHome,
    PageOne,
    // PageTwo,
    PageThree,
    PageFour,
    PageFive,
  },
  data() {
    return {
      pageConfigs: [
        { title: '平台首页', component: PageHome },
        { title: '叙事地图和情感统计图', component: PageOne },
        // { title: '游记情感词统计', component: PageTwo },
        { title: '省份游历甘特图', component: PageThree },
        { title: '徐霞客与山水名胜', component: PageFour },
        { title: '徐霞客与各色景观', component: PageFive },
      ],
      currentPage: 0,
      fullpageInstance: null,
    };
  },
  computed: {
    pages() {
      return this.pageConfigs.map(item => item.title);
    },
    pageComponents() {
      return this.pageConfigs.map(item => item.component);
    }
  },
  mounted() {
    this.fullpageInstance = new fullpage('#fullpage', {
      navigation: false,
      scrollingSpeed: 700,
      anchors: this.pageConfigs.map((_, index) => `page${index + 1}`),
      showActiveTooltip: false,
      licenseKey: '你的密钥', // 记得添加
      afterLoad: (origin, destination) => {
        this.currentPage = destination.index;
      },
    });
  },
  methods: {
    goToSection(index) {
      if (this.fullpageInstance) {
        this.fullpageInstance.moveTo(index + 1);
      }
    },
    
    navigateToPage(pageIndex) {
      // 供子组件调用，pageIndex 是目标页面在 fullpage 中的索引（1-based）
      if (this.fullpageInstance) {
        this.fullpageInstance.moveTo(pageIndex);
      }
    }
  },
};
</script>

<style>
/* 避免 100vw + 滚动条导致横向溢出；fullpage 整页滚动由插件接管 */
html,
body {
  margin: 0;
  max-width: 100%;
  overflow-x: hidden;
  box-sizing: border-box;
}
html.fp-enabled,
html.fp-enabled body {
  overflow: hidden;
  height: 100%;
}

/* 设置页面样式 */
#app {
  font-family: var(--font-serif);
}

.navbar {
  position: fixed;
  top: 0;
  width: 100%;
  background: linear-gradient(180deg, rgba(246, 241, 226, 0.92), rgba(239, 230, 207, 0.78));
  color: var(--ink-0);
  display: flex;
  justify-content: space-between; /* 让 logo 和导航项分开 */
  align-items: center;
  padding: 10px 20px;
  z-index: 1000;
  border-bottom: 1px solid var(--hairline);
  backdrop-filter: blur(8px);
}

.logo-text{
  font-family: var(--font-kaiti);
  font-weight: 900;
  font-size: 26px;
  letter-spacing: 3px;
  color: var(--ink-0);
  cursor: pointer;
  white-space: nowrap;
  padding: 4px 0;
  position: relative;
}
.logo-text::after{
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: 2px;
  height: 8px;
  background: linear-gradient(90deg, rgba(31,36,36,0.0), rgba(31,36,36,0.14), rgba(31,36,36,0.0));
  pointer-events: none;
}

.navbar ul {
  list-style-type: none;
  display: flex;
  gap: 20px;
  margin: 0;
  padding: 0;
  flex-grow: 1; /* 让导航项占据剩余空间 */
  justify-content: center; /* 在剩余空间内居中对齐 */
}

.navbar li {
  cursor: pointer;
  padding: 5px 10px;
  font-size: 18px;
  font-family: var(--font-kaiti);
  font-weight: 700;
  color: var(--ink-1);
  border-radius: 0px;
  transition: background-color 0.15s ease, color 0.15s ease;
}
.navbar li:hover{
  background: rgba(179, 58, 43, 0.06);
  color: var(--ink-0);
}

.navbar li.active {
  font-weight: bold;
  color: var(--seal);
  background: rgba(179, 58, 43, 0.10);
}

/* 兜底隐藏 fullpage 右侧圆点导航 */
.fp-right{
  display: none !important;
}

.section {
  display: flex;
  justify-content: center;
  align-items: stretch;
  height: 100vh;
  width: 100%;
  box-sizing: border-box;
  background-color: #e4e0cf; /* 默认背景色 */
   /* 缩小背景图片，确保完全显示 */
  
  background-repeat: no-repeat; /* 防止背景图片重复 */
  /*flex-direction: column;*/
  /*overflow: hidden;*/
}

/* 让各页根组件铺满整屏，避免内容高度收缩导致 fullpage 视觉错位 */
.section > * {
  flex: 1 1 auto;
  width: 100%;
  min-height: 0;
  box-sizing: border-box;
}

.section1 {
  
  /*background-image: url('./assets/img/background.png'); */
  background-position: left bottom; /* 背景图片靠左下 */
  background-size: 50%;
}

.section2 {
  /* background-image: url('./assets/img/background3.png'); */
  background-position: right bottom;
  background-size: 35%;
}

.section3 {
  /*background-image: url('./assets/img/background3.png'); */
  background-position: right bottom;
  background-size: 35%;
}

.section4 {
  /*background-image: url('./assets/img/background.png');  */
  background-position: left bottom; /* 背景图片靠左下 */
  background-size: 50%;
}

.section5 {
  /* background-image: url('./assets/img/background.png');  */
  background-position: left bottom; /* 背景图片靠左下 */
  background-size: 50%;
}
</style>
