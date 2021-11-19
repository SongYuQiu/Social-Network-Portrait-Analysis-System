<template>
  <div class="app-container">
    <div v-if="user">
      <el-row :gutter="20">

        <el-col :span="6" :xs="24">
          <user-card :user="user" />
        </el-col>

        <el-col :span="18" :xs="24">
          <el-card>
            <el-tabs v-model="activeTab" >
              <el-tab-pane label="性别｜年龄｜兴趣" name="activity">
                <activity v-if="activeTab==='activity'" />
              </el-tab-pane>
              <el-tab-pane label="微博定位" name="location">
                <location-map v-if="activeTab==='location'" />
              </el-tab-pane>
              <el-tab-pane label="时间轴" name="timeline">
                <timeline />
              </el-tab-pane>
            </el-tabs>
          </el-card>
        </el-col>

      </el-row>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import UserCard from './components/UserCard'
import Activity from './components/Activity'
import Timeline from './components/Timeline'
import Account from './components/Account'
import LocationMap from '@/views/profile/components/LocationMap'
import { getUserPortrait } from '@/api/weibo-user'
import * as echarts from 'echarts'
import agePieChart from '@/views/dashboard/admin/components/agePieChart'
import PieChart from '@/views/dashboard/admin/components/PieChart'

export default {
  name: 'Profile',
  components: { UserCard, Activity, Timeline, Account, LocationMap },
  data() {
    return {
      user: {},
      activeTab: 'activity',
      weibo_user_id:'',
    }
  },
  computed: {
    ...mapGetters([
      'name',
      'avatar',
    ])
  },
  created() {
    this.getUser()
  },
  methods: {
    getUser() {
      this.weibo_user_id = localStorage.getItem('weibo_user_id')
      getUserPortrait(this.weibo_user_id).then(response => {
        this.user = {
          name:response.data.screen_name,
          avatar: response.data.avatar_hd,
          weibo_user_id:response.data.weibo_user_id,
          description:response.data.description !== "" ? response.data.description : "暂无简介",
          verified:response.data.verified,
          verified_reason:response.data.verified ? response.data.verified_reason : "尚未认证",
          interest:response.data.interest,
          location:response.data.location,
          member_rank:response.data.member_rank,
          weibo_rank:response.data.weibo_rank,
          since_date:response.data.since_date.toString().split("T")[0],
          followers_count:this.numberFormat(response.data.followers_count),
          follow_count:this.numberFormat(response.data.follow_count),
          last_date:response.data.last_date.toString().split("T")[0],
          portrait_date:response.data.portrait_date.toString().split("T")[0],
          statuses_count:this.numberFormat(response.data.statuses_count),
        }
      })
    },
    numberFormat(value) {
      var param = {};
      var k = 10000,
        sizes = ['', '万', '亿', '万亿'],
        i;
      if(value < k){
        param.value =value
        param.unit=''
      }else{
        i = Math.floor(Math.log(value) / Math.log(k));

        param.value = ((value / Math.pow(k, i))).toFixed(2);
        param.unit = sizes[i];
      }
      return String(param.value) + String(param.unit);
    },
    handleClick(tab) {
      if(tab.name === 'activity') {
        agePieChart.mounted();
        PieChart.mounted();
      } else if(tab.name === 'location') {
        LocationMap.mounted();
      }
    }
  },
}
</script>
