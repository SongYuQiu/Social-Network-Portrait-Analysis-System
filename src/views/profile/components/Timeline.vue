<template>
  <div class="block">
    <el-timeline>
      <el-timeline-item v-for="(item,index) of timeline" :key="index" :timestamp="item.created_at" placement="top">
        <el-card>
          <el-tag type="success" effect="plain" v-if="item.tool">{{ item.tool }}</el-tag>
          <h3>{{ item.text }}</h3>
          <span>
            <svg-icon icon-class="like" /> {{item.like_count}}
            <svg-icon icon-class="comment" style="margin-left: 8px" /> {{item.comment_count}}
            <svg-icon icon-class="repost" style="margin-left: 8px" /> {{item.repost_count}}
            <svg-icon icon-class="location" style="margin-left: 8px" v-if="item.location" /> {{item.location}}
          </span>
        </el-card>
      </el-timeline-item>
    </el-timeline>
  </div>
</template>

<script>
import { fetchWeiboTimeLine } from '@/api/weibo-user'

export default {
  data() {
    return {
      timeline: []
    }
  },
  created() {
    this.getTimeLine()
  },
  methods:{
    getTimeLine(){
      this.weibo_user_id = localStorage.getItem('weibo_user_id')
      fetchWeiboTimeLine(this.weibo_user_id).then(response => {
        this.timeline = response.data
      })
    }
  }
}
</script>
