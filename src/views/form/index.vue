<template>
  <div class="app-container">
    <div style="height: 100px"></div>
    <el-form ref="form" :model="form" label-width="200px" style="margin-left:20%;width:45%">
      <el-form-item label="微博ID号">
        <el-input v-model="form.weibo_user_id" />
      </el-form-item>
      <el-form-item label="微博账号名">
        <el-input v-model="no_use_name" />
      </el-form-item>
      <el-form-item label="爬取起始日期">
        <el-col :span="11">
          <el-date-picker v-model="form.since_date" type="date" placeholder="选择一个日期" value-format="yyyy-MM-dd" style="width: 100%;" />
        </el-col>
      </el-form-item>
      <el-form-item label="爬取转发微博">
        <el-switch v-model="no_use_select" />
      </el-form-item>
      <el-form-item label="勾选下载其他">
        <el-checkbox-group v-model="no_use_type">
          <el-checkbox label="原创微博图片" name="type" />
          <el-checkbox label="转发微博图片" name="type" />
          <el-checkbox label="原创微博视频" name="type" />
          <el-checkbox label="转发微博视频" name="type" />
        </el-checkbox-group>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit" v-loading.fullscreen.lock="fullscreenLoading" element-loading-text="微博爬取中，请稍等......"
                   element-loading-spinner="el-icon-loading">确认</el-button>
        <el-button @click="onCancel">清空</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { crawlerWeibo } from '@/api/weibo-crawler'

export default {
  data() {
    return {
      fullscreenLoading: false,
      no_use_name:'',
      no_use_type:[],
      no_use_select:false,
      form: {
        weibo_user_id: '',
        since_date: ''
      }
    }
  },
  methods: {
    onSubmit() {
      this.fullscreenLoading = true
      crawlerWeibo(this.form).then(response => {
        this.$message({
          message:'微博爬取完成，可去生成用户画像',
          type:'success',
          duration:3000
        })
        this.fullscreenLoading = false
        this.no_use_type = []
        this.no_use_name = ''
        this.no_use_select = false
        this.form['weibo_user_id'] = ''
        this.form['since_date'] = ''
      })
    },
    onCancel() {
      this.$message({
        message: 'cancel!',
        type: 'warning'
      })
    }
  }
}
</script>

<style scoped>
.line{
  text-align: center;
}
</style>

