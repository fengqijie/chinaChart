<template>
    <div id="main" style="width: 100%; height:500px;"></div>
</template>

<script>
import $echarts from 'echarts';
import cityMap from '@/js/citymap.js';
import $axios from 'axios';

const provinces = {
    //23个省
    "台湾": "taiwan",
    "河北": "hebei",
    "山西": "shanxi",
    "辽宁": "liaoning",
    "吉林": "jilin",
    "黑龙江": "heilongjiang",
    "江苏": "jiangsu",
    "浙江": "zhejiang",
    "安徽": "anhui",
    "福建": "fujian",
    "江西": "jiangxi",
    "山东": "shandong",
    "河南": "henan",
    "湖北": "hubei",
    "湖南": "hunan",
    "广东": "guangdong",
    "海南": "hainan",
    "四川": "sichuan",
    "贵州": "guizhou",
    "云南": "yunnan",
    "陕西": "shanxi1",
    "甘肃": "gansu",
    "青海": "qinghai",
    //5个自治区
    "新疆": "xinjiang",
    "广西": "guangxi",
    "内蒙古": "neimenggu",
    "宁夏": "ningxia",
    "西藏": "xizang",
    //4个直辖市
    "北京": "beijing",
    "天津": "tianjin",
    "上海": "shanghai",
    "重庆": "chongqing",
    //2个特别行政区
    "香港": "xianggang",
    "澳门": "aomen"
};
export default {
    data() {
        return {
            special: ["北京","天津","上海","重庆","香港","澳门"],
            mapdata: [],
            option: {
                title: {
                    text: '全国终端分布',
                    subtext: '下钻',
                    link: 'https://blog.csdn.net/example440982',
                    left: 'center',
                    textStyle: {
                        color: '#fff',
                        fontSize: 16,
                        fontWeight: 'normal',
                        fontFamily: "Microsoft YaHei"
                    },
                    subtextStyle: {
                        color: '#ccc',
                        fontSize: 13,
                        fontWeight: 'normal',
                        fontFamily: "Microsoft YaHei"
                    }
                },
                tooltip: {
                    trigger: 'item',
                    formatter: '{b}'
                },
                toolbox: {
                    show: true,
                    orient: 'vertical',
                    left: 'right',
                    top: 'center',
                    feature: {
                        dataView: {readOnly: false},
                        restore: {},
                        saveAsImage: {}
                    },
                    iconStyle: {
                        normal: {
                            color: '#fff'
                        }
                    }
                },
                animationDuration: 1000,
                animationEasing: 'cubicOut',
                animationDurationUpdate: 1000,
                visualMap: {
                  type: 'continuous',
                  min: 0,
                  max: 300,
                  text:['High','Low'],
                  realtime: false,
                  calculable : true,
                  color: ['orangered','yellow','lightskyblue']
              },
            },
        }
    },
    methods: {
        initChart() {
            let chart = $echarts.init(document.getElementById('main'));
            let _self = this;
            $axios.get('/map/china.json')
                .then(function(res){
                    let d = [];
                    for( var i=0; i<res.data.features.length; i++ ){
                        d.push({
                            name: res.data.features[i].properties.name,
                            value: Math.random()*100
                        })
                    }
                    _self.mapdata = d;
                    //注册地图
                    $echarts.registerMap('china', res.data);
                    //绘制地图
                    _self.renderMap('china',d);
                });
            //地图点击事件
            chart.on('click', function (params) {
                // console.log( params );
                if( params.name in provinces ){
                    //如果点击的是34个省、市、自治区，绘制选中地区的二级地图
                    $axios.get('/map/province/'+ provinces[params.name] +'.json')
                        .then(function(res){
                            $echarts.registerMap( params.name, res.data );
                            var d = [];
                            for( var i=0; i<res.data.features.length; i++ ){
                                d.push({
                                    name: res.data.features[i].properties.name
                                })
                            }
                            _self.renderMap(params.name, d);
                        });
                }else if( params.seriesName in provinces ){
                    //如果是【直辖市/特别行政区】只有二级下钻
                    if( _self.special.indexOf( params.seriesName ) >= 0 ){
                        _self.renderMap('china', _self.mapdata);
                    }else{
                        //显示县级地图
                        $axios.get('/map/city/'+ cityMap[params.name] +'.json')
                            .then(function(res){
                                $echarts.registerMap( params.name, res.data );
                                var d = [];
                                for( var i=0; i<res.data.features.length; i++ ){
                                    d.push({
                                        name: res.data.features[i].properties.name
                                    })
                                }
                                _self.renderMap(params.name,d);
                            });	
                    }	
                }else{
                    _self.renderMap('china',this.mapdata);
                }
            });
        },
        renderMap(map, data) {
            let chart = $echarts.init(document.getElementById('main'));
            this.option.title.subtext = map;
            this.option.series = [ 
                {
                    name: map,
                    type: 'map',
                    mapType: map,
                    roam: false,
                    nameMap:{
                        'china':'中国'
                    },
                    label: {
                        normal:{
                            show:true,
                            textStyle:{
                                color:'#999',
                                fontSize:13
                            }  
                        },
                        emphasis: {
                            show: true,
                            textStyle:{
                                color:'#fff',
                                fontSize:13
                            }
                        }
                    },
                    itemStyle: {
                        normal: {
                            areaColor: '#323c48',
                            borderColor: 'dodgerblue'
                        },
                        emphasis: {
                            areaColor: 'darkorange'
                        }
                    },
                    data:data
                }	
            ];
            //渲染地图
            chart.setOption(this.option);
        }
    },
    mounted() {
        this.initChart();
    }
}
</script>

