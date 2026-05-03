<template>
  <div style="width:100%">
    <div style="display: flex; height: 136px;">
      <div class="title" style="width: 512px; visibility: hidden;">
        <p>{{ title }}</p>
      </div>
      <t-divider layout="vertical" style="height: 90%; visibility: hidden;"></t-divider>
    </div>

    <div style="display: flex; width:100%">
      <div ref="ganttChart" class="gantt"></div>
      <div style="display: flex; justify-content: center; align-items: center;">
        <img :src="require('../assets/img/各省份旅行甘特图.png')" width="50%" />
      </div>
    </div>
  </div>
</template>

<script>
import * as echarts from "echarts";
import { GridComponent, CalendarComponent, VisualMapComponent, TooltipComponent } from 'echarts/components';
import { SVGRenderer, CanvasRenderer } from 'echarts/renderers';

echarts.use([
  SVGRenderer,
  CanvasRenderer,
  TooltipComponent,
  VisualMapComponent,
  CalendarComponent,
  GridComponent
]);

export default {
  name: 'PickView',
  data() {
    return {
      title: '徐霞客游历各地时间表',
      cell_ratio: 0.026836,
      progressChart: null,
      resizeHandler: null,
    };
  },

  computed: {
    dataAxis() {
      return ['江苏', '浙江', '安徽', '福建', '江西', '湖北', '湖南', '广西', '贵州', '云南'];
    },
  },

  mounted() {
    this.$nextTick(() => {
      this.createGantt();
      this.resizeHandler = () => {
        if (this.progressChart) {
          this.progressChart.resize();
        }
      };
      window.addEventListener('resize', this.resizeHandler);
    });
  },

  beforeUnmount() {
    if (this.resizeHandler) {
      window.removeEventListener('resize', this.resizeHandler);
    }
    if (this.progressChart) {
      this.progressChart.dispose();
      this.progressChart = null;
    }
  },

  methods: {
    createGantt() {
      const chartDom = this.$refs.ganttChart;
      if (!chartDom) return;

      if (this.progressChart) {
        this.progressChart.dispose();
      }

      const progressChart = echarts.init(chartDom, null, { renderer: 'svg' });
      this.progressChart = progressChart;

      const dataAxis = this.dataAxis;

      const option = {
        tooltip: {
          trigger: 'item',
          alwaysShowContent: false,
          formatter: (params) => {
            const dataIndex = params.dataIndex;
            const seriesIndex = params.seriesIndex;
            const provinceName = this.dataAxis[dataIndex] || params.name;

            // 奇数序列为浅色占位段：提示当前不在该省
            if (seriesIndex % 2 === 1) {
              return `${provinceName}<br/>徐霞客此时不在该省`;
            }

            // 偶数序列为实际停留段：显示起止时间与游历天数
            const startValue = option.series[seriesIndex]?.data?.[dataIndex];
            const endValue = option.series[seriesIndex + 1]?.data?.[dataIndex];

            if (!startValue || !endValue) {
              return `${provinceName}<br/>无数据`;
            }

            const startDate = new Date(startValue);
            const endDate = new Date(endValue);
            const diffTime = endDate.getTime() - startDate.getTime();
            const diffDays = Math.round(diffTime / (1000 * 60 * 60 * 24)) + 1;

            return `${provinceName}<br/>起始时间：${startValue}<br/>结束时间：${endValue}<br/>游历时间：${diffDays} 天`;
          }
        },

        grid: {
          containLabel: true,
          show: false,
          right: 10,
          left: 10,
          bottom: 20,
          top: 10,
          backgroundColor: 'transparent'
        },

        xAxis: {
          type: 'time',
          position: 'top',
          axisTick: {
            show: true
          },
          axisLine: {
            show: true
          },
          splitLine: {
            show: false
          },
          min: new Date('1613-03-15').getTime(),
          max: new Date('1639-12-31').getTime(),
          axisLabel: {
            formatter: function (value) {
              const date = new Date(value);
              return date.toLocaleDateString('zh-CN', { year: 'numeric' });
            }
          }
        },

        yAxis: {
          inverse: true,
          axisTick: {
            show: true
          },
          axisLine: {
            show: true
          },
          splitLine: {
            show: true
          },
          axisPointer: {
            type: 'shadow'
          },
          dataZoom: [
            {
              type: 'inside'
            }
          ],
          data: dataAxis,
        },

        series: [
          // 1
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function (params) {
                const colors = [
                  'rgba(50,132,110,0.8)', 'rgba(50,132,110)', 'rgba(50,132,110)',
                  'rgba(50,132,110)', 'rgba(50,132,110,0.6)', 'rgba(35,118,183)',
                  'rgba(35,118,183,0.4)', 'rgba(35,118,183,0.6)', 'rgba(35,118,183,0.8)',
                  'rgba(35,118,183,0.3)',
                ];
                return colors[params.dataIndex];
              },
              borderColor: 'transparent',
              borderWidth: 1
            },
            zlevel: 1,
            z: 10,
            data: [
              null, '1613-03-31', '1616-01-26', '1616-02-21', '1618-08-18', '1623-03-11', '1637-01-11', '1637-04-16', '1638-03-11', '1638-05-10'
            ]
          },
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            itemStyle: {
              borderColor: 'transparent',
              color: '#e4e0cf'
            },
            zlevel: 1,
            z: 20,
            data: [
              null, '1613-04-14', '1616-02-11', '1616-03-20', '1618-08-23', '1623-04-09', '1637-04-16', '1638-03-10', '1638-05-09', '1638-07-15'
            ]
          },

          // 2
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function (params) {
                const colors = [
                  'rgba(50,132,110,0.8)', 'rgba(50,132,110)', 'rgba(50,132,110)',
                  'rgba(50,132,110)', 'rgba(50,132,110,0.6)', 'rgba(35,118,183)',
                  'rgba(35,118,183,0.4)', 'rgba(35,118,183,0.6)', 'rgba(35,118,183,0.8)',
                  'rgba(35,118,183,0.3)',
                ];
                return colors[params.dataIndex];
              },
              borderColor: 'transparent',
              borderWidth: 1
            },
            zlevel: 2,
            z: 10,
            data: [
              '1614-01-01', '1620-05-06', '1618-09-04', '1620-06-08', '1636-10-17', null, null, '1638-05-10', '1638-08-26', '1638-08-16'
            ]
          },
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            itemStyle: {
              borderColor: 'transparent',
              color: '#e4e0cf'
            },
            zlevel: 2,
            z: 20,
            data: [
              '1615-12-31', '1620-05-23', '1618-09-06', '1620-06-11', '1637-01-11', null, null, '1638-05-10', '1638-09-01', '1638-08-26'
            ]
          },

          // 3
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function (params) {
                const colors = [
                  'rgba(50,132,110,0.8)', 'rgba(50,132,110)', 'rgba(50,132,110)',
                  'rgba(50,132,110)', 'rgba(50,132,110,0.6)', 'rgba(35,118,183)',
                  'rgba(35,118,183,0.4)', 'rgba(35,118,183,0.6)', 'rgba(35,118,183,0.8)',
                  'rgba(35,118,183,0.3)',
                ];
                return colors[params.dataIndex];
              },
              borderColor: 'transparent',
              borderWidth: 1
            },
            zlevel: 3,
            z: 10,
            data: [
              '1624-01-01', '1630-07-17', null, '1628-03-12', null, null, null, '1638-08-07', null, '1638-09-01'
            ]
          },
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            itemStyle: {
              borderColor: 'transparent',
              color: '#e4e0cf'
            },
            zlevel: 3,
            z: 20,
            data: [
              '1624-12-31', '1630-07-30', null, '1628-04-05', null, null, null, '1638-08-16', null, '1639-03-09'
            ]
          },

          // 4
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function (params) {
                const colors = [
                  'rgba(50,132,110,0.8)', 'rgba(50,132,110)', 'rgba(50,132,110)',
                  'rgba(50,132,110)', 'rgba(50,132,110,0.6)', 'rgba(35,118,183)',
                  'rgba(35,118,183,0.4)', 'rgba(35,118,183,0.6)', 'rgba(35,118,183,0.8)',
                  'rgba(35,118,183,0.3)',
                ];
                return colors[params.dataIndex];
              },
              borderColor: 'transparent',
              borderWidth: 1
            },
            zlevel: 4,
            z: 10,
            data: [
              '1636-09-19', '1632-03-14', null, '1630-08-02', null, null, null, '1639-03-18', null, '1639-03-26'
            ]
          },
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            itemStyle: {
              borderColor: 'transparent',
              color: '#e4e0cf'
            },
            zlevel: 4,
            z: 20,
            data: [
              '1636-09-25', '1632-05-08', null, '1630-08-19', null, null, null, '1639-03-22', null, '1639-09-14'
            ]
          },

          // 5
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function (params) {
                const colors = [
                  'rgba(50,132,110,0.8)', 'rgba(50,132,110)', 'rgba(50,132,110)',
                  'rgba(50,132,110)', 'rgba(50,132,110,0.6)', 'rgba(35,118,183)',
                  'rgba(35,118,183,0.4)', 'rgba(35,118,183,0.6)', 'rgba(35,118,183,0.8)',
                  'rgba(35,118,183,0.3)',
                ];
                return colors[params.dataIndex];
              },
              borderColor: 'transparent',
              borderWidth: 1
            },
            zlevel: 5,
            z: 10,
            data: [
              null, '1636-09-25', null, null, null, null, null, null, null, null
            ]
          },
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            itemStyle: {
              borderColor: 'transparent',
              color: '#e4e0cf'
            },
            zlevel: 5,
            z: 20,
            data: [
              null, '1636-10-16', null, null, null, null, null, null, null, null
            ]
          },
        ]
      };

      progressChart.setOption(option);

      const zoomSize = 6;
      progressChart.on('click', function (params) {
        const shouldntclick = 1;
        if (shouldntclick) { return; }
        progressChart.dispatchAction({
          type: 'dataZoom',
          startValue: dataAxis[Math.max(params.dataIndex - zoomSize / 2, 0)],
          endValue: dataAxis[Math.min(params.dataIndex + zoomSize / 2, dataAxis.length - 1)]
        });
      });
    }
  }
}
</script>

<style scoped>
.gantt {
  width: 70%;
  height: calc(100vh - 170px);
}

.histogram {
  width: 30%;
  height: calc(100vh - 170px);
}

.title {
  display: flex;
  margin-left: 0%;
  margin-top: 0%;
  font-size: 30px;
  font-weight: bold;
  color: rgba(50, 132, 110, 1);
  text-align: left;
  vertical-align: top;
  padding: 48px 0;
  white-space: nowrap;
}

.note-container {
  display: flex;
  align-items: center;
}

.note {
  color: var(--td-brand-color-6);
  text-align: left;
  text-indent: 2em;
  line-height: 17px;
  font-size: 14px;
}
</style>