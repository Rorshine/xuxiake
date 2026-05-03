<template>
  <div class="page-content">
    <div ref="chart" class="chart-container"></div>
    <div style="position: absolute; right: 80px; top: 50%; transform: translateY(-50%); z-index: 10;">
        <img src="../assets/img/page5.png" alt="Image" style="max-width: 300px; height: auto;">
    </div>
  </div>
</template>



<script lang="js">
import * as echarts from 'echarts';

export default {
  name: 'PageFive',
  data() {
    return {
      colors: ['#c0d695', '#428675', '#83a78d', '#83a78d', '#83a78d'],
      bgColor: '#90a07d',
      itemStyle: {
        star5: { color: '#a8b78c' },
        star4: { color: '#428675' },
        star3: { color: '#428675' },
        star2: { color: '#428675' }
      },
      data: [
        {
          name: '自然景观',
          itemStyle: { color: '#a9be7b' },
          children: [
            {
              name: '山',
              children: [
                { name: '62', children: [{ name: '云阳山' }, { name: '雨母山' }, { name: '香炉山' }] },
              ]
            },
            {
              name: '峰',
              children: [
                { name: '17', children: [{ name: '紫微峰' }, { name: '祝融峰' }, { name: '回雁峰' }] },
              ]
            },
            {
              name: '崖',
              children: [
                { name: '5', children: [{ name: '观音崖' }, { name: '出水崖' }, { name: '舍身崖' }] },
              ]
            },
            {
              name: '岭',
              children: [
                { name: '15', children: [{ name: '金牛岭' }, { name: '和尚岭' }, { name: '兰麻岭' }] },
              ]
            },
            {
              name: '顶',
              children: [
                { name: '6', children: [{ name: '金宝顶' }, { name: '金顶' }, { name: '华顶' }] },
              ]
            },
            {
              name: '石林',
              children: [
                { name: '11', children: [{ name: '狮子林' }, { name: '旎檀林' }, { name: '牛恋石' }] },
              ]
            },
            {
              name: '其他山',
              children: [
                { name: '4', children: [{ name: '四平坡' }, { name: '松坡' }, { name: '虎坡' }] },
              ]
            },
            {
              name: '江',
              children: [
                { name: '7', children: [{ name: '江渡' }, { name: '湘江' }, { name: '盘江' }] },
              ]
            },
            
            {
              name: '瀑布',
              children: [
                { name: '5', children: [{ name: '水帘洞瀑布' }, { name: '石梁飞瀑' }, { name: '断桥三曲瀑布' }] },
              ]
            },
            {
              name: '湖',
              children: [
                { name: '2', children: [{ name: '茈碧湖' }, { name: '太湖' }] },
              ]
            },
            {
              name: '潭',
              children: [
                { name: '7', children: [{ name: '云集潭' }, { name: '龙潭' }, { name: '香米龙潭' }] },
              ]
            },
            {
              name: '池',
              children: [
                { name: '8', children: [{ name: '温泉浴池' }, { name: '玉泉池' }, { name: '九台温泉' }] },
              ]
            },
            {
              name: '其他水',
              children: [
                { name: '14', children: [{ name: '皇雩仙' }, { name: '水车坝' }, { name: '清海子' }] },
              ]
            },
            {
              name: '岩',
              children: [
                { name: '48', children: [{ name: '灵岩' }, { name: '朝阳岩' }, { name: '澹岩' }] },
              ]
            },
            {
              name: '洞',
              children: [
                { name: '50', children: [{ name: '秦人洞' }, { name: '麻叶洞' }, { name: '水帘洞' }] },
              ]
            },
            {
              name: '滩',
              children: [
                { name: '13', children: [{ name: '钩栏滩' }, { name: '兰麻滩' }, { name: '苦竹滩' }] },
              ]
            }
          ]
        },
        {
          name: '人文景观',
          itemStyle: { color: '#c0d695' },
          children: [
            {
              name: '寺庙',
              children: [
                { name: '63', children: [{ name: '上封寺' }, { name: '天台寺' }, { name: '福严寺' }] },
              ]
            },
            {
              name: '祠堂',
              children: [
                { name: '7', children: [{ name: '潇湘祠' }, { name: '柳子祠' }, { name: '濂溪祠' }] },
              ]
            },
            {
              name: '宫',
              children: [
                { name: '9', children: [{ name: '静乐宫' }, { name: '迎恩宫' }, { name: '遇真宫' }] },
              ]
            },
            {
              name: '庵',
              children: [
                { name: '11', children: [{ name: '绿竹庵' }, { name: '清泉庵' }, { name: '朝阳庵' }] },
              ]
            },
            {
              name: '亭',
              children: [
                { name: '2', children: [{ name: '怡云亭' }, { name: '玉虚亭' }] },
              ]
            },
            {
              name: '桥',
              children: [
                { name: '15', children: [{ name: '草桥' }, { name: '九宫桥' }, { name: '南溪桥' }] },
              ]
            },
            {
              name: '寨',
              children: [
                { name: '6', children: [{ name: '东基寨' }, { name: '佛光寨' }, { name: '白王寨' }] },
              ]
            },
            {
              name: '阁',
              children: [
                { name: '12', children: [{ name: '观音阁' }, { name: '千佛阁' }, { name: '真武阁' }] },
              ]
            },
            {
              name: '观',
              children: [
                { name: '6', children: [{ name: '七星观' }, { name: '白基观' }, { name: '云台观' }] },
              ]
            },
            {
              name: '其他',
              children: [
                { name: '27', children: [{ name: '大罗城' }, { name: '参将府' }, { name: '大禹碑' }] },
              ]
            },
          ]
        }
      ]
    };
  },
  mounted() {
    this.formatData();
    this.initChart();
  },
  methods: {
    softenValue(count) {
      const n = Number(count) || 0;
      // 用幂函数压缩极差：既保留“数量越多角度越大”，又避免小类扇形过窄
      return Math.pow(n, 0.65) + 0.8;
    },
    formatData() {
      const { data, itemStyle } = this;
      for (let j = 0; j < data.length; ++j) {
        const level1 = data[j].children;
        let level0Total = 0;
        // 按每个景观子类的数量从多到少排序
        level1.sort((a, b) => {
          const getCount = (item) => {
            if (!item.children || !item.children.length) return 0;
            const countNode = item.children[0];
            return Number(countNode.name) || 0;
          };
          return getCount(b) - getCount(a);
        });
        for (let i = 0; i < level1.length; ++i) {
          const block = level1[i].children || [];
          let level1Total = 0;

          for (let star = 0; star < block.length; ++star) {
            let style = itemStyle.star5;
            const countNode = block[star];
            const rawCount = Number(countNode.name) || 0;
            const softenedCount = this.softenValue(rawCount);

            countNode.value = softenedCount;
            countNode.label = {
              color: style.color,
              downplay: { opacity: 0.5 }
            };

            if (countNode.children && countNode.children.length) {
              style = { opacity: 1, color: style.color };
              const leafValue = softenedCount / countNode.children.length;
              countNode.children.forEach((book) => {
                book.value = leafValue;
                book.itemStyle = style;
                book.label = { color: style.color };
              });
            }

            level1Total += softenedCount;
          }

          level1[i].value = level1Total;
          level1[i].itemStyle = { color: data[j].itemStyle.color };
          level0Total += level1Total;
        }

        data[j].value = level0Total;
      }
    },
    initChart() {
      const chart = echarts.init(this.$refs.chart);
      this.chart = chart; // Save the chart instance

      // Get the container size (width and height)
      const containerWidth = this.$refs.chart.offsetWidth;
      const containerHeight = this.$refs.chart.offsetHeight;

      // Calculate chart radius based on container size (keeping similar proportions)
      const minDimension = Math.min(containerWidth, containerHeight);
      const radius = minDimension / 11; // Set the base radius to be half of the smallest dimension

      chart.setOption({
        backgroundColor: 'transparent',
        color: this.colors,
        series: [
          {
            type: 'sunburst',
            center: ['50%', '50%'],
            radius: ['0%', '110%'],
            data: this.data,
            sort: null,
            label: { rotate: 'radial', color: this.bgColor },
            itemStyle: { borderColor: this.bgColor, borderWidth: 1.5 },
            levels: [
              {},
              { r0: 0, r: radius, label: { rotate: 0, fontSize:14,fontWeight: 'bold' } },
              { r0: radius, r: radius*2.9, 
                label:{fontSize:14,fontWeight: 'bold'}
              },
              {
                r0: radius*3,
                r: radius*3.4,
                itemStyle: { shadowBlur: 2, shadowColor: 'transparent', color: 'transparent' },
                label: { rotate: 'tangential', fontSize: 14, color: this.bgColor }
              },
              {
                r0: radius*3.55,
                r: radius*3.675,
                itemStyle: { shadowBlur: 80, shadowColor: 'transparent' },
                label: { position: 'outside',show: false,opacity: 0, textShadowBlur: 5, textShadowColor: 'transparent',fontSize:15,fontWeight:'bold'},
                downplay: { label: { opacity: 0.5 }},
                emphasis: { label: { opacity: 1,show: true} } 
              }
            ]
          }
        ]
      });
    }
  }
};
</script>

<style scoped>
.page-content {
  width: 100%;
  height: 100%;
  
}
.chart-container {
  width: 100%;
  height: 100%;
}


</style>
