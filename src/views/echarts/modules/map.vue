<template >
  <div>
    <div ref="modelMap" style="width:100%;height:800px;"></div>
  </div>
</template>
<script>
import shanxi from "@/assets/shanxi.json";
export default {
  name: "ModelMap",
  components: {},
  props: [],
  data() {
    return {
      //图表对象
      myCharts: null,
      anormalData: [
        {
          name: "李雅庄",
          value: [111.751046, 36.636808]
        },
        {
          name: "吉宁",
          value: [110.59965909697166, 35.81665825616643]
        }
      ],
      normalData:[]
    };
  },
  watch: {},
  computed: {},
  mounted() {
    this.initCharts();
  },
  methods: {
    initCharts() {
      this.myCharts = this.$echarts.init(this.$refs.modelMap);
      this.$echarts.registerMap("shanxi", shanxi);
      this.dataRender();
    },
    dataRender() {
      let option = null;
      option = {
        backgroundColor: "#0f375f",
        tooltip: {
          trigger: "item",
          formatter: function(params) {
            return params.name;
          }
        },
        legend: {
          orient: "vertical",
          id: 1,
          y: "bottom",
          x: "right",
          itemWidth: 15,
          data: ["正常", "异常"],
          textStyle: {
            color: "#fff"
          }
        },
        geo: {
          show: true,
          map: "shanxi",
          label: {
            show: true,
            color: "#e0e0e0",
          },
          zoom: 1.2,
          roam: true,
          itemStyle: {
            normal: {
              //只有线样式
              // areaColor: "transparent",
              // borderColor: "#3fdaff",
              //  color: "#fff",
              // borderWidth: 1,
              // shadowColor: "rgba(63, 218, 255, 0.5)",
              // shadowBlur: 30
              //立体样式
              //  areaColor: '#2274e2',
              //       borderColor: '#0a53e9',//线
              //       shadowColor: '#092f8f',//外发光
              //       shadowBlur: 20
              //立体边缘发光
               "borderColor": "#87ebff",
                    "areaColor": "#2274e2",
                    "borderWidth": 1,
                    "shadowBlur": 30,
                    "shadowColor": "rgba(63, 218, 255, 0.5)"
                    // 最简单，区域白色线断分隔
              //  "areaColor": "#2274e2",
              //       "borderColor": "#87ebff"
            },
            emphasis: {
              areaColor: "#2B91B7"
            }
          },
          silent: true
        },
        series: [
          {
            name: "异常",
            type: "effectScatter",
            coordinateSystem: "geo",
            data: this.anormalData,
            symbolSize: 15,
            showEffectOn: "render",
            rippleEffect: {
              brushType: "stroke"
            },
            hoverAnimation: true,
            label: {
              normal: {
                formatter: "{b}",
                position: "inside",
                show: true
              }
            },
            itemStyle: {
              normal: {
                color: "#f00",
                shadowBlur: 10,
                shadowColor: "#333"
              }
            },
            zlevel: 1
          },
          {
            name: "正常",
            type: "scatter",
            coordinateSystem: "geo",
            data: this.normalData,
            symbolSize: 5,
            hoverAnimation: true,
            label: {
              normal: {
                formatter: "{b}",
                position: "inside",
                show: true
              }
            },
            itemStyle: {
              normal: {
                color: "#38f038"
              }
            },
            zlevel: 0
          }
        ]
      };

      this.myCharts.setOption(option);
    }
  }
};
</script>
<style lang="sass" scoped>

</style>