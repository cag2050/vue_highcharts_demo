<template>
    <div class="box">
        <div id="highmaps" class="container"></div>
    </div>
</template>

<script>
import Highcharts from 'highcharts/highstock'
import HighchartsMore from 'highcharts/highcharts-more'
import HighchartsDrilldown from 'highcharts/modules/drilldown'
import Highcharts3D from 'highcharts/highcharts-3d'
import Highmaps from 'highcharts/modules/map'
import $ from 'jquery'
// 除了 import 导入的 Highcharts，其他都要执行下类似下面的语句
HighchartsMore(Highcharts)
HighchartsDrilldown(Highcharts)
Highcharts3D(Highcharts)
Highmaps(Highcharts)

export default {
    name: '',
    data () {
        return {
        }
    },
    mounted () {
        this.highmaps()
    },
    methods: {
        highmaps () {
            Highcharts.setOptions({
                lang: {
                    drillUpText: '< 返回 “{series.name}”'
                }
            })
            let map = null
            let geochina = 'https://data.jianshukeji.com/jsonp?filename=geochina/'
            $.getJSON(geochina + 'china.json&callback=?', function (mapdata) {
                var data = []
                // 随机数据
                Highcharts.each(mapdata.features, function (md, index) {
                    var tmp = {
                        name: md.properties.name,
                        value: Math.floor((Math.random() * 100) + 1) // 生成 1 ~ 100 随机值
                    }
                    if (md.properties.drilldown) {
                        tmp.drilldown = md.properties.drilldown
                    }
                    data.push(tmp)
                })
                console.log(data)
                map = new Highcharts.Map('highmaps', {
                    chart: {
                        events: {
                            drilldown: function (e) {
                                console.log(e)
                                // 异步下钻
                                if (e.point.drilldown) {
                                    var pointName = e.point.properties.fullname
                                    // 不请求县级数据
                                    // 如果 e.point.drilldown 中含义字符'/'，是请求县级数据，进行 return 操作
                                    if (/\//.test(e.point.drilldown)) {
                                        return
                                    }
                                    map.showLoading('下钻中，请稍后...')
                                    // 获取二级行政地区数据并更新图表
                                    $.getJSON(geochina + e.point.drilldown + '.json&callback=?', function (data) {
                                        data = Highcharts.geojson(data)
                                        Highcharts.each(data, function (d) {
                                            if (d.properties.drilldown) {
                                                d.drilldown = d.properties.drilldown
                                            }
                                            d.value = Math.floor((Math.random() * 100) + 1) // 生成 1 ~ 100 随机值
                                        })
                                        map.hideLoading()
                                        map.addSeriesAsDrilldown(e.point, {
                                            name: e.point.name,
                                            data: data,
                                            dataLabels: {
                                                enabled: true,
                                                format: '{point.name}'
                                            }
                                        })
                                        map.setTitle({
                                            text: pointName
                                        })
                                    })
                                }
                            },
                            drillup: function () {
                                map.setTitle({
                                    text: '中国'
                                })
                            }
                        }
                    },
                    title: {
                        text: '中国地图'
                    },
                    subtitle: {
                        text: '<a href="https://www.hcharts.cn/mapdata">点击查看地图数据及详情，注意县级数据为收费数据，如果您有需要，请联系我们购买</a>'
                    },
                    mapNavigation: {
                        enabled: true,
                        buttonOptions: {
                            verticalAlign: 'bottom'
                        }
                    },
                    tooltip: {
                        useHTML: true,
                        headerFormat: '<table><tr><td>{point.name}</td></tr>',
                        pointFormat: '<tr><td>全称</td><td>{point.properties.fullname}</td></tr>' +
                        '<tr><td>行政编号</td><td>{point.properties.areacode}</td></tr>' +
                        '<tr><td>父级</td><td>{point.properties.parent}</td></tr>' +
                        '<tr><td>经纬度</td><td>{point.properties.longitude},{point.properties.latitude}</td></tr>',
                        footerFormat: '</table>'
                    },
                    colorAxis: {
                        min: 0,
                        minColor: '#fff',
                        maxColor: '#006cee',
                        labels: {
                            style: {
                                'color': 'red', 'fontWeight': 'bold'
                            }
                        }
                    },
                    series: [{
                        data: data,
                        mapData: mapdata,
                        joinBy: 'name',
                        name: '中国地图',
                        states: {
                            hover: {
                                color: '#a4edba'
                            }
                        }
                    }],
                    credits: {
                        // 去掉版权信息
                        enabled: false
                    }
                })
            })
        }
    }
}
</script>

<style>
.box {
    width: 60%;
}
</style>
