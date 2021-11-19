<template>
  <div class="app-container">

    <panel-group :all_weibo_user_count="this.all_weibo_user_count" :all_weibo_text_count="this.all_weibo_text_count" :all_portrait_count="this.all_portrait_count" />

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
      v-loading.fullscreen.lock="fullscreenLoading"
      element-loading-text="画像生成中，请稍等......"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(255, 255, 255, 0.8)"
    >
      <el-table-column label="ID" prop="id" align="center" width="80">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="微博ID" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.weibo_user_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="微博名" width="250px">
        <template slot-scope="{row}">
          <span class="link-type">{{ row.screen_name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="爬取微博数" width="100px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.crawler_num }}</span>
        </template>
      </el-table-column>
      <el-table-column label="爬取起始日期" min-width="121px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.since_date }}</span>
        </template>
      </el-table-column>
      <el-table-column label="上次爬取日期" align="center" min-width="121px">
        <template slot-scope="{row}">
          <span>{{ row.last_date }}</span>
        </template>
      </el-table-column>
      <el-table-column label="画像生成状态" class-name="status-col" width="120">
        <template slot-scope="{row}">
          <el-tag :type="row.portrait_status?'success':'warning'">
            {{ row.portrait_status }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <el-button :disabled="row.portrait_status" type="primary" size="mini" @click="handleUpdate(row.weibo_user_id)">
            生成
          </el-button>
          <el-button :disabled="!row.portrait_status" size="mini" type="success" @click="handleModifyStatus(row.weibo_user_id)">
            查看
          </el-button>
          <el-button size="mini" type="danger" @click="handleDelete(row.weibo_user_id)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" />
  </div>
</template>

<script>
import { fetchPv, createArticle, updateArticle } from '@/api/article'
import { fetchList, deleteWeiboUser, createUserPortrait, fetchDataCount } from '@/api/weibo-user'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination/index' // secondary package based on el-pagination
import PanelGroup from '@/views/dashboard/admin/components/PanelGroup'

const calendarTypeOptions = [
  { key: 'CN', display_name: 'China' },
  { key: 'US', display_name: 'USA' },
  { key: 'JP', display_name: 'Japan' },
  { key: 'EU', display_name: 'Eurozone' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'ComplexTable',
  components: { Pagination, PanelGroup },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  data() {
    return {
      fullscreenLoading: false,
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: false,
      all_weibo_user_count:0,
      all_weibo_text_count:0,
      all_portrait_count:0,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: '+id'
      },
      importanceOptions: [1, 2, 3],
      calendarTypeOptions,
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
      statusOptions: ['published', 'draft', 'deleted'],
      showReviewer: false,
      temp: {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        type: '',
        status: 'published'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        type: [{ required: true, message: 'type is required', trigger: 'change' }],
        timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      },
      downloadLoading: false
    }
  },
  created() {
    this.getList()
    this.get_all_data_count()
  },
  methods: {
    getList() {
      // this.listLoading = true
      fetchList().then(response => {

        this.list = response.data
        for (let i =0;i<this.list.length;i++){
          this.list[i]["id"]=i+1
          this.list[i]["since_date"] = this.list[i]["since_date"].toString().split("T")[0]
          this.list[i]["last_date"] = this.list[i]["last_date"].toString().split("T")[0]
        }
        for (let j=0;j<this.list.length;j++){
          if (this.list[j]["portrait_status"]){
            localStorage.setItem("weibo_user_id",this.list[j]['weibo_user_id'])
            break;
          }
        }

      })
    },
    get_all_data_count(){
      fetchDataCount().then(response => {
        // console.log(response)
        this.all_weibo_user_count = response.data.all_user_count
        this.all_weibo_text_count = response.data.all_weibo_count
        this.all_portrait_count = response.data.all_portrait_count
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleModifyStatus(weibo_user_id) {
      localStorage.setItem("weibo_user_id",weibo_user_id)
      this.$router.push({ path: '/portrait/index'})
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    resetTemp() {
      this.temp = {
        id: undefined,
        importance: 1,
        remark: '',
        timestamp: new Date(),
        title: '',
        status: 'published',
        type: ''
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createArticle(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleUpdate(weibo_user_id) {
      this.fullscreenLoading = true
      createUserPortrait(weibo_user_id).then(response =>{
        if (response.status === 200){
          this.$message({
            message:'画像生成成功，可点击查看！',
            type:'success',
            duration:1000
          })
          this.getList()
        } else {
          this.$message({
            message:'不好，哪里出错了，画像生成失败！',
            type:'error',
            duration:1000
          })
        }
        this.fullscreenLoading = false
      })
      // this.temp = Object.assign({}, row) // copy obj
      // this.temp.timestamp = new Date(this.temp.timestamp)
      // this.dialogStatus = 'update'
      // this.dialogFormVisible = true
      // this.$nextTick(() => {
      //   this.$refs['dataForm'].clearValidate()
      // })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateArticle(tempData).then(() => {
            const index = this.list.findIndex(v => v.id === this.temp.id)
            this.list.splice(index, 1, this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleDelete(weibo_user_id) {
      const that = this
      that.$confirm('此操作将删除该微博用户的所有数据, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deleteWeiboUser(weibo_user_id).then(response => {
          if(response.status === 200){
            that.$message({
              message:'删除成功！',
              type:'success',
              duration:1000
            })
            that.getList()
            that.get_all_data_count()
          }
        })
      }).catch(() => {
        that.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
        const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal) {
      return this.list.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}` ? 'ascending' : 'descending'
    }
  }
}
</script>
