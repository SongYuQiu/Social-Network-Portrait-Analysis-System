<template>
  <div id="myChart" :class="className" :style="{height:height,width:width}" />
</template>

<script>
import * as echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'
import { fetchUserProbability } from '@/api/weibo-user'

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
      male_p:0,
      female_p:0
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
      // console.log(that.weibo_user_id)
      fetchUserProbability(that.weibo_user_id).then(response => {
        that.male_p = response.data.male_probability
        that.female_p = response.data.female_probability
        // console.log(that.male_p,that.female_p)
        that.male_p = Math.round(that.male_p*1000)
        that.female_p = Math.round(that.female_p*1000)
        that.chart  = echarts.init(document.getElementById('myChart'),'macarons');

        that.chart.setOption({
          tooltip: {
            trigger: 'item',
            formatter: '{b}:{d}%',
            backgroundColor: 'rgba(255,255,255,0.7)'
          },
          legend: {
            left: 'center',
            bottom: '10',
            data: ['男', '女']
          },
          series: [
            {
              name: '性别预测',
              type: 'pie',
              roseType: 'radius',
              radius: [15, 95],
              center: ['50%', '50%'],
              itemStyle: {
                borderRadius: 8
              },
              data: [
                { value: that.male_p, name: '男' },
                { value: that.female_p, name: '女' },
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
