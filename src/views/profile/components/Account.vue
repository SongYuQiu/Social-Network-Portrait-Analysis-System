<template>
  <div id="chart2" :style="{height:height,width:width}" ref="chart2"></div>
</template>

<script>
import * as echarts from 'echarts'
import 'echarts-wordcloud/dist/echarts-wordcloud.min'
import 'echarts-wordcloud/dist/echarts-wordcloud'
import { fetchUserInterest, changeInterest } from '@/api/weibo-user'
require('echarts/theme/macarons') // echarts theme

export default {
  data() {
    return {
      interest: '',
      weibo_user_id: ''
    }
  },
  mounted() {
    this.initCharts()
  },
  props: {
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  methods: {
    initCharts() {
      const that = this
      that.weibo_user_id = localStorage.getItem('weibo_user_id')
      fetchUserInterest(that.weibo_user_id).then(response => {
        that.interest = response.data.interest
        let myChart2 = echarts.init(document.getElementById('chart2'), 'macarons')
        let word_list = that.interest.toString().split(' ')
        let word_data_list = []
        for (var i = 0; i < word_list.length; i++) {
          var temp = {}
          temp['name'] = word_list[i]
          temp['value'] = parseInt(Math.random()*5)
          temp['textStyle'] = this.createTextColor()
          word_data_list[i] = temp
        }
        //console.log(word_data_list)
        myChart2.setOption({
          tooltip: {
            trigger: 'item',
            formatter: '点击删除该词',
            backgroundColor: 'rgba(255,255,255,0.7)'
          },
          backgroundColor: '#fff',
          series: [{
            type: 'wordCloud',
            // shape:'circle',
            center: ['50%', '50%'],
            //用来调整词之间的距离
            gridSize: 15,
            autoSize: {
              enable: true,
              minSize: 4
            },
            //用来调整字的大小范围
            drawOutOfBound:false,
            sizeRange: [14, 60],
            //用来调整词的旋转方向，，[0,0]--代表着没有角度，也就是词为水平方向，需要设置角度参考注释内容
            rotationRange: [0, 0],
            // maskImage: maskImage,
            // rotationRange: [0,-30,30,45,-45,20,-20,90],
            data: word_data_list
          }]
        })
        myChart2.on('click',function(params){
          word_data_list.splice(params.dataIndex,1)
          let changedInterest = ''
          for(let i =0 ;i<word_data_list.length;i++){
            changedInterest = changedInterest+word_data_list[i].name+' '
          }
          that.$confirm('是否删除此关键词？', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            changeInterest(that.weibo_user_id,changedInterest).then(response =>{
              if(response.status === 200){
                that.$message({
                  message:'删除成功！',
                  type:'success',
                  duration:1000
                })
                that.initCharts()
              }
            })
          }).catch(() => {
            that.$message({
              type: 'info',
              message: '已取消删除'
            });
          });
        })
      })
    },
    createTextColor(){
      const macron = ['#2ec7c9', '#b6a2de', '#5ab1ef', '#ffb980', '#d87a80', '#8d98b3', '#e5cf0d', '#97b552',
                      '#95706d', '#dc69aa', '#07a2a4', '#9a7fd1', '#588dd5', '#f5994e', '#c05050', '#59678c',
                      '#c9ab00', '#7eb00a', '#6f5553', '#c14089' ]
      var itemColor = macron[Math.floor((Math.random()*macron.length))]
      return {
        color: itemColor
      }
    }
  }
}
</script>
