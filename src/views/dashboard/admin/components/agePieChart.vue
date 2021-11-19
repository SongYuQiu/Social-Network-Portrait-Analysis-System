<template>
  <div id="ageChart" :class="className" :style="{height:height,width:width}" />
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'
import { fetchUserProbability, getUserPortrait } from '@/api/weibo-user'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '50%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  data() {
    return {
      chart: null,
      s_p:0,
      e_p:0,
      n_p:0
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      const that = this
      that.weibo_user_id = localStorage.getItem('weibo_user_id')
      fetchUserProbability(that.weibo_user_id).then(response => {

        that.s_p = response.data.seven_probability
        that.e_p = response.data.eight_probability
        that.n_p = response.data.nine_probability

        that.s_p = Math.round(that.s_p*1000)
        that.e_p = Math.round(that.e_p*1000)
        that.n_p = Math.round(that.n_p*1000)

        that.chart  = echarts.init(document.getElementById('ageChart'),'macarons');

        that.chart.setOption({
          tooltip: {
            trigger: 'item',
            formatter: '{b}:{d}%',
            backgroundColor: 'rgba(255,255,255,0.7)'
          },
          legend: {
            left: 'center',
            bottom: '10',
            data: ['70后', '80后', '90后']
          },
          series: [
            {
              name: '年龄预测',
              type: 'pie',
              roseType: 'radius',
              radius: [15, 95],
              center: ['50%', '50%'],
              itemStyle: {
                borderRadius: 8
              },
              data: [
                { value: that.s_p, name: '70后' },
                { value: that.e_p, name: '80后' },
                { value: that.n_p, name: '90后' },
                // { value: 149, name: 'Forex' },
                // { value: 100, name: 'Gold' },
                // { value: 59, name: 'Forecasts' }
              ],
              animationEasing: 'cubicInOut',
              animationDuration: 2000
            }
          ]
        })
      })
    }
  }
}
</script>
