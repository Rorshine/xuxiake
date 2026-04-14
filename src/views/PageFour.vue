<template>
    <div class="page-content">
        <!-- Image on the Left -->
        <div class="page-four-side-image">
            <img src="../assets/img/Page4content.png" alt="Image" style="max-width: 300px; height: auto;">
        </div>

        <!-- Tea Plant Component -->
        <div class="page-four-main">
            
            <div class="tea-info-container" v-if="visible">
                <!-- 图片部分 -->
                <img :src="tea_info[current].teaimg" alt="" class="tea-info-img">

                <!-- 文本内容部分 -->
                <p class="tea-info-content" v-html="tea_info[current].teacontent"></p>
            </div>

            <div style="bottom: 20px; right: 20px; z-index:99;position:absolute;">
                <p class="note">
                    The chart is inspired by
                    <t-link
                        theme="primary"
                        underline
                        href="https://cardcaptorsakura.visualcinnamon.com/&#34;"
                        target="_blank"
                    >
                        <template v-slot:prefix-icon>
                            <link-icon></link-icon>
                        </template>
                        An Ode to Cardcaptor Sakura
                    </t-link>
                </p>
            </div>
            <div id="china-map"></div>
            <!-- 圆形地图 -->
            <div
                id="chart"
                style="height: 100vh; display: flex; justify-content: center; align-items: center;"
            >
                <div id="province" style="width: 32vh; height: 32vh; z-index: 999;"></div>
            </div>
        </div>
    </div>
</template>
  
  <script>
import { defineComponent } from 'vue';
import * as echarts from 'echarts/core';
import * as d3 from 'd3';
import 'tdesign-vue-next/es/style/index.css';
//import { MessagePlugin } from 'tdesign-vue-next';
import '../assets/js/china.js';
//import chinaMap from '@/assets/json/china.json';

import pro140000 from '@/assets/json/areas_v3/bound/140000_full.json';
import pro330000 from '@/assets/json/areas_v3/bound/330000_full.json';
import pro340000 from '@/assets/json/areas_v3/bound/340000_full.json';
import pro350000 from '@/assets/json/areas_v3/bound/350000_full.json';
import pro360000 from '@/assets/json/areas_v3/bound/360000_full.json';
import pro410000 from '@/assets/json/areas_v3/bound/410000_full.json';
import pro420000 from '@/assets/json/areas_v3/bound/420000_full.json';
import pro430000 from '@/assets/json/areas_v3/bound/430000_full.json';
import pro450000 from '@/assets/json/areas_v3/bound/450000_full.json';
import pro520000 from '@/assets/json/areas_v3/bound/520000_full.json';
import pro530000 from '@/assets/json/areas_v3/bound/530000_full.json';
import pro610000 from '@/assets/json/areas_v3/bound/610000_full.json';


export default defineComponent({
    name: 'PageFour',
    components: {
        
    },
    beforeUnmount() {
        window.removeEventListener('resize', this.handleResize);
        if (this.myChart) {
            this.myChart.dispose(); // 销毁 echarts 实例
        }
        // eslint-disable-next-line vue/return-in-computed-property
    },
    mounted() {
        console.log(333333);
        const teaInfoContainer = document.querySelector('.tea-info-container');
        if (teaInfoContainer) {
            teaInfoContainer.style.display = 'none';  // 隐藏 tea-info-container
        }
        this.initChinaMap();
        this.handleResize();
        window.addEventListener('resize', this.handleResize);
        // eslint-disable-next-line vue/return-in-computed-property
    },
    methods: {
        initChinaMap() {
            this.myChart = echarts.init(document.getElementById('china-map'));
            this.updateMapOption();
        },
        updateMapOption() {
            // ... method content remains the same
            const option = {
                series: [
                    {
                        name: '中国',
                        type: 'map',
                        mapType: 'china',
                        selectedMode: 'none', // 取消点击效果
                        roam: false, // 禁止缩放和平移
                        label: {
                            emphasis: {
                                show: false,
                                color: '#000',
                            },
                        },
                        itemStyle: {
                            normal: {
                                areaColor: '#8FBC8F', // 设置地图的初始颜色
                                borderColor: '#DEFF9B', // 设置地图边界线颜色
                            },
                            emphasis: { show: false }, // 清除鼠标悬停效果
                        },
                        hoverLink: false, // 禁用悬停链接
                        data: [],
                    },
                ],
            };
            const highlightedProvinces = [
                '山西',
                '贵州',
                '江西',
                '安徽',
                '浙江',
                '陕西',
                '湖南',
                '湖北',
                '广西',
                '福建',
                '云南',
            ];
            option.series[0].data = highlightedProvinces.map((province) => {
                return {
                    name: province,
                    itemStyle: { areaColor: '#2E8B57' },
                    label: {
                        show: true, // 显示标签
                        formatter: '{b}', // 标签内容为省份名称
                        color: 'black', // 标签文字颜色
                        fontSize: 14, // 标签文字大小
                        fontWeight: 'bold', // 标签文字加粗
                    },
                };
            });
            this.myChart.setOption(option);
        },
        
        handleResize() {
            this.windowWidth = window.innerWidth;
            this.windowHeight = window.innerHeight;
            this.visible = false;
            this.removeChart();
            this.createPieChart();
        },
        /*handle_guide_click() {
            console.log('click');
            this.guide_current = 0;
        },*/
        handleChange(current) {
            
            if (current === 3) {
                this.createProvinceChart();
            }
            this.clearProvinceChart(); // Moved outside the if statement
        },
        handleFinish() {
            console.log('finish');
            this.visible = false;
            this.clearProvinceChart();
        },
        clearProvinceChart() {
            var chartDom = document.getElementById('province');
            var myChart = echarts.init(chartDom);
            myChart.clear();
        },
        createProvinceChart(index) {
            let province_geo_data = [
                {
                    name: 'ZJ',
                    province: '浙江省',
                    value: [
                        {
                            marker: [120.107334, 30.220546],

                            color: '#D0DC89',
                            type: '江郎山',
                        },
                        {
                            marker: [121.007624, 29.221509],

                            color: '#D0DC89',
                            type: '雁荡山',
                        },
                        {
                            marker: [121.007624, 29.221509],

                            color: '#D0DC89',
                            type: '天台山',
                        },
                    ],
                    path: pro330000,
                    legend: [
                        { name: '江郎山', color: '#D0DC89' },
                        { name: '雁荡山', color: '#D0DC89' },
                        { name: '天台山', color: '#D0DC89' },
                    ],
                },
                {
                    name: 'AH',
                    province: '安徽省',
                    value: [
                        {
                            marker: [118.41359, 29.86076],
                            location_name: '黄山白茶-安徽省歙县',
                            color: '#D9D9D9',
                            type: '白岳山',
                        },
                        {
                            marker: [118.14161, 30.27296],
                            location_name: '黄山毛峰-安徽省黄山',
                            color: '#D0DC89',
                            type: '黄山',
                        },
                    ],
                    path: pro340000,
                    legend: [
                        { name: '白岳山', color: '#D9D9D9' },
                        { name: '黄山', color: '#D0DC89' },
                    ],
                },
                {
                    name: 'FJ',
                    province: '福建省',
                    value: [
                        {
                            marker: [120.21671, 27.32436],
                            location_name:
                                '白毫银针、白牡丹、寿眉-福建省福鼎市',
                            color: '#D9D9D9',
                            type: '武夷山',
                        },
                        {
                            marker: [118.7854, 27.52629],
                            location_name: '白毫银针、寿眉、贡眉-福建省松溪县',
                            color: '#D9D9D9',
                            type: '石竹山',
                        },
                    ],
                    path: pro350000,
                    legend: [
                        { name: '武夷山', color: '#D9D9D9' },
                        { name: '石竹山', color: '#C54522' },
                    ],
                },
                {
                    name: 'JS',
                    province: '江西省',
                    value: [
                        {
                            marker: [120.63132, 31.30227],
                            location_name: '碧螺春-江苏省苏州市',
                            color: '#D0DC89',
                            type: '庐山',
                        },
                    ],
                    path: pro360000,
                    legend: [{ name: '庐山', color: '#D0DC89' }],
                },
                {
                    name: 'HN2',
                    province: '河南省',
                    value: [
                        {
                            marker: [103.046, 30.106],
                            location_name: '蒙顶黄牙-四川省雅安市蒙顶山',
                            color: '#E9CB6A',
                            type: '嵩山',
                        },
                    ],
                    path: pro410000,
                    legend: [{ name: '嵩山', color: '#E9CB6A' }],
                },
                {
                    name: 'SX',
                    province: '陕西省',
                    value: [
                        {
                            marker: [108.842623, 34.527114],
                            location_name: '泾阳茯砖茶-陕西省泾阳县',
                            color: '#594445',
                            type: '华山',
                        },
                    ],
                    path: pro610000,
                    legend: [{ name: '华山', color: '#594445' }],
                },
                {
                    name: 'HB',
                    province: '湖北省',
                    value: [
                        {
                            marker: [109.48817, 30.27217],
                            location_name: '玉露-湖北省恩施市',
                            color: '#D0DC89',
                            type: '武当山',
                        },
                    ],
                    path: pro420000,
                    legend: [{ name: '武当山', color: '#D0DC89' }],
                },
                {
                    name: 'SX2',
                    province: '山西省',
                    value: [
                        {
                            marker: [113.597302, 24.811112],
                            location_name: '广东大叶青-广东省韶关市',
                            color: '#E9CB6A',
                            type: '五台山',
                        },
                        {
                            marker: [113.597302, 24.811112],
                            location_name: '广东大叶青-广东省韶关市',
                            color: '#E9CB6A',
                            type:'恒山',
                        }
                    ],
                    path: pro140000,
                    legend: [
                        { name: '五台山', color: '#E9CB6A' },
                        { name: '恒山', color: '#E9CB6A' },
                    ],
                },
                {
                    name: 'HN',
                    province: '湖南省',
                    value: [
                        {
                            marker: [111.212846, 28.374107],
                            location_name:
                                '安化黑茶、千两茶-湖南省益阳市安化县',
                            color: '#594445',
                            type: '茶陵州',
                        },
                        
                        {
                            marker: [113.152243, 29.356149],
                            location_name: '北港毛尖-湖南省岳阳市北港',
                            color: '#E9CB6A',
                            type: '石鼓书院',
                        },
                        {
                            marker: [113.152243, 29.356149],
                            location_name: '北港毛尖-湖南省岳阳市北港',
                            color: '#E9CB6A',
                            type: '浯溪碑林',
                        },
                        {
                            marker: [113.152243, 29.356149],
                            location_name: '北港毛尖-湖南省岳阳市北港',
                            color: '#E9CB6A',
                            type: '永州八景',
                        },
                        {
                            marker: [113.152243, 29.356149],
                            location_name: '北港毛尖-湖南省岳阳市北港',
                            color: '#E9CB6A',
                            type: '濂溪故里',
                        },
                        {
                            marker: [113.152243, 29.356149],
                            location_name: '北港毛尖-湖南省岳阳市北港',
                            color: '#E9CB6A',
                            type: '舜帝陵',
                        },
                        {
                            marker: [113.152243, 29.356149],
                            location_name: '北港毛尖-湖南省岳阳市北港',
                            color: '#E9CB6A',
                            type: '石门龙洞',
                        },
                    ],
                    path: pro430000,
                    legend: [
                        { name: '茶陵州', color: '#594445' },
                        { name: '衡阳城', color: '#E9CB6A' },
                        { name: '石鼓书院', color: '#E9CB6A' },
                        { name: '浯溪碑林', color: '#E9CB6A' },
                        { name: '永州八景', color: '#E9CB6A' },
                        { name: '濂溪故里', color: '#E9CB6A' },
                        { name: '舜帝陵', color: '#E9CB6A' },
                        { name: '石门龙洞', color: '#E9CB6A' },
                    ],
                },
                {
                    name: 'GX',
                    province: '广西壮族自治区',
                    value: [
                        {
                            marker: [111.380906, 23.812374],
                            location_name: '六堡茶-广西壮族自治区梧州市六堡镇',
                            color: '#594445',
                            type: '炎井温泉',
                        },
                        {
                            marker: [110.299121, 25.274215],
                            location_name: '六堡茶-广西壮族自治区桂林市阳朔县',
                            color: '#594445',
                            type: '灵渠',
                        },
                        {
                            marker: [110.299121, 25.274215],
                            location_name: '六堡茶-广西壮族自治区桂林市阳朔县',
                            color: '#594445',
                            type: '桂林十景',
                        },
                        {
                            marker: [110.299121, 25.274215],
                            location_name: '六堡茶-广西壮族自治区桂林市阳朔县',
                            color: '#594445',
                            type: '麒麟山',
                        },
                        {
                            marker: [110.299121, 25.274215],
                            location_name: '六堡茶-广西壮族自治区桂林市阳朔县',
                            color: '#594445',
                            type: '邕州八景',
                        },
                        {
                            marker: [110.299121, 25.274215],
                            location_name: '六堡茶-广西壮族自治区桂林市阳朔县',
                            color: '#594445',
                            type: '佛子山',
                        },
                    ],
                    path: pro450000,
                    legend: [
                        { name: '炎井温泉', color: '#594445' },
                        { name: '灵渠', color: '#594445' },
                        { name: '桂林十景', color: '#594445' },
                        { name: '麒麟山', color: '#594445' },
                        { name: '邕州八景', color: '#594445' },
                        { name: '佛子山', color: '#594445' },
                    ],
                },
            
                {
                    name: 'GZ',
                    province: '贵州省',
                    value: [
                        {
                            marker: [105.613174, 27.141682],
                            location_name: '海马宫茶-贵州省大方县海马宫村',
                            color: '#E9CB6A',
                            type: '白云山',
                        },
                    ],
                    path: pro520000,
                    legend: [{ name: '白云山', color: '#E9CB6A' }],
                },
                {
                    name: 'YN',
                    province: '云南省',
                    value: [
                        {
                            marker: [100.97692, 22.78684],
                            location_name: '滇青-云南省思茅市',
                            color: '#D0DC89',
                            type: '鸡足山',
                        },
                        {
                            marker: [100.79739, 22.00749],
                            location_name: '滇青-云南省西双版纳',
                            color: '#D0DC89',
                            type: '黄龙潭',
                        },
                        {
                            marker: [100.08233, 23.89516],
                            location_name: '滇青-云南省临沧市',
                            color: '#D0DC89',
                            type: '澜沧江',
                        },
                    ],
                    path: pro530000,
                    legend: [
                        { name: '鸡足山', color: '#D0DC89' },
                        { name: '黄龙潭', color: '#D0DC89' },
                        { name: '澜沧江', color: '#D0DC89' },
                    ],
                },
                
                
            ];



            this.clearProvinceChart();
            var chartDom = document.getElementById('province');
            var myChart = echarts.init(chartDom);

            //var path = province_geo_data[index].path;
            //var _this = this

        

            var localMapData = province_geo_data[index].path; // 假设 path 已经是导入的 JSON 数据

            // 直接注册地图
            echarts.registerMap(province_geo_data[index].name, localMapData);

            // 设置图表选项
            var option = {
                title: {
                    text: province_geo_data[index].province,
                    left: 'center',
                },
                tooltip: {
                    show: true,
                    trigger: 'item',
                    showDelay: 0,
                    transitionDuration: 0.2,
                    formatter: function (params) {
                        return params.data.name;
                    },
                },
                legend: {
                    data: province_geo_data[index].legend.map(function (item) {
                        return {
                            name: item.name,
                            itemStyle: {
                                color: item.color,
                            },
                        };
                    }),
                    left: 'center',
                    bottom: 5,
                },
                geo: {
                    map: province_geo_data[index].name,
                    itemStyle: {
                        areaColor: {
                            type: 'linear', // 设置渐变色
                            x: 0,
                            y: 0,
                            x2: 0.5,
                            y2: 1,
                            colorStops: [
                                { offset: 0, color: '#5eb139' },
                                { offset: 1, color: '#2d8241' },
                            ],
                        },
                        borderColor: '#ffecb2',
                        borderWidth: 0.5,
                    },
                    tooltip: {
                        show: false,
                    },
                    emphasis: {
                        itemStyle: {
                            areaColor: {
                                type: 'linear',
                                x: 0,
                                y: 0,
                                x2: 0.5,
                                y2: 1,
                                colorStops: [
                                    { offset: 0, color: '#5eb139' },
                                    { offset: 1, color: '#2d8241' },
                                ],
                            },
                            borderColor: '#ffecb2',
                            borderWidth: 0.5,
                        },
                        label: {
                            show: false,
                        },
                        tooltip: {
                            show: false,
                        },
                    },
                },
            };

            myChart.setOption(option);
        },
        removeChart() {
            d3.select('#chart').selectAll('*').remove();
        },
        createPieChart() {
            const _this = this;

            // 定义变量
            var color = d3
                .scaleOrdinal()
                .range([
                    '#D0DC89',
                    '#D9D9D9',
                    '#E9CB6A',
                    '#EC8924',
                    '#C54522',
                    '#594445',
                ]);
            var province_color = d3
                .scaleOrdinal()
                .range([
                    '#FF7043',
                    '#FFC107',
                    '#CDDC39',
                    '#8BC34A',
                    '#4CAF50',
                    '#009688',
                    '#00BCD4',
                    '#03A9F4',
                    '#3F51B5',
                    '#673AB7',
                    '#E91E63',
                    '#F44336',
                ]);
            var data = [
                ['green', 0],
                ['white', 11],
                ['yellow', 2],
                ['wulong', 17],
                ['red', 0],
                ['black', 0],
            ];

            var name_data = this.name_data;
            var province_name = this.province_name;

            var name_data_center_angle = [];
            var province_name_center_angle = [];

            var margin = { top: 0, right: 0, bottom: 0, left: 0 },
                width = this.windowWidth - margin.right - margin.left,
                height = this.windowHeight - margin.top - margin.bottom;

            var province_geo_scale = 300 / 396.5;

            var size, radius;

            if (width < height) {
                size = width;
                radius = width / 2 - 80;
            } else {
                size = height;
                radius = height / 2 - 80;
            }

            radius = height / 2 - 80;

            var cover_alpha = 0.3;

            // 线条粗细缩放因子
            var line_scale = radius * (1 / 397.5);

            var scale_rad_max = 0.32;
            var scale_rad_min = 0.255;

            var rad_line_max = scale_rad_max,
                rad_line_min = scale_rad_min,
                pi2 = Math.PI * 2;

            //var basewidth = 1600;
            var mouse_over_in_action = false;
            var province_click_current = -1;
            var name_click_current = -1;

            //var size_factor = this.windowWidth / basewidth;

            // 设置省份的宽度
            //document.getElementById('province').setAttribute('style','width='+(province_geo_scale*radius)+'px;height='+(province_geo_scale*radius)+'px')
            d3.select('#province').attr(
                'style',
                'width=' +
                    province_geo_scale * radius +
                    'px;height=' +
                    province_geo_scale * radius +
                    'px'
            );

            console.log(123);
            var container = d3.select('#chart');
            var canvas = container
                .append('canvas')
                .attr('id', 'canvas-target')
                .attr('class', 'canvas-target')
                .attr('style', 'position:absolute;top:0;');
            var ctx = canvas.node().getContext('2d');
            canvas
                .attr('width', size)
                .attr('height', size)
                .style('width', size + 'px')
                .style('height', size + 'px');
            ctx.scale(1, 1);

            ctx.translate(size / 2, size / 2);
            //canvas设置
            ctx.globalCompositeOperation = 'multiply';
            ctx.lineCap = 'round';

            // 将地图放进去
            container
                .append('div')
                .attr('id', 'province')
                .attr(
                    'style',
                    'width: 32vh;height: 32vh;z-index: 999;position:absolute;top: 50%; transform: translateY(-50%);'
                );

            var arc = d3
                .arc()
                .outerRadius(radius - 10)
                .innerRadius(radius - 20)
                .cornerRadius(5);

            // var name_label_group = chart.append("g").attr('class','name-group')

            var name_label_arc = d3
                .arc()
                .outerRadius(radius - 26)
                .innerRadius(radius - 42)
                .cornerRadius(7);

            var province_label_arc = d3
                .arc()
                .outerRadius(0.5 * radius)
                .innerRadius(0.5 * radius - 10)
                .cornerRadius(5);

            var pie = d3
                .pie()
                .sort(null)
                .padAngle(0.01)
                .value(function (d) {
                    return d[1];
                });

            var name_pie = d3
                .pie()
                .sort(null)
                .padAngle(0.01)
                .value(function (d) {
                    return d[1];
                });

            var province_pie = d3
                .pie()
                .sort(null)
                .padAngle(0.01)
                .value(function (d) {
                    return d[1];
                });

            var svg = container
                .append('svg')
                .attr('id', 'main-svg')
                .attr('class', 'main-svg')
                .attr('style', 'position:absolute;top:0;')
                .attr('width', size)
                .attr('height', size)
                .append('g')
                .attr('id', 'outer-ring')
                .attr(
                    'transform',
                    'translate(' + size / 2 + ',' + size / 2 + ')'
                );

            var inner_ring = d3
                .select('#main-svg')
                .append('g')
                .attr('id', 'inner-ring')
                .attr(
                    'transform',
                    'translate(' + size / 2 + ',' + size / 2 + ')'
                );

            var g = svg
                .selectAll('.arc')
                .data(pie(data))
                .enter()
                .append('g')
                .attr('class', 'arc');

            var name_g = svg
                .selectAll('.name-arc')
                .data(name_pie(name_data))
                .enter()
                .append('g')
                .attr('class', 'name-arc');

            var province_name_g = inner_ring
                .selectAll('.province-arc')
                .data(province_pie(province_name))
                .enter()
                .append('g')
                .attr('class', 'province-arc');

            g.append('path')
                .attr('d', arc)
                .style('fill', function (d) {
                    return color(d.data[0]);
                });

            name_g
                .append('path')
                .attr('d', name_label_arc)
                .attr('class', 'name-pie')
                .attr('id', function (d, i) {
                    return 'name-pie-' + i;
                })
                .style('fill', '#00000000')
                .attr('stroke', '#00000014')
                .attr('stroke-width', '2');

            name_g
                .append('circle')
                .attr('class', 'name-label-circle')
                .attr('id', function (d, i) {
                    return 'name-label-circle-' + i;
                })
                .attr('r', 5)
                .attr('fill', '#00000014')
                .attr('stroke', 'white')
                .attr('stroke-width', '2')
                .attr('transform', function (d) {
                    var centerAngle = (d.startAngle + d.endAngle) / 2;
                    var finalAngle = centerAngle;
                    // let deg = 90-(d.endAngle*180)/(2*Math.PI)-(d.startAngle*180)/(2*Math.PI)+6

                    return (
                        'rotate(' +
                        ((finalAngle * 180) / Math.PI - 90) +
                        ')' +
                        'translate(' +
                        (radius - 42) +
                        ')'
                    );
                })
                .style('z-index', 1);

            name_g
                .append('text')
                .attr('class', 'name-label')
                .attr('id', function (d, i) {
                    return 'name-label-' + i;
                })
                .attr('dy', '0.35em')
                .attr('transform', function (d) {
                    name_data_center_angle.push(
                        (d.startAngle + d.endAngle) / 2
                    );
                    var centerAngle = (d.startAngle + d.endAngle) / 2;
                    var finalAngle = centerAngle;
                    return (
                        'rotate(' +
                        ((finalAngle * 180) / Math.PI - 90) +
                        ')' +
                        'translate(' +
                        radius +
                        ')' +
                        ((finalAngle > 0) & (finalAngle < Math.PI)
                            ? ''
                            : 'rotate(180)')
                    );
                })
                .attr('text-anchor', (d) =>
                    (d.endAngle + d.startAngle) / 2 > Math.PI ? 'end' : null
                )
                .style('font-size', '12px')
                .style('font-weight', '600')
                .attr('fill', 'rgba(0,0,0,0.5)')
                .text(function (d) {
                    return d.data[0];
                })
                .style('z-index', 50);

            province_name_g
                .append('path')
                .attr('d', province_label_arc)
                .style('fill', function (d) {
                    return province_color(d.index);
                });

            province_name_g
                .append('text')
                .attr('class', 'province-label')
                .attr('id', function (d, i) {
                    return 'province-label-' + i;
                })
                .attr('dy', '0.35em')
                // .attr("transform", (d) => `translate(${name_label_arc.centroid(d)})`)
                .attr('transform', function (d) {
                    var centerAngle = (d.startAngle + d.endAngle) / 2;
                    province_name_center_angle.push(centerAngle);
                    var finalAngle = centerAngle;
                    // let deg = 90-(d.endAngle*180)/(2*Math.PI)-(d.startAngle*180)/(2*Math.PI)
                    // console.log(deg)
                    return (
                        'rotate(' +
                        ((finalAngle * 180) / Math.PI - 90) +
                        ')' +
                        'translate(' +
                        (0.5 * radius + 5) +
                        ')' +
                        ((finalAngle > 0) & (finalAngle < Math.PI)
                            ? ''
                            : 'rotate(180)')
                    );
                })

                .attr('text-anchor', (d) =>
                    (d.endAngle + d.startAngle) / 2 > Math.PI ? 'end' : null
                )
                .style('font-size', '12px')
                .style('font-weight', '600')
                .attr('fill', 'rgba(0,0,0,0.5)')
                .text(function (d) {
                    return d.data[0];
                });

            province_name_g
                .append('circle')
                .attr('class', 'province-label-circle')
                .attr('id', function (d, i) {
                    return 'province-label-circle' + i;
                })
                .attr('r', 5)
                .attr('fill', function (d, i) {
                    return province_color(i);
                })
                .attr('stroke', 'white')
                .attr('stroke-width', '2')
                .attr('transform', function (d) {
                    var centerAngle = (d.startAngle + d.endAngle) / 2;
                    var finalAngle = centerAngle;
                    return (
                        'rotate(' +
                        ((finalAngle * 180) / Math.PI - 90) +
                        ')' +
                        'translate(' +
                        (0.5 * radius + 25) +
                        ')'
                    );
                });

            var cover_data = [
                //浙江1
                [
                    {
                        source_a: 0,
                        target_a: 0,
                        strokeStyle: province_color(0),
                    },
                ],
                [
                    {
                        source_a: 0,
                        target_a: 1,
                        strokeStyle: province_color(0),
                    },
                ],
                //安徽
                [
                    {
                        source_a: 1,
                        target_a: 2,
                        strokeStyle: province_color(1),
                    },
                ],
                [
                    {
                        source_a: 1,
                        target_a: 3,
                        strokeStyle: province_color(1),
                    },
                ],
                //福建1
                [
                    {
                        source_a: 2,
                        target_a: 4,
                        strokeStyle: province_color(2),
                    },
                ],
                //江西
                [
                    {
                        source_a: 3,
                        target_a: 5,
                        strokeStyle: province_color(3),
                    },
                ],
                //浙江2
                [
                    {
                        source_a: 0,
                        target_a: 6,
                        strokeStyle: province_color(0),
                    },
                ],
                //福建2
                [
                    {
                        source_a: 2,
                        target_a: 7,
                        strokeStyle: province_color(2),
                    },
                ],
                //河南
                [
                    {
                        source_a: 4,
                        target_a: 8,
                        strokeStyle: province_color(4),
                    },
                ],
                //陕西
                [
                    {
                        source_a: 5,
                        target_a: 9,
                        strokeStyle: province_color(5),
                    },
                ],
                //湖北
                [
                    {
                        source_a: 6,
                        target_a: 10,
                        strokeStyle: province_color(6),
                    },
                ],
                //山西
                [
                    {
                        source_a: 7,
                        target_a: 11,
                        strokeStyle: province_color(7),
                    },
                ],

                [
                    {
                        source_a: 7,
                        target_a: 12,
                        strokeStyle: province_color(7),
                    },
                ],

                //湖南
                [
                    {
                        source_a: 8,
                        target_a: 13,
                        strokeStyle: province_color(8),
                    },
                ],
                //云南
                [
                    {
                        source_a: 8,
                        target_a: 14,
                        strokeStyle: province_color(8),
                    },
                ],
                [
                    {
                        source_a: 8,
                        target_a: 15,
                        strokeStyle: province_color(8),
                    },
                ],
                [
                    {
                        source_a: 8,
                        target_a: 16,
                        strokeStyle: province_color(8),
                    },
                ],

                [
                    {
                        source_a: 8,
                        target_a: 17,
                        strokeStyle: province_color(8),
                    },
                ],
                [
                    {
                        source_a: 8,
                        target_a: 18,
                        strokeStyle: province_color(8),
                    },
                ],
                [
                    {
                        source_a: 8,
                        target_a: 19,
                        strokeStyle: province_color(8),
                    },
                ],
                //广西
                [
                    {
                        source_a: 9,
                        target_a: 20,
                        strokeStyle: province_color(9),
                    },
                ],
                [
                    {
                        source_a: 9,
                        target_a: 21,
                        strokeStyle: province_color(9),
                    },
                ],
                [
                    {
                        source_a: 9,
                        target_a: 22,
                        strokeStyle: province_color(9),
                    },
                ],

                [
                    {
                        source_a: 9,
                        target_a: 23,
                        strokeStyle: province_color(9),
                    },
                ],

                [
                    {
                        source_a: 9,
                        target_a: 24,
                        strokeStyle: province_color(9),
                    },
                ],
                //贵州
                [
                    {
                        source_a: 10,
                        target_a: 25,
                        strokeStyle: province_color(10),
                    },
                ],
                
                [
                    {
                        source_a: 10,
                        target_a: 26,
                        strokeStyle: province_color(10),
                    },
                ],

                //云南
                [
                    {
                        source_a: 11,
                        target_a: 27,
                        strokeStyle: province_color(11),
                    },
                ],
                
                [
                    {
                        source_a: 11,
                        target_a: 28,
                        strokeStyle: province_color(11),
                    },
                ],
                [
                    {
                        source_a: 11,
                        target_a: 29,
                        strokeStyle: province_color(11),
                    },
                ],
                [
                    {
                        source_a: 11,
                        target_a: 30,
                        strokeStyle: province_color(11),
                    },
                ],
            ];

            // console.log(name_data_center_angle)
            // console.log(province_name_center_angle)

            // 画出线条
            var line = d3
                .lineRadial()
                .angle(function (d) {
                    return d.angle;
                })
                .radius(function (d) {
                    return d.radius;
                })
                .curve(d3.curveBasis)
                .context(ctx);

            ctx.globalAlpha = cover_alpha;
            create_lines('character', cover_data);

            // 画出线条的函数
            function create_lines(type, data) {
                for (var i = 0; i < data.length; i++) {
                    var d = data[i];
                    var line_data = [];

                    // 首先，函数提取出当前元素的一些信息，如source_a和source_r表示线条的起始角度和半径，target_a表示线条的目标角度，target_r表示线条的目标半径。
                    var source_a = province_name_center_angle[d[0].source_a],
                        source_r = 0.5 * radius + 25;
                    var target_a = name_data_center_angle[d[0].target_a],
                        target_r = radius - 42;

                    // console.log(source_a,target_r)

                    // 接下来，根据起始角度和目标角度的差值，确定线条的走向和弧度大小。根据差值的不同情况，确定了线条的走向（顺时针或逆时针）和弧度大小。
                    //Figure out some variable that will determine the path points to create
                    var side, da, angle_sign;
                    if (target_a - source_a < -Math.PI) {
                        // 当目标角度和起始角度插值小于-pi即小于-180度 cw
                        side = 'cw';
                        da = 2 + (target_a - source_a) / Math.PI;
                        angle_sign = 1;
                    } else if (target_a - source_a < 0) {
                        // 当目标角度和起始角度插值小于0
                        side = 'ccw';
                        da = (source_a - target_a) / Math.PI;
                        angle_sign = -1;
                    } else if (target_a - source_a < Math.PI) {
                        side = 'cw';
                        da = (target_a - source_a) / Math.PI;
                        angle_sign = 1;
                    } else {
                        side = 'ccw';
                        da = 2 - (target_a - source_a) / Math.PI;
                        angle_sign = -1;
                    }

                    // 然后，根据线条类型和弧度大小，计算出线条中间弧线部分的半径。
                    //Calculate the radius of the middle arcing section of the line
                    var range =
                        type === 'character'
                            ? [rad_line_max, rad_line_min]
                            : [rad_line_min, rad_line_max];
                    var scale_rad_curve = d3
                        .scaleLinear()
                        .domain([0, 1])
                        .range(range);
                    var rad_curve_line = scale_rad_curve(da) * size;

                    // 然后，稍微偏移起始点和结束点的角度，以使线条有一定的曲线效果。
                    //Slightly offset the first point on the curve from the source
                    //var range = type === "character" ? [0.0, 0.07] : [0, 0.03];
                    var scale_angle_start_offset = d3
                        .scaleLinear()
                        .domain([0, 1])
                        .range(range);
                    var start_angle =
                        source_a +
                        angle_sign * scale_angle_start_offset(da) * Math.PI;

                    //Slightly offset the last point on the curve from the target
                    var range2 = type === 'character' ? [0, 0.02] : [0.0, 0.07];
                    var scale_angle_end_offset = d3
                        .scaleLinear()
                        .domain([0, 1])
                        .range(range2);
                    var end_angle =
                        target_a -
                        angle_sign * scale_angle_end_offset(da) * Math.PI;

                    // 接下来，根据弧度大小和步长，计算出曲线部分的点的数量，并根据线条走向和步长逐渐增加或减小角度，并将点的信息（角度和半径）添加到line_data数组中。
                    var da_inner;
                    if (target_a - source_a < -Math.PI) {
                        da_inner = pi2 + (end_angle - start_angle);
                    } else if (target_a - source_a < 0) {
                        da_inner = start_angle - end_angle;
                    } else if (target_a - source_a < Math.PI) {
                        da_inner = end_angle - start_angle;
                    } else if (target_a - source_a < 2 * Math.PI) {
                        da_inner = pi2 - (end_angle - start_angle);
                    }

                    //Attach first point to data
                    line_data.push({
                        angle: source_a,
                        radius: source_r,
                    });

                    //Attach first point of the curve section
                    line_data.push({
                        angle: start_angle,
                        radius: rad_curve_line,
                    });

                    //Create points in between for the curve line
                    var step = 0.06;
                    var n = Math.abs(Math.floor(da_inner / step));
                    var curve_angle = start_angle;
                    var sign = side === 'cw' ? 1 : -1;
                    if (n >= 1) {
                        for (var j = 0; j < n; j++) {
                            curve_angle += (sign * step) % pi2;
                            line_data.push({
                                angle: curve_angle,
                                radius: rad_curve_line,
                            });
                        } //for j
                    } //if

                    //Attach last point of the curve section
                    line_data.push({
                        angle: end_angle,
                        radius: rad_curve_line,
                    });

                    //Attach last point to data
                    line_data.push({
                        angle: target_a,
                        radius: target_r,
                    });

                    //Draw the path
                    ctx.beginPath();
                    line(line_data);
                    ctx.strokeStyle = d[0].strokeStyle;
                    ctx.lineWidth = 3 * line_scale;
                    ctx.stroke();
                } //for

                ctx.globalAlpha = 0.3;
                ctx.lineWidth = 3 * line_scale;
            } //function create_lines

            // 创建省份的hoverpie
            var hover_province_arc = d3
                .arc()
                .outerRadius(0.5 * radius + 30)
                .innerRadius(0.5 * radius - 10);

            var hover_province_pie = d3
                .pie()
                .sort(null)
                .padAngle(0)
                .value(function (d) {
                    return d[1];
                });

            var hover_province_arcs = hover_province_pie(province_name);
            hover_province_arcs.forEach(function (d, i) {
                d.province = i;
            });

            var province_hover_g = inner_ring
                .append('g')
                .attr('class', 'province-hover-arc-g');
            // eslint-disable-next-line
            var province_hover = province_hover_g
                .selectAll('.province-hover-arc')
                .data(hover_province_arcs)
                .enter()
                .append('path')
                .attr('class', 'province-hover-arc')
                .attr('d', hover_province_arc)
                .style('fill', 'none')
                .style('pointer-events', 'all')
                .on('click', mouse_click_province)
                .on('mouseover', mouse_over_province)
                .on('mouseout', mouse_out);

            d3.selectAll('.name-pie')
                .on('mouseover', mouse_over_name)
                .on('mouseout', mouse_out)
                .on('click', mouse_click_name);

            function mouse_click_province(d, i) {
                console.log(d, i, 'provinceclick');
                d.stopPropagation();
                name_click_current = -1;

                    _this.visible=false;

                // 上次点击的和这次点击的不同
                if (province_click_current != i.province) {
                    province_click_current = i.province;
                    //清空目前的曲线和外圈的圆圈
                    clear_outer_ring();
                    ctx.clearRect(-width / 2, -height / 2, width, height);
                    // console.log(i)

                    create_province_current_selected_line(i.province);
                    _this.createProvinceChart(i.province);


                    // 中间显示地图
                    // ………………
                } else {
                    // 上次点击的和这次点击的相同
                    //清空目前的曲线和外圈的圆圈
                    clear_outer_ring();
                    ctx.clearRect(-width / 2, -height / 2, width, height);
                    // 清空中间的地图
                    _this.clearProvinceChart();

                    // 画出线条
                    create_lines('character', cover_data);

                    
                    province_click_current = -1;
                }
            }

            function clear_outer_ring() {
                // 高亮外圈的茶叶
                for (var i = 0; i < 30; i++) {
                    d3.select('#name-pie-' + i).style('fill', '#00000000');
                    d3.select('#name-label-circle-' + i).style(
                        'fill',
                        '#00000014'
                    );
                }
            }

            function mouse_over_name(d, i) {
                d.stopPropagation();
                // console.log(i.index);
                mouse_over_in_action = true;
                ctx.clearRect(-width / 2, -height / 2, width, height);
                clear_outer_ring();
                var line_data = cover_data.filter(function (c) {
                    return c[0].target_a === i.index;
                });
                // console.log(line_data);
                create_lines('character', line_data);
                d3.select('#name-pie-' + i.index).style(
                    'fill',
                    line_data[0][0].strokeStyle
                );
                d3.select('#name-label-circle-' + i.index).style(
                    'fill',
                    line_data[0][0].strokeStyle
                );
            }

            function mouse_out() {
                // 获取 .tea-info-container 元素
                const teaInfoContainer = document.querySelector('.tea-info-container');
                if (!mouse_over_in_action) return;
                mouse_over_in_action = false;
                ctx.clearRect(-width / 2, -height / 2, width, height);
                clear_outer_ring();

                // 如果当前有外圈的显示，只显示线+圆
                if (name_click_current != -1) {
                    var line_data = cover_data.filter(function (c) {
                        return c[0].target_a === name_click_current;
                    });
                    
                    ctx.globalAlpha = 0.8;
                    create_lines('character', line_data);
                    d3.select('#name-pie-' + name_click_current).style(
                        'fill',
                        line_data[0][0].strokeStyle
                    );
                    d3.select('#name-label-circle-' + name_click_current).style(
                        'fill',
                        line_data[0][0].strokeStyle
                    );
                    return;
                }

                // 如果上次选了省份，移除后应该仍然显示省份的茶叶数据
                if (province_click_current != -1) {
                    // 恢复显示 .tea-info-container 部分
                        if (teaInfoContainer) {
                            teaInfoContainer.style.display = 'block';  // 恢复显示 .tea-info-container
                        }
                    create_province_current_selected_line(
                        province_click_current
                    );
                    return;
                }
                // 否则，保持不变

                create_lines('character', cover_data);
            }

            function create_province_current_selected_line(index) {
                // 画出要用的曲线
                ctx.globalAlpha = 0.8;

                var line_data = cover_data.filter(function (c) {
                    return c[0].source_a === index;
                });
                var line_color = line_data[0][0].strokeStyle;
                create_lines('character', line_data);

                // 高亮外圈的茶叶
                for (var i = 0; i < line_data.length; i++) {
                    var target_index = line_data[i][0].target_a;
                    // console.log(target_index);
                    d3.select('#name-pie-' + target_index).style(
                        'fill',
                        line_color
                    );
                    d3.select('#name-label-circle-' + target_index).style(
                        'fill',
                        line_color
                    );

                    // console.log(el)
                }
            }

            function mouse_click_name(d, i) {
                d.stopPropagation();
 
                // 若之前选择的茶和现在选择的茶不同，点击后显示该茶,省份选择数据清空
                console.log(d, i, 'menuclick');
                // 获取 .tea-info-container 元素
                    const teaInfoContainer = document.querySelector('.tea-info-container');
                if (name_click_current != i.index) {
                    province_click_current = -1;
                    name_click_current = i.index;
                    clear_outer_ring();
                    var line_data = cover_data.filter(function (c) {
                        return c[0].target_a === i.index;
                    });
                    ctx.globalAlpha = 0.8;
                    create_lines('character', line_data);
                    d3.select('#name-pie-' + i.index).style(
                        'fill',
                        line_data[0][0].strokeStyle
                    );
                    d3.select('#name-label-circle-' + i.index).style(
                        'fill',
                        line_data[0][0].strokeStyle
                    );

                    _this.current = name_click_current;
                    _this.visible = true;

                    // 显示茶叶信息部分
                    if (teaInfoContainer) {
                        teaInfoContainer.style.display = 'block';  // 显示 tea_info_container
                    }

                    console.log(this.visible);
                    console.log(i?.data[0], 'i?.data[0]');
                    console.log(
                        _this.tea_info.filter(
                            (el) => i?.data[0] === el.teatitle
                        )[0]?.teacontent
                    );
                    _this.clearProvinceChart();

                    // _this.$emit('customEvent', name_click_current);
                } else {
                    name_click_current = -1;
                    d3.select('#name-pie-' + i.index).style(
                        'fill',
                        '#00000000'
                    );

                    _this.visible = false;
                    console.log(this.visible);
                    // _this.$emit('customEvent', -1);
                     // 隐藏茶叶信息部分
                        if (teaInfoContainer) {
                            teaInfoContainer.style.display = 'none';  // 隐藏 tea_info_container
                        }
                }

                // console.log('点击了茶index:'+i.index);
            }

            function mouse_over_province(d, i) {
                d.stopPropagation();
                
                mouse_over_in_action = true;
                ctx.clearRect(-width / 2, -height / 2, width, height);
                clear_outer_ring();
                var line_data = cover_data.filter(function (c) {
                    return c[0].source_a === i.index;
                });
                
                ctx.globalAlpha = 0.3;
                create_lines('character', line_data);
                for (var j = 0; j < line_data.length; j++) {
                    var index = line_data[j][0].target_a;
                    d3.select('#name-pie-' + index).style(
                        'fill',
                        line_data[0][0].strokeStyle
                    );
                    d3.select('#name-label-circle-' + index).style(
                        'fill',
                        line_data[0][0].strokeStyle
                    );
                }
            }
        },
    },
    computed: {
        
        name_data() {
            return [
                //浙江1
                ['天台山', 1],
                ['雁荡山', 1],

                //安徽
                ['白岳山', 1],
                ['黄山', 1],
                //福建1
                ['武夷山', 1],
                //江西
                ['庐山', 1],
                //浙江2
                ['江郎山', 1],
                //福建2
                ['石竹山', 1],

                //河南
                ['嵩山', 1],
                //陕西
                ['华山', 1],
                //湖北
                ['武当山', 1],
                //山西
                ['五台山', 1],
                ['恒山', 1],

                //湖南
                ['茶陵州', 1],
                
                ['石鼓书院', 1],
                ['浯溪碑林', 1],
                ['永州八景', 1],
                ['濂溪故里', 1],
                ['舜帝陵', 1],
                ['石门龙洞', 1],
                //广西
                ['炎井温泉', 1],
                ['灵渠', 1],
                ['桂林十景', 1],
                ['麒麟山', 1],
                ['邕州八景', 1],
                //贵州
                ['独山州', 1], 
                ['白云山', 1],
                //云南
                ['鸡足山', 1],
                ['黄龙潭', 1],
                ['澜沧江', 1],

                
               
            ];
        },

        province_name() {
            return [
                ['浙', 3],
                ['皖', 2],
                ['闽', 2],
                ['赣', 1],
                ['豫', 1],
                ['陕', 1],
                ['鄂', 1],
                ['晋', 2],
                ['湘', 7],
                ['桂', 5],
                ['黔', 2],
                ['滇', 3],
                              
            ];
        },
    },
    data() {
        return {
            tea_info_container_visible: false,// 控制 tea-info-container 是否可见
            /*GreenTea: true,
            WhiteTea: false,
            YellowTea: false,
            oolong: false,
            RedTea: false,
            BlackTea: false,*/
            
            
           
            windowHeight: window.innerHeight,
            windowWidth: window.innerWidth - 300,
            visible: false,
            mode: 'push',
            placement: 'left',
            current: 0,
            myChart: '',
            distributionOptions: '',
            radius: 0,
            tea_info: [
                //浙江1
                {
                    teatitle: '天台山',
                    teacontent:
                        '<p>台州市天台山景区，位于浙江省台州市天台县和合街1号，地处宁波、绍兴、金华、温州四市的交界地带，西南连仙霞岭，东北遥接舟山群岛，为曹娥江与甬江的分水岭，多悬岩、峭壁、瀑布。</p><p>天台山是浙江省重要的生态屏障，对于保持区域生态平衡有着重要的作用；是禅宗发源地之一，拥有悠久的佛教文化历史，其间涌现出一批著名的佛教文化名人和历史文化遗产；是中国著名的地质和地貌科学研究区域，对于研究区域地质演化、自然环境变化等具有重要意义。</p>',
                    teaimg: require("../assets/img/天台山.png"),
                    teaprizerange: [500, 3500],
                    teafermentdegrange: [0, 0],
                },
                {
                    teatitle: '雁荡山',
                    teacontent:
                        '<p>雁荡山，位于浙江省温州市乐清市雁荡镇，简称雁山、雁岩，素有“海上名山、寰中绝胜”“东南第一山”之誉。雁荡山分为南雁荡山和北雁荡山，瓯江断裂以北称为北雁荡，以南称为南雁荡。主峰为百岗尖，海拔1057米，总面积450平方千米。</p><p>雁荡山开山凿胜始于南北朝，兴于唐，盛于宋，文化底蕴丰厚。 雁荡山分灵峰、灵岩、大龙湫、三折瀑、雁湖、显胜门、羊角洞、仙桥八大景区，有500多处景点。雁荡山大尺度的奇特景观形象，给诗人、画家、文人学士以强烈的美感和灵感。于此赋诗作画，留下大批作品，其中诗词5000多首，以及龙鼻洞等400多处摩崖石刻，还有南阁牌坊等历史古迹，谢灵运、贯休、沈括、徐霞客、康有为、张大千、沙孟海、潘天寿、郁达夫、郭沫若、邓拓、舒婷等都留下了诗篇和墨迹。</p>',
                    teaimg: require("../assets/img/浙江雁荡山.jpg"),
                    teaprizerange: [400, 4000],
                    teafermentdegrange: [0, 0],
                },

                //安徽
                {
                    teatitle: '白岳山',
                    teacontent:
                        '<p>齐云山，位于安徽省休宁县，古称白岳，因遥观山顶与云平齐得名。矗立于横江南岸，是一座呈紫红色的砂砾的丹霞地貌山。山不高，却时常云雾缭绕，故名“白岳山”，又名齐云山。至明代，齐云山与黄山齐名，又因嘉靖皇帝笃信道教，道观兴盛，与湖北武当山、四川青城山、江西龙虎山并列为道教四大名山。</p>',
                    teaimg: require("../assets/img/安徽白岳山.jpeg"),
                    teaprizerange: [600, 1500],
                    teafermentdegrange: [5, 10],
                },
                
                {
                    teatitle: '黄山',
                    teacontent:
                        '<p>黄山，古称黟山，位于安徽省黄山市境内，地处安徽省南部、黄山市北部，以奇松、怪石、云海、温泉、冬雪“五绝”及历史遗存、书画、文学、传说、名人“五胜”著称于世，有“天下第一奇山”“天开图画”“松海云川”之称。 </p><p>黄山境内有千米以上高峰88座，其中“莲花”“光明顶”“天都”为黄山三大主峰,且有着大量的文化遗存，如古蹬道、古楹联、古桥、古亭、古寺、古塔等，另有现存摩崖石刻300余处，孕育了中国山水画“黄山画派”。</p><p>徐霞客游黄山归来，留下了“五岳归来不看山，黄山归来不看岳”的名句。</p>',
                    teaimg: require("../assets/img/安徽黄山.png"),
                    teaprizerange: [1000, 3000],
                    teafermentdegrange: [5, 10],
                },
                {
                    teatitle: '武夷山',
                    teacontent:
                        '<p>武夷山，位于福建省西北部与江西省的交界处，形成了北部长江下游支流与南部福建省闽江流域之间的分水岭；是中国东南部最高山脉，被称为“华东屋脊”；拥有110多座超过1000米的山峰（最高峰黄岗山，海拔2158米）。</p><p>武夷山属亚热带季风季候， 拥有众多的清泉、飞瀑、山涧、溪流和多样性基本完整的林带，是中国亚热带森林和中国南部雨林多样性面积最大、最具代表性的例证；并且对于生物多样性的保护具有巨大的重要性。该地区拥有不少于35处北宋至清朝时期（10—19世纪）的古代儒家书院，为朱子理学的发展和传播提供了良好环境。此外，该地区还拥有大量的坟墓遗址、摩崖石刻和存放着可追溯到商朝的木制船棺的岩穴，60多处道观和寺院遗迹，以及汉朝统治者于公元1世纪建立的行政首府，均极具考古价值。其主要景点有：有武夷宫、水帘洞、大王峰、虎啸岩、遇林亭等。</p>',
                    teaimg: require("../assets/img/福建武夷山.jpg"),
                    teaprizerange: [1000, 3000],
                    teafermentdegrange: [5, 10],
                },
                {
                    teatitle: '庐山',
                    teacontent:
                        '<p>庐山，又名匡山、匡庐，位于江西省九江市庐山市境内。北濒长江，东接鄱阳湖，南靠南昌滕王阁，西邻京九铁路，耸峙于长江中下游平原与鄱阳湖畔，主峰汉阳峰，海拔1474米。山体呈椭圆形，典型的地垒式断块山。</p><p>庐山以雄、奇、险、秀闻名于世，被誉为“人文圣山”，素有“匡庐奇秀甲天下”之誉。自古命名的山峰有171座，群峰间散布冈岭26座，壑谷20条，岩洞16个，怪石22处。水流在河谷发育裂点，形成许多急流与瀑布，瀑布22处，溪涧18条，湖潭14处。最为著名的三叠泉瀑布，落差达155米，有“不到三叠泉，不算庐山客”之美誉。庐山独特的局地气候和地形决定了庐山独特的气候。庐山地处我国亚热带东部季风区域，气候受到大范围气候状况的制约，具有鲜明的季风气候，表现出夏短冬长，春迟秋早的四季特色。故而有“清凉世界”的美誉。</p>',
                    teaimg: require("../assets/img/江西庐山.jpg"),
                    teaprizerange: [1000, 3000],
                    teafermentdegrange: [5, 10],
                },
                {
                    teatitle: '江郎山',
                    teacontent:
                        '<p>江郎山，古称金纯山、须郎山、玉郎山，位于浙江省江山市西南，仙霞山脉北麓、浙、闽、赣三省交界处。唐朝高士廉《文思博要》记载“有江氏三兄弟登巅化为石，故又名江郎山，”素有“雄奇冠天下，秀丽甲东南”之誉。</p><p>江郎山以三爿石著称于世，聚岩、洞、云、瀑于一山，集奇、险、陡、峻于三石。该山属于亚热带季风性湿润气候，温暖湿润，四季分明。江郎山景区由江郎山、峡里湖、仙霞关、廿八都和浮盖山五个部分组成，沿着中轴线，自南向北分别有一线天、登天坪、百步峡、钟鼓洞、会仙岩、伟人峰、十八曲、江郎书院、开明禅寺等主要景点。</p><p>从唐宋以来，白居易、陆游、杨万里、辛弃疾、徐渭等名人都在江郎山留下诗作，徐霞客也曾三次造访江郎山。有全国最大的毛泽东手书体“江山如此多娇”摩崖题刻，自然景观与人文古迹交相辉映。</p>',
                    teaimg: require("../assets/img/浙江江郎山.jpeg"),
                    teaprizerange: [160, 1500],
                    teafermentdegrange: [5, 10],
                },
                {
                    teatitle: '石竹山',
                    teacontent:
                        '<p>石竹山风景区位于福建省福州市福清市西宏路镇内，紧靠鲤鱼湖。总面积47.48平方千米，其中核心景区面积15.67平方千米，以其翠竹满山与奇山嶙峋而得名。石竹山是“梦文化的发祥地和传承地”，享有“中华梦乡·石竹山仙山”的美誉。石竹山是“国家首批水利风景区”、国家AAAA级旅游景区。因怪石嶙峋，修竹茂盛而得名。</p><p>徐霞客在游记中评价道：“岩石最胜，亦为九仙祈梦所”</p>',
                    teaimg: require("../assets/img/福建石竹山.jpg"),
                    teaprizerange: [1000, 10000],
                    teafermentdegrange: [5, 10],
                },
                {
                    teatitle: '嵩山',
                    teacontent:
                        '<p>嵩山，五岳之“中岳”，秦岭山系东延的余脉，位于河南省郑州市登封市，嵩山山系自西向东依次有万安山、安坡山、马鞍山、五佛山、挡阳山、玉寨山、嵩山主峰（峻极峰）、五指岭和尖山等</p><p>嵩山少林景区是国家旅游局首批公布的5A级景区。景区内人文景观丰富，自然景观奇特。人文景点集中于景区核心区2.18平方千米以内，主要有“天下第一名刹”之称的禅宗祖庭和少林武术发祥地的少林寺、有中国现存规模最大、数量最多的塔林，还有中国现存最早的石阙少室阙以及武术馆、达摩洞、初祖庵、二祖庵、十方禅院、甘露台等景观。自然景观集中于少室山三皇寨，融汇了前寒武纪时期著名的“嵩阳、中岳、少林”三大造山、造陆运动，是嵩山世界地质公园最佳观景处，代表景观有猴子观天、云峰虎啸、御寨落日、少室秋色等40余处。</p>',
                    teaimg: require("../assets/img/河南嵩山.png"),
                    teaprizerange: [300, 1000],
                    teafermentdegrange: [5, 10],
                },

                
                {
                    teatitle: '华山',
                    teacontent:
                        '<p>华山，古称“西岳”，雅称“太华山”，为中国著名五岳之一、国家AAAAA级旅游景区，位于陕西省华阴市，有着“奇险天下第一山”的美誉。</p><p>华山是中华文明的发祥地，“中华”“华夏”中的“华”字就源于华山，因此此地有着“华夏之根”之称；而在原始社会，华山还是祭祀神山，秦始皇、汉武帝、武则天、唐玄宗等帝王在此曾进行过大规模祭祀活动。另外，华山是道教全真派的发祥地之一，华山的西玄洞为“第四洞天”，全山共拥有72个半悬空洞、20余座道观，有陈抟、郝大通、贺元希等道教高人，流传着“劈山救母”“吹萧引凤”等道教神话传说，留下了诗歌、碑记等1200余篇，摩岩石刻多达千余处。1982年，华山成为“首批国家级风景名胜区”。2004年，成为“中华十大名山”。</p>',
                    teaimg: require("../assets/img/陕西华山.png"),
                    teaprizerange: [200, 500],
                    teafermentdegrange: [10, 20],
                },
                {
                    teatitle: '武当山',
                    teacontent:
                        '<p>武当山，位于湖北省西北部十堰市丹江口市，是中国道教圣地，又名太和山、谢罗山、参上山、仙室山，古有“太岳”、“玄岳”、“大岳”之称。</p><p>武当山以“四大名山皆拱揖，五方仙岳共朝宗”的“五岳之冠”地位闻名于世。武当山是道教名山和武当武术的发源地，被称为“亘古无双胜境，天下第一仙山”。武当武术，是中华武术的重要流派。元末明初，道士张三丰集其大成，开创武当派。截至2013年，武当山有古建筑53处，建筑面积2.7万平方米，建筑遗址9处，占地面积20多万平方米，全山保存各类文物5035件。</p><p>1994年12月，武当山古建筑群入选《世界遗产名录》，2006年被整体列为“全国重点文物保护单位”。2007年，武当山和长城、丽江、周庄等景区一起入选 “欧洲人最喜爱的中国十大景区”。2010至2013年，武当山分别被评为国家AAAAA级旅游风景区、国家森林公园、中国十大避暑名山、海峡两岸交流基地，入选最美 “国家地质公园”。</p>',
                    teaimg: require("../assets/img/湖北武当山.jpg"),
                    teaprizerange: [600, 1500],
                    teafermentdegrange: [10, 20],
                },
                {
                    teatitle: '五台山',
                    teacontent:
                        '<p>五台山位于山西省忻州市，是中国佛教四大名山之一。五台山属太行山系的北端，地层完整丰富，特别是前寒武系地层典型奇特，是中国地质科考的重点地区。</p><p>《名山志》载：“五台山五峰耸立，高出云表，山顶无林木，有如垒土之台，故曰五台。”五台山是中国青庙黄庙共处的佛教道场，有宗教活动场所86处，其中多敕建寺院，多朝皇帝前来参拜。著名的有：显通寺、塔院寺、菩萨顶、南山寺、黛螺顶、金阁寺、万佛阁、碧山寺等。</p>',
                    teaimg: require("../assets/img/山西五台山.png"),
                    teaprizerange: [250, 580],
                    teafermentdegrange: [10, 20],
                },

                {
                    teatitle: '恒山',
                    teacontent:
                        '<p>恒山，含狭义和广义两个概念。狭义的恒山，即所谓“北岳恒山”，亦名“太恒山”，古称玄武山、崞山，高是山，玄岳等，明末清初被确定为“五岳”之北岳恒山。位于山西省浑源县城南10公里处，主峰天峰岭海拔2017米，号称“人天北柱”、“绝塞名山”。广义的恒山，或称恒山山脉，是山西省大同市东南部、河北省张家口市南部，桑干河、滹沱河之间一系列山峰的总称，大致西南－东北走向延伸，东西长近300公里左右，南北宽80公里左右，最高峰为山阴县、应县边界处的馒头山，海拔2426米。</p><p>恒山横亘于山西北部高原与冀中平原之间，因其险峻的自然山势和地理位置的特点，成为兵家必争之地。山间河谷处的倒马关、紫荆关、平型关、雁门关、宁武关虎踞为险，是塞外高原通向太原盆地、冀中平原之咽喉要冲。许多帝王、名将都在此打过仗，古代关隘、城堡、烽火台等众多古代战场遗迹保存了下来。自明代，被列为北岳，清初，北岳国家祭祀也逐渐由河北曲阳移至浑源恒山，五岳文化融入恒山文化中。恒山道教始于西汉，北魏始兴盛，成为主流全真派圣地。</p>',
                    teaimg: require("../assets/img/山西恒山.png"),
                    teaprizerange: [150, 580],
                    teafermentdegrange: [10, 20],
                },

                {
                    teatitle: '茶陵州',
                    teacontent:
                        '<p>茶陵州，隶属湖南省株洲市，因地处“茶山之阴”谓之为“茶乡”，又因炎帝神农氏“崩葬于茶乡之尾”而得名。是中国唯一以茶命名的行政县，茶叶产业为全县主导产业之一。茶陵自古“农勤于耕、士勤于学”，历代兴办的书院数量达38所，居湖南各州县之首；科举考试中进士者127人，其中刘三吾、李东阳、张治、彭维新并称明清“四大学士”，素有“四相文章冠两朝”的美誉。李东阳开创的“茶陵诗派”在中国文学史上影响深远。炎帝神农氏在茶陵发现了茶，千年国饮，始于茶陵。</p><p>茶陵州的景色俱佳，有茶陵八景之称。徐霞客游茶陵州的麻叶洞时写下：“两壁石质石色，光莹欲滴，垂柱倒莲，纹若镂雕，形欲飞舞……”他的记载点燃了不少的旅游探险者来游麻叶洞的热情，然而奇怪的是，从那以后，再也没有人真正探访过麻叶洞。</p>',
                    teaimg: require("../assets/img/茶陵州.png"),
                    teaprizerange: [200, 500],
                    teafermentdegrange: [10, 20],
                },
                {
                    teatitle: '石鼓书院',
                    teacontent:
                        '<p>石鼓书院位于今石鼓区石鼓山，始建于唐元和五年（810年），迄今已有1000多年的历史。宋代太平兴国二年（978年），衡州名士李宽在石鼓山寻真观旁结庐读书，宋太宗赵匡义赐名“石鼓书院”。宋至道三年（公元997年），邑人李士真拓展其院，作为衡州学者讲学之所。宋景祜二年（公元1035年），朝廷赐额“石鼓书院”，遂与睢阳、白鹿洞、岳麓书院并称四大书院，素有“衡湘洙泗”“道南正脉”之誉。</p><p>徐霞客造访石鼓书院，赞誉称“石鼓书院兼具滕王阁、黄鹤楼名胜之优越”</p>',
                    teaimg: require("../assets/img/石鼓书院.jpg"),
                    teaprizerange: [60, 335],
                    teafermentdegrange: [10, 20],
                },
                {
                    teatitle: '浯溪碑林',
                    teacontent:
                        '<p>浯溪碑林，位于祁阳市西南部、湘江西岸。苍崖石壁连绵78米，最高处30余米，是湖南省风景名胜区、爱国主义教育基地、十大文化遗产，这里文奇、字奇、石奇，历代共有250多名文人学士到此游览，题诗作赋，铭刻石上，为南国摩崖第一一家， 为神州一颗璀璨的文化明珠。</p><p>浯溪碑林是珍贵的书法石刻宝库，现存摩座石刻1605方，其中唐代17、宋代116、元代5、明代84、清代2民国9、时代不明的12万。最高的刻在30米崖峭壁上，字径28米;最低的在鲜崖脚，字径1厘米。系隶指行章诸体皆全。其作者荟草了唐以来的历代名家。</p><p>徐霞客称：“浯溪之胜，不可不一登”，即便当时腹痛难忍，也执意造访了浯溪的三绝碑、石镜、元颜祠、中宫寺。',
                    teaimg: require("../assets/img/浯溪碑林.jpg"),
                    teaprizerange: [300, 800],
                    teafermentdegrange: [5, 10],
                },

                {
                    teatitle: '永州八景',
                    teacontent:
                        '<p>永州古称零陵，因舜帝南巡崩于宁远九嶷山而得名。因潇水与湘江在永州城区汇合，永州自古雅称潇湘。</p><p>永州八景为朝阳旭日、回龙夕照、萍洲春涨、香零烟雨、恩院风荷、愚溪眺雪、绿天蕉影、山寺晚钟。徐霞客在他的游记中写道：“一山怒而竖石，奔与江斗。”朝阳岩依山傍水，这潇水，青绿如绸，是一曲“山水绿”的流动的歌；这洞口“九曲回香泉”，是一幅有声的画。这石洞清爽宜人又幽奇含蓄，三洞通幽，引人入静，令人思离尘世喧嚣，大发洞天福地之感慨。</p><p>徐霞客在游记中如此记载永州八景之一的朝阳岩：“逾其上，俯而东入石关，其内飞石浮空，下瞰潇水，即朝阳岩矣。其岩后通前豁，上覆重崖，下临绝壑，中可憩可倚，云帆远近，纵送其前”，此刻此景，徐霞客不顾船家的催促，攀岩入洞，细细考察一番才离去。</p>',
                    teaimg: require("../assets/img/永州八景.png"),
                    teaprizerange: [50, 800],
                    teafermentdegrange: [30, 40],
                },
                {
                    teatitle: '濂溪故里',
                    teacontent:
                        '<p>濂溪故里古建筑群，省级风景名胜区、省廉政教育示范基地、爱国主义教育基地、湖南省历史文化名村。是宋代理学鼻祖周敦颐的故乡，位于道县清塘镇楼田村，距县城10公里，据有关资料证实，鲁迅、周恩来均为周敦颐后裔。故里景区曾在古道州八大景中占两景（即道山和濂溪风光），村庄依山而建，成带形分布。 2013年，濂溪故里古建筑群被国务院公布为第七批全国重点文物保护单位。</p><p>村中以明清建筑为主，多数为清代。村中古文物建筑尚存38栋，均为天井式院落。外观为青砖围护的清水墙，与湘南农村古民居类同。古建筑群包括上门楼、爱莲堂等18栋古建筑及圣脉泉“圣脉”“寻源”明代石刻。此外，还有多处摩崖石刻以及濂溪书院、五星墩、宋墓群、文塔、五亭、大富桥等历史文化遗迹。濂溪故里古建筑群绝大部分为清代建筑，某些建筑构件或基础为明代遗存，是一处以古代建筑群为文物主体，以道山、道岩、龙山、圣脉泉为自然景观，并蕴含着丰厚理学人文内涵的周姓血缘宗族村，具有较高的历史价值和科学价值。</p>',
                    teaimg: require("../assets/img/濂溪故里.jpg"),
                    teaprizerange: [260, 2000],
                    teafermentdegrange: [20, 70],
                },
                {
                    teatitle: '舜帝陵',
                    teacontent:
                        '<p>永州市九嶷山舜帝陵，位于湖南省永州市宁远县九嶷山瑶族乡境内。由于依山为陵，最高处达638米，气势雄浑，被尊为“华夏第一圣陵”。陵内有九龙柱、毛泽东诗词碑、祭祀墙、历代名人游九嶷浮雕、明德牌坊、金水桥、神道等人文景观。陵庙有仪门、午门、拜殿、正殿、寝殿、厢房、钟鼓楼等多个精美的单体建筑。庙内有42块古碑，是珍贵的历史文物。</p><p>舜帝陵在《尚书》《山海经》《国语》《吕氏春秋》《淮南子》《史记》《帝王世纪》等五十多部史书都记叙了这一史实。《史记》载：“舜南巡狩，崩于苍梧之野，葬于江南九疑。”长沙马王堆出土的西汉帛书地图就绘记了九嶷山舜帝陵。清《古陵墓志》一书稽考，古籍文献发现五帝陵中唯舜帝陵一处指为实陵，古谓“此陵最古”，距今四千多年。由于依山为陵，陵墓最高，高达638米；陵墓最大，全山一座，占地千亩。舜帝陵始建于夏朝，是我国有史以来帝陵中最古老的陵，号称华夏第一陵。</p>',
                    teaimg: require("../assets/img/舜帝陵.jpg"),
                    teaprizerange: [150, 10000],
                    teafermentdegrange: [60, 70],
                },
                {
                    teatitle: '石门龙洞',
                    teacontent:
                        '<p>石门龙洞，位于石门湖南省临武县花塘乡村。龙洞内，一条鳞甲分明、栩栩如生的“石龙”弯曲着深潜水底。这条“石龙”长13米多，粗0.6米至0.8米，在水中弯曲着深潜水底。据《临武县志》记载：“龙洞在县北二十里，石潭中，有石龙一丈余，鳞爪俱备，唯首潸入水中……”。地质专家称，“石龙”的形成大概有上亿年的时间，实在令人称奇。</p><p>1637年，徐霞客游完永州的溶洞后，评价说：“月岩第一，道州;紫霞洞第二……”，而当他游到临武龙洞的龙潭时，因“水底白石龙一条……，真异景也”，还称“此洞品第，固当在月岩上。”</p>',
                    teaimg: require("../assets/img/石门龙洞.jpeg"),
                    teaprizerange: [100, 1200],
                    teafermentdegrange: [20, 40],
                },
                {
                    teatitle: '炎井温泉',
                    teacontent:
                        '<p>炎井温泉位于全州县大西江镇炎井村，距离桂林市153千米，距离全州县城58千米。日流量558吨，恒温42℃，该泉属低钠低矿化度含氟偏硅酸纯天然温泉。风景区内冬暖夏凉，气候宜人，高山幽谷，环境优雅，宛若一幅清新秀丽的山水画。景区内设有温泉宾馆、温泉休闲保健、水上乐园、药浴、花浴泡澡等休闲养生项目。2016年12月被评为AAA级景区。</p><p>地理环境；以炎井温泉为中心，方圆数十里内山峰林立，原始森林苍翠葱茏，蔚为壮观。在这里，炎井温泉的浴疗功能和原始自然景观相得益彰，犹为难得。含有锂、锶、锌、锗、铜、氡等多种对人体有保健、医疗作用的元素，属低纳、低矿化度，含氟氢硅酸矿泉水。炎井河谷风光绮丽，十公里一线江天十公里景。以炎井温泉为中心，沿途还有虹饮桥、千年古樟、精忠祠、童母岩等众多景点。</p>',
                    teaimg: require("../assets/img/炎井温泉.jpg"),
                    teaprizerange: [100, 1200],
                    teafermentdegrange: [20, 70],
                },
                {
                    teatitle: '灵渠',
                    teacontent:
                        '<p>灵渠，古称秦凿渠、零渠、陡河、兴安运河、湘桂运河，是秦始皇于秦三十三年（公元前214年）为一统中国大业而建的水利工程，是古代中国劳动人民创造的一项伟大工程。位于广西壮族自治区兴安县境内，于公元前214年凿成通航。灵渠流向由东向西，将兴安县东面的海洋河（湘江源头，流向由南向北）和兴安县西面的大溶江（漓江源头，流向由北向南）相连，是世界上最古老的运河之一，有着“世界古代水利建筑明珠”的美誉。</p><p>灵渠的凿通，沟通了湘江、漓江，打通了南北水上通道，为秦王朝统一岭南提供了重要的保证，大批粮草经水路运往岭南，有了充足的物资供应。公元前214年，即灵渠凿成通航的当年，秦兵就攻克岭南，随即设立桂林、象郡、南海3郡，将岭南正式纳入秦王朝的版图。灵渠联接了长江和珠江两大水系，构成了遍布华东华南的水运网。自秦以来，对巩固国家的统一，加强南北政治、经济、文化的交流，密切各族人民的往来，都起到了积极作用。灵渠虽经历代修整，依然发挥着重要作用。</p>',
                    teaimg: require("../assets/img/灵渠.jpg"),
                 
                },

                {
                    teatitle: '桂林十景',
                    teacontent:
                        '<p>桂林市，简称“桂”，古称桂州、静江、始安，广西壮族自治区辖地级市，省域副中心城市。“桂林”之名，始于秦代，秦始皇置桂林、象、南海三郡，桂林郡因当地盛产玉桂而成名，这是“桂林”名称的最早起源。</p><p>桂林是山水甲天下的国际旅游名城、底蕴深厚的国家历史文化名城、宜居宜养的生态山水名城。“万年文化桂林三宝”是桂林市具有世界级品质的文化瑰宝。桂林十景：漓江风景区、阳朔西街、象鼻山、两江四湖、独秀峰·王城景区、东西巷、芦笛岩、印象刘三姐、银子岩、会仙湿地。</p>',
                    teaimg: require("../assets/img/桂林十景.png"),
                 
                },

                {
                    teatitle: '麒麟山',
                    teacontent:
                        '<p>麒麟山风景区，位于桂林市永福县罗锦镇东南7公里的大西村南登屯金钟山附近。该风景区最著名的就是麒麟岩，岩洞内步步皆景，景象变幻无穷，最为奇特的是：在一个洞内，可以观赏到四季不同的神奇景致，洞外山峰叠翠，非岩溶丘陵区植被覆盖良好。</p>',
                    teaimg: require("../assets/img/麒麟山.jpg"),

                },

                {
                    teatitle: '邕州八景',
                    teacontent:
                        '<p>‌邕州八景‌是指古代南宁（古称邕州）的八处著名景观，这些景观在历史上曾是文人墨客游览胜地，留下了许多诗词歌赋。邕州八景包括：罗峰晓霞、青秀松涛、马退远眺、象岭烟岚、邕江春泛、望仙怀古、花洲月夜和弘仁晚钟。随着历史的变迁、城市的建设发展，古八景中的许多景观已成记忆中的往事，演变成如今南宁十大景观（扬美古风、青山塔影、明山锦绣、望仙怀古、伊岭神宫、南湖情韵、龙虎猴趣、邕江春泛、凤江绿野、九龙戏珠）</p>',
                    teaimg: require("../assets/img/邕州八景.png"),
                   
                },
                {
                    teatitle: '独山州',
                    teacontent:
                        '<p>独山县，贵州省黔南布依族苗族自治州辖县，位于贵州省最南端，与广西壮族自治区接壤。独山县系喀斯特岩溶地区，自然风景优美，北部有深峡幽谷，飞瀑流泉、百里杜鹃花和天下奇石翻天印、风动石，南部有奇异的峰林溶洞和温泉。 </p><p>徐霞客深入甲定洞探险，详细记录了洞内的结构、钟乳石和石笋景观。他对洞内的自然景象赞叹不已，称其为“洞中有洞，洞外有洞，变化万千”</p>',
                    teaimg: require("../assets/img/独山州.png"),
                    
                },

                {
                    teatitle: '白云山',
                    teacontent:
                        '<p>白云山，距金沙县城11公里，与中国酒都仁怀市茅台镇相距50公里。白云区境内有国家4A级景区2个，拥有贵阳唯一的天然泉水公园——泉湖公园、贵州首个现代休闲农业旅游景区——蓬莱仙界、贵州唯一的中心城区国家级森林公园——长坡岭国家森林公园等旅游文化景点。</p><p>徐霞客参观了建文帝亲手种植的杉树和他创立的白云寺，见到“跪勺泉”，传说由神龙为建文帝所供，访问了传说中的流米洞，见证了与建文帝相关的遗迹和神话。</p>',
                    teaimg: require("../assets/img/白云山2.jpg"),
                    
                },

                //云南
                {
                    teatitle: '鸡足山',
                    teacontent:
                        '<p>鸡足山，又名九曲山、青巅山，位于云南省大理白族自治州宾川县城西北方，东西长7千米，南北宽6千米，总面积28.22平方千米。</p><p>鸡足山是国家级风景名胜区大理景区的重要景点、是以展示佛教文化和生态景观为主的集佛事朝拜、佛学研究、观光旅游、科普科考为一体的多功能旅游景区 。鸡足山共有“大寺八，小寺三十有四，庵院六十有五，静室一百七十余所”，其中著名的大寺有金顶寺、传灯寺、太子阁、九莲寺、迦叶殿、石钟寺、寂光寺、慧灯庵等。</p>',
                    teaimg: require("../assets/img/鸡足山.jpg"),
                  
                },

                {
                    teatitle: '黄龙潭',
                    teacontent:
                        '<p>黄龙潭位于鹤庆古城西南螺峰山脚下的黄龙潭景区（包括黄龙潭、朝霞寺），是省级黄龙潭风景名胜区的重要组成部份，是鹤庆古老的水利工程和景色宜人的游览地。</p><p>黄龙潭西枕螺峰，“螺峰野色”是“鹤阳八景”之一。螺峰俗名谷堆山，海拔2990米，由裸露的石灰岩构成。因山势高峻陡峭，形状像螺蛳又像发髻，故名螺髻峰；每当雨季，山顶经常白云缠绕，又名云峰山。山脚悬崖上有一石壁，上面有“活泼泼地”四个大字。传说在远古时代，鹤庆原是一片汪洋。有一年两颗珠宝从崖下的水洞里蹦出来撞在崖上，突然炸开，里面飞出一双男女。后来他俩繁衍了鹤庆坝各姓的始祖，人们称这里为“活石珠光”，称“活石”下的水洞为龙洞。每当晴天中午，日光直射龙洞，就有像金丝束的反光射出，传说是被黄龙吸走的那口钟的反光，人们称这种现象为“金钟映日”或“金线吊葫芦”。</p>',
                    teaimg: require("../assets/img/黄龙潭2.jpg"),
                   
                },
                {
                    teatitle: '澜沧江',
                    teacontent:
                        '<p>澜沧江，发源于中国青海省唐古拉山东北部，流经西藏、云南两省区，出中国国境后被称为湄公河，经缅甸、老挝、泰国、柬埔寨，于越南胡志明市注入南海，是东南亚最大的国际河流。流域地貌类型复杂多样，不同的地貌类型以及地势高低、坡度大小、山川走向，直接影响着水热条件的再分配。河流上、中、下游自然环境差异显著，从地势上看，流域由北向南呈阶梯状下降趋势，其主体地貌特征表现为高山峡谷相间。随山脉南延，山川间距由上游向下逐渐展宽，上紧下疏如帚状。</p><p>徐霞客对澜沧江下游的流向非常感兴趣，他54岁时候决定专程赶赴云州（今云县）追问澜沧江下游的流向。《徐霞客游记》中，“澜沧江”“澜沧”“沧江”之名前后70多次交替出现，其着笔频率远远高于其他江河，澜沧江也是徐霞客一生中最后考察的一条大江。与澜沧江的江河之约，成为徐霞客人生最后的途旅。</p>',
                    teaimg: require("../assets/img/澜沧江.jpg"),
               
                },
            ],
        };
    },
});
</script>
  
  <style scoped>
.page-content {
    position: relative;
    width: 100%;
    height: 100%;
    min-height: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    box-sizing: border-box;
}
.page-four-side-image {
    position: absolute;
    left: 50px;
    top: 50%;
    transform: translateY(-50%);
    z-index: 10;
}
.page-four-main {
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    box-sizing: border-box;
}
.tea-info-container {
    position: absolute;  /* 绝对定位，使其相对于页面定位 */
    right: 5%;           /* 设置右侧距离，可以调节这个值来控制距离右侧的间距 */
    top: 50%;            /* 设置垂直居中，top: 50% 将元素从页面顶部位置开始 */
    transform: translateY(-50%);  /* 使用 translateY(-50%) 来精确垂直居中对齐 */
    width: 260px;        /* 控制容器宽度，您可以根据需要调整 */
    /* 如果您希望容器的高度自动根据内容来调整，则无需设置固定的 height */
    z-index: 10;         /* 确保该元素不被其他内容覆盖 */
}
.tea-info-img {
    width: 260px;
    height: 180px;
    border-radius: 5px;
    object-fit: cover;
    margin-top: 10px;  /* 调整图片与文本的间距 */
}
.tea-info-title {
    font-size: 18px;
    font-weight: 400;
    letter-spacing: 0px;
    line-height: 26.06px;
    color: rgba(0, 0, 0, 1);
    margin-bottom: 15px;
    text-align: left;
}
.tea-info-content {
    font-size: 14px;
    font-weight: 400;
    letter-spacing: 1px;
    line-height: 20px;
    color: rgba(0, 0, 0, 0.85);
    text-align: left;
    text-indent: 2em;
    margin-bottom: 10px;  /* 给文本和图片之间添加一点间距 */
    width: 100%;  /* 文本宽度占满父容器 */
}
.tea-info-card {
    width: 48%;
    border-radius: 5px;
    border: 1px solid rgba(0, 0, 0, 0.5);
    padding: 6px 0;
}
.flex3 {
    display: flex;
    justify-content: center;
    align-items: center;
}
.range-text {
    font-size: 18px;
}
.range-note {
    font-size: 8px;
    display: flex;
    justify-content: center;
    align-items: flex-end;
}
.text {
    font-size: 25px;
    font-weight: 1000;
    color: rgba(50, 132, 40, 1);
    display: flex;
    margin-left: 10%;
}
.textsub {
    margin-top: 20px;
    text-indent: 30px;
    text-align: left;
    margin-left: 15%;
    width: 50vw;
    display: flex;
    flex-direction: column;
    font-size: 13px;
    font-weight: 500;
    color: rgba(50, 132, 40, 1);
}
</style>
