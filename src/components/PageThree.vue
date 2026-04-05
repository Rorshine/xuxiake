<template>
  <div style="width:100%">
    <div style="display: flex; height: 136px;">
      <div class="title" style="width: 512px;visibility: hidden;">
        <p>{{ title }}</p>
      </div>
      <t-divider layout="vertical" style="height: 90%;visibility: hidden;"></t-divider>

    </div>

    <div style="display: flex;width:100%">
      <div ref="ganttChart" class="gantt" ></div>
      <div style="display: flex; justify-content: center; align-items: center; "><img :src="require('../assets/img/各省份旅行甘特图.png')" width="50%" /></div>
    </div>
  </div>
</template>

<script>

//import { color } from "d3";
import * as echarts from "echarts";
import { GridComponent } from 'echarts/components';
import { CalendarComponent } from 'echarts/components';
import { VisualMapComponent } from 'echarts/components';
import { TooltipComponent } from 'echarts/components';
import { SVGRenderer, CanvasRenderer } from 'echarts/renderers';
//import $ from 'jquery';


echarts.use([SVGRenderer, CanvasRenderer,TooltipComponent,VisualMapComponent,CalendarComponent,GridComponent]);

export default {
  name: 'PickView',
  data(){
    return{
      title:'徐霞客游历各地时间表',
      cell_ratio : 0.026836,
      
    };
  },
  mounted() {
    this.createCalendar();
    this.createGantt();
  },
  watch: {
  //监听语言是否变化，若变化调用createPieChart()
  '$i18n.locale': {
    handler() {
      // 处理语言变化的逻辑
      this.handleResize()
    },
    immediate: false // 立即执行一次回调函数
  }
},
  computed:{
    dataAxis(){
      return ['江苏','浙江','安徽','福建','江西','湖北','湖南','广西','贵州','云南'];
    },
    
  },

  methods: {
    handleResize(){
      //let myChart = this.$echarts.init(document.getElementById("calendar"), null, { renderer: 'svg' });
      //let progressChart = echarts.init(this.$refs.ganttChart, null, { renderer: 'svg' });
      this.createCalendar();
      this.createGantt();
    },
    
    createCalendar() {
      let histogram = echarts.init(document.getElementById("histogram"), null, {
        renderer: "svg",
      });

      // 数据
      const provinces = [
        "云南省",
        "广西壮族自治区",
        "湖南省",
        "江西省",
        "贵州省",
        "浙江省",
        "福建省",
        "安徽省",
        "河南省",
        "山西省",
        "湖北省",
        "陕西省",
        "江苏省",
        "河北省",
        "上海市",
      ];
      const positiveWords = [4216, 3874, 962, 640, 639, 454, 242, 173, 98, 80, 53, 32, 10, 8, 4];
      const negativeWords = [1029, 1055, 302, 156, 167, 112, 53, 34, 18, 13, 8, 10, 4, 2, 0];

      // 数据预处理：按总量排序
      const sortedData = provinces
        .map((province, index) => ({
          province,
          positive: positiveWords[index],
          negative: negativeWords[index],
          total: positiveWords[index] + negativeWords[index],
        }))
        .sort((a, b) => a.total - b.total); // 总量从大到小排序

      const sortedProvinces = sortedData.map(item => item.province);
      const sortedPositiveWords = sortedData.map(item => item.positive);
      const sortedNegativeWords = sortedData.map(item => item.negative);

      // 配置项
      let option = {
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow",
          },
        },
        legend: {
          data: ["消极情感词", "积极情感词"],
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: {
          type: "value",
          boundaryGap: [0, 0.01],
        },
        yAxis: {
          type: "category",
          data: sortedProvinces, // 按总量排序的省份
        },
        series: [
          {
            name: "消极情感词",
            type: "bar",
            stack: "总量",
            data: sortedNegativeWords, // 按总量排序的消极词数据
            itemStyle: {
              color: "#9daeb6",
            },
          },
          {
            name: "积极情感词",
            type: "bar",
            stack: "总量",
            data: sortedPositiveWords, // 按总量排序的积极词数据
            itemStyle: {
              color: "#f0b8ab",
            },
          },
        ],
      };

      // 设置图表
      histogram.setOption(option);

      // 监听窗口大小变化
      window.addEventListener("resize", function () {
        histogram.resize();
      });
    },




    createGantt() {
      let progressChart = echarts.init(this.$refs.ganttChart, null, { renderer: 'svg' });
      let dataAxis = this.dataAxis;
      let option = {
        tooltip: {
          trigger: 'item',

          axisPointer: {
            type: 'cross',
            label: {
              show: true
            },
            shadowStyle: {
              color: 'rgba(177,243,173,0.2)'
            }
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
          min: new Date('1613-03-15').getTime(), // 设置 x 轴最小值
          max: new Date('1639-12-31').getTime(), // 设置 x 轴最大值
          axisLabel: {
            formatter: function(value) {
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
//1
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function(params) {
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
            z:10,
            data: [
              null,'1613-03-31','1616-01-26','1616-02-21','1618-08-18','1623-03-11','1637-01-11','1637-04-16','1638-03-11','1638-05-10'
            ] // 结束时间
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
              null,'1613-04-14','1616-02-11','1616-03-20','1618-08-23','1623-04-09','1637-04-16','1638-03-10','1638-05-09','1638-07-15'
            ] // 开始时间
          },
//2
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function(params) {
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
            z:10,
            data: [
            '1614-01-01','1620-05-06','1618-09-04','1620-06-08','1636-10-17',null,null,'1638-05-10','1638-08-26','1638-08-16'
            ] // 结束时间
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
            '1615-12-31','1620-05-23','1618-09-06','1620-06-11','1637-01-11',null,null,'1638-05-10','1638-09-01','1638-08-26'
              
            ] // 开始时间
          },
//3
          {
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function(params) {
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
            z:10,
            data: [
            //'1624-01-01','1630-07-17','1628-03-12',null,null,null,'1638-08-07',null,'1638-09-01'
            '1624-01-01','1630-07-17',null,'1628-03-12',null,null,null,'1638-08-07',null,'1638-09-01'

            ] // 结束时间
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
              //'1624-12-31','1630-07-30','1628-04-05',null,null,null,'1638-08-16',null,'1639-03-09'
              '1624-12-31','1630-07-30',null,'1628-04-05',null,null,null,'1638-08-16',null,'1639-03-09'


            ] // 开始时间
          },
//4
{
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function(params) {
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
            z:10,
            data: [
            //'1636-09-19','1632-03-14','1630-08-02',null,null,null,'1639-03-18',null,'1639-03-26'
            '1636-09-19','1632-03-14',null,'1630-08-02',null,null,null,'1639-03-18',null,'1639-03-26'
            ] // 结束时间
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
              //'1636-09-25','1632-05-08','1630-08-19',null,null,null,'1639-03-22',null,'1639-09-26'
              '1636-09-25','1632-05-08',null,'1630-08-19',null,null,null,'1639-03-22',null,'1639-09-14'
            ] // 开始时间
          },
//5
{
            name: '持续时间',
            type: 'bar',
            stack: 'duration',
            barWidth: 10,
            itemStyle: {
              color: function(params) {
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
            z:10,
            data: [
            null,'1636-09-25',null,null,null,null,null,null,null,null

            ] // 结束时间
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
              null,'1636-10-16',null,null,null,null,null,null,null,null
              
            ] // 开始时间
          },

        ]
      };

      progressChart.setOption(option);

      // Enable data zoom when user click bar.
      const zoomSize = 6;
      progressChart.on('click', function (params) {
        const shouldntclick=1;
        if(shouldntclick){return;}
        console.log(dataAxis[Math.max(params.dataIndex - zoomSize / 2, 0)]);
        progressChart.dispatchAction({
          type: 'dataZoom',
          startValue: dataAxis[Math.max(params.dataIndex - zoomSize / 2, 0)],
          endValue:
          // eslint-disable-next-line
            dataAxis[Math.min(params.dataIndex + zoomSize / 2, data.length - 1)]
        });
      });
      // 浏览器窗口大小变化，图表大小自适应
      window.addEventListener('resize', function (){
        progressChart.resize();
      })
    }
  }
}

</script>

<style scoped>
.gantt{
  width: 70%;
  height: calc(100vh - 170px);
}
.histogram{
  width: 30%;
  height:calc(100vh - 170px);
}
.title{
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

.note-container{
  /* padding:90px 24px; */
  display:flex;
  align-items:center;
  /* background-color: var(--td-bg-color-container); */
}

.note{
  color:var(--td-brand-color-6);
  text-align:left;
  text-indent:2em;
  line-height:17px;
  font-size:14px;
}
</style>