<template>
	<div :id="id" style="height:580px;padding:12px;" />
</template>
<script>
    import echarts from 'echarts'
    import { getRealTimeDataToGis } from '@/api/pannel-main/pannel.js'
    import $ from 'jquery'

    export default {
        name: 'GenMapTooltip',
        props: ['id'],
        data() {
            return {
                receivedData: null,
                receiveDetail: null,
                EC: null,
                chart: null,
                warnData: null,
                config: {
                    priority: 'top', // 默认在点上方OR下方（top/bottom）
                    partition: 1.4, // 左右分割比例
                    lineColor: '#fff', // 引导线颜色
                    offset: [5, 5],
                    L1: {
                        time: 0.3, // L1动画时长(单位s)
                        long: 40 // L1长度
                    },
                    L2: {
                        time: 0.3,
                        long: 40
                    },
                    text: {
                        time: 0.5,
                        font: '14px Arial',
                        color: '#fff',
                        padding: [10, 10],
                        width: 120,
                        height: 60,
                        lineHeight: 24,
                        backgroundColor: 'rgba(50, 50, 50, 0.8)'
                    }
                },
                level: {
                    level1: [],
                    level2: [],
                    level3: []
                },
                geoCoordMap: {
                },
                levelColorMap: [
                    'rgba(246,73,84, 1)',
                    'rgba(240,219,0, .8)',
                    'rgba(56,240,56, .7)',
                    'rgba(192,192,192,1)'
                ],
                data: [
                ],
                left: false,
                top: false,
                priority: 'top',
                faultByHourIndex: 0,

                option: {
                    legend: {
                        show: true,
                        orient: 'vertical',
                        left: 0,
                        textStyle: { color: '#fff' },
                        backgroundColor: 'rgba(255, 255, 255, 0.1)',
                        borderRadius: 4,
                        padding: [20, 30],
                        fontSize: 14
                    },
                    geo: {
                        map: 'shanxi',
                        label: { show: true, color: '#fff' },
                        itemStyle: {
                            normal: {
                                label: {
                                    show: true,
                                    color: '#fff'
                                },
                                areaColor: {
                                    type: 'radial',
                                    x: 0.5,
                                    y: 0.5,
                                    r: 0.8,
                                    colorStops: [
                                        {
                                            offset: 0,
                                            color: 'rgba(147, 235, 248, 0)' // 0% 处的颜色
                                        },
                                        {
                                            offset: 1,
                                            color: 'rgba(147, 235, 248, .2)' // 100% 处的颜色
                                        }
                                    ],
                                    globalCoord: false // 缺省为 false
                                },
                                borderColor: 'rgba(147, 235, 248, 1)',
                                borderWidth: 1,
                                shadowColor: 'rgba(128, 217, 248, 1)',
                                shadowOffsetX: -2,
                                shadowOffsetY: 5,
                                shadowBlur: 10
                            }
                        },
                        silent: true
                    },
                    tooltip: {
                        trigger: 'item',
                        triggerOn: 'click',
                        backgroundColor: 'transparent',
                        alwaysShowContent: true
                    },
                    series: [
                        {
                            name: '预警',
                            type: 'effectScatter',
                            coordinateSystem: 'geo',
                            data: [],
                            symbolSize: 15,
                            showEffectOn: 'render',
                            rippleEffect: {
                                brushType: 'stroke'
                            },
                            hoverAnimation: true,
                            label: {
                                normal: {
                                    formatter: '{b}',
                                    position: 'right',
                                    show: false
                                }
                            },
                            itemStyle: {
                                normal: {
                                    color: '#f00',
                                    shadowBlur: 10,
                                    shadowColor: '#333'
                                }
                            },
                            zlevel: 1
                        },
                        {
                            name: '正常',
                            type: 'effectScatter',
                            coordinateSystem: 'geo',
                            data: [],
                            symbolSize: 5,
                            hoverAnimation: true,
                            label: {
                                normal: {
                                    show: false
                                }
                            },
                            itemStyle: {
                                normal: {
                                    color: '#38f038'
                                }
                            },
                            zlevel: 0
                        }
                    ]
                }
            }
        },
        watch: {
            receivedData(nVal, oVal) {
                if (this.EC !== null) {
                    // this.warnData  gis接口拿到的所有信息
                    // receiveDetail  表格传入的数据
                    for (var i = 0; i < this.warnData.length; i++) {
                        if (this.warnData[i].id !== null) {
                            this.receiveDetail = nVal
                            // 传入的表格条数据公司id与gis地图里的集团id对应，并用条数据里的矿编号在集团所属矿里，刚更改点颜色
                            if (this.receivedData.row.companyId === this.warnData[i].corporationId && this.warnData[i].mineCode.includes(this.receivedData.row.mineCode)) {
                                const dataSingles = {
                                    name: this.receiveDetail.row.companyName,
                                    value: 10,
                                    detail: this.receiveDetail.row
                                }

                                this.drawLine()
                                this.level['level1'].push(dataSingles)
                                var hash = {}
                                this.level['level1'] = this.level['level1'].reduce(function(item, next) {
                                    hash[next.name] ? '' : hash[next.name] = true && item.push(next)
                                    return item
                                }, [])

                                var newData = this.convertData(this.level['level1'])
                                this.option.series[0].data = newData
                                this.EC.dispatchAction({
                                    type: 'showTip',
                                    seriesIndex: 0,
                                    name: this.receiveDetail.row.companyName
                                })
                            }
                        }
                    }
                }
            }
        },
        mounted() {
            this.getData()
        },
        methods: {

            // 获取后台数据/
            getData() {
                getRealTimeDataToGis({}).then(res => {
                    // 异步调用
                    const msg = res.result.content
                    for (let i = 0, len = msg.length; i < len; i++) {
                        const dataSingle = {
                            name: msg[i].companyName,
                            value: 10
                        }
                        this.data.push(dataSingle)
                        this.geoCoordMap[msg[i].companyName] = [msg[i].longitude, msg[i].latitude]
                    }
                    this.drawLine()
                    this.dataSplit(this.data, 2)
                    for (var i = 0; i < this.option.series.length; i++) {
                        var levelName = 'level' + (i + 1)
                        var newData = this.convertData(this.level[levelName])
                        this.option.series[i].data = newData
                    }
                    this.warnData = msg
                }).catch(error => {
                    this.$message({ showClose: true, message: error, type: 'error' })
                })
            },
            dataSplit(data, num) {
                // var data = this.data
                const levelS = 'level' + num
                for (var i = 0; i < data.length; i++) {
                    this.level[levelS].push(data[i])
                }
            },
            sort(data) {
                data.sort((a, b) => {
                    return b.value - a.value
                })
            },
            // 数据处理
            convertData(sortData) {
                var res = []
                for (var i = 0; i < sortData.length; i++) {
                    var geoCoord = this.geoCoordMap[sortData[i].name]
                    if (geoCoord) {
                        res.push({
                            name: sortData[i].name,
                            value: geoCoord.concat(sortData[i].value),
                            detail: sortData[i].detail
                        })
                    }
                }
                return res
            },
            // 绘制地图
            drawLine() {
                const _this = this
                $.get('static/shanxi.json', function(shanxi) {
                    echarts.registerMap('shanxi', shanxi)
                    _this.chart = echarts.init(document.getElementById(_this.id))
                    var myTooltipC = new myTooltip(_this.id)
                    // 提示框样式
                    _this.option.tooltip = {
                        trigger: 'item',
                        triggerOn: 'click',
                        renderMode: 'richText',
                        backgroundColor: 'transparent',
                        position(pos) {
                            const position = myTooltipC.getPosOrSize('pos', pos)
                            return position
                        },
                        formatter(params, p, a) {
                            const detail = _this.receiveDetail.row
                            // const detail = params.data.detail
                            // var text =
                            //     "<div style='text-align:left;'><div style='border-bottom:1px solid #ddd;height:20px;font-size:16px;margin-bottom:5px;padding-bottom:5px;'>" +
                            //     detail.mineralName +
                            //     '<div style="width:10px;height:10px;display:inline-block;border-radius:5px;background-color:#f00;text-align:right;margin-left:30px;"></div></div>' +
                            //     '预警类型:' +
                            //     detail.info.type +
                            //     '<br>' +
                            //     '监测值:' +
                            //     detail.info.value +
                            //     '<br>' +
                            //     '未处理时间:<span style="color:#f00;">' +
                            //     detail.info.time +
                            //     '</span><br>' +
                            //     '</div>'
                            var text =
                                // `<div style='background-color:#256dae'>` + detail.companyName + `</div>` +
                                detail.mineName +
                                '\n' +
                                '预警类型：' +
                                detail.warnType +
                                '\n' +
                                '监测值：' +
                                detail.warnRealData + ' ' + detail.warnRealDataUnit +
                                '\n' +
                                '发生时间：' +
                                detail.warnStartDate.slice(0, 10) +
                                '\n' + '               ' + detail.warnStartDate.slice(10, 19)
                            var tooltipDom = myTooltipC.getTooltipDom(text)
                            return tooltipDom
                        }
                    }
                    _this.EC = echarts.getInstanceByDom(document.getElementById(_this.id))

                    _this.chart.setOption(_this.option)
                    window.addEventListener('resize', function() {
                        _this.chart.resize()
                    })
                })
            }
        }
    }
</script>
