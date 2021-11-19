<template>
  <el-card style="margin-bottom:20px;">
    <div slot="header" class="clearfix">
      <span>微博ID：{{ user.weibo_user_id }}</span>
      <svg-icon class="link-type" icon-class="delete" style="float: right" @click="deleteUserPortrait" />
    </div>

    <div class="user-profile">
      <div class="box-center">
        <pan-thumb :image="String(user.avatar)" :height="'100px'" :width="'100px'" :hoverable="false">
        </pan-thumb>
      </div>
      <div class="box-center">
        <div class="user-name text-center">{{ user.name }}</div>
      </div>
    </div>

    <div class="user-bio">
      <div class="user-education user-bio-section">
        <div class="user-bio-section-header"><svg-icon icon-class="education" /><span>个人简介</span></div>
        <div class="user-bio-section-body">
          <div class="text-muted">
            {{ user.description }}
          </div>
        </div>
      </div>

      <div class="user-education user-bio-section">
        <div class="user-bio-section-header"><svg-icon icon-class="verified" /><span>认证信息</span></div>
        <div class="user-bio-section-body">
          <div class="text-muted">
            {{ user.verified_reason }}
          </div>
        </div>
      </div>

      <div class="user-skills user-bio-section">
        <div class="user-bio-section-header"><svg-icon icon-class="star" /><span>个人微博数据</span></div>
        <div class="user-bio-section-body" style="flex-direction: row;display: flex;padding-top: 10px">
          <div class="user-profile-data">
            <span>{{ user.statuses_count }}</span>
            <span>微博</span>
          </div>
          <div class="user-profile-data">
            <span>{{ user.followers_count }}</span>
            <span>粉丝</span>
          </div>
          <div class="user-profile-data">
            <span>{{ user.follow_count }}</span>
            <span>关注</span>
          </div>
          <div class="user-profile-data">
            <span>{{ user.weibo_rank }}</span>
            <span>等级</span>
          </div>
          <div class="user-profile-data">
            <span>{{ user.member_rank }}</span>
            <span>会员</span>
          </div>
        </div>
      </div>

      <div class="user-education user-bio-section">
        <div class="user-bio-section-header"><svg-icon icon-class="calendar" /><span>画像生成日期</span></div>
        <div class="user-bio-section-body">
          <div class="text-muted">
            {{ user.portrait_date }}
          </div>
        </div>
      </div>

      <div class="user-education user-bio-section">
        <div class="user-bio-section-header"><svg-icon icon-class="timeline" /><span>微博爬取范围</span></div>
        <div class="user-bio-section-body">
          <div class="text-muted">
            {{ user.since_date }} —— {{ user.last_date }}
          </div>
        </div>
      </div>

    </div>
  </el-card>
</template>

<script>
import PanThumb from '@/components/PanThumb/index'
import { deleteUserPortrait } from '@/api/weibo-user'

export default {
  components: { PanThumb },
  props: {
    user: {
      type: Object,
      default: () => {
        return {
          name: '',
          email: '',
          avatar: '',
          role: ''
        }
      }
    }
  },
  methods:{
    deleteUserPortrait(){
      const that = this
      that.$confirm('此操作将删除该微博用户的用户画像, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteUserPortrait(that.user.weibo_user_id).then(response => {
          if(response.status === 200){
            that.$message({
              message:'删除成功！',
              type:'success',
              duration:1000
            })
            this.$router.push({ path: '/documentation/index'})
          }
        })
      }).catch(() => {
        that.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    }
  }
}
</script>

<style lang="scss" scoped>
.box-center {
  margin: 0 auto;
  display: table;
}

.text-muted {
  color: #777;
}

.user-profile {
  .user-name {
    font-weight: bold;
  }

  .box-center {
    padding-top: 10px;
  }

  .user-role {
    padding-top: 10px;
    font-weight: 400;
    font-size: 14px;
  }

  .box-social {
    padding-top: 30px;

    .el-table {
      border-top: 1px solid #dfe6ec;
    }
  }

  .user-follow {
    padding-top: 20px;
  }
}

.user-bio {
  margin-top: 20px;
  color: #606266;

  span {
    padding-left: 4px;
  }

  .user-bio-section {
    font-size: 14px;
    padding: 15px 0;

    .user-bio-section-header {
      border-bottom: 1px solid #dfe6ec;
      padding-bottom: 10px;
      margin-bottom: 10px;
      font-weight: bold;
    }
  }
}

.user-profile-data{
  flex-direction: column;
  display: flex;
  text-align: center;
  margin: auto;
}
</style>
