<template>
  <div class="app-container">

    <div style="margin-bottom: 10px;font-size: 10px;">选择：{{ baseMap.hosname }}  /  {{ depname }}  /  {{ workDate }}</div>
    <el-container style="height: 100%">
      <el-aside width="200px" style="border: 1px silver solid">
        <el-tree
          :data="data"
          :props="defaultProps"
          :default-expand-all="true"
          @node-click="handleNodeClick">
        </el-tree>
      </el-aside>
      <el-main style="padding: 0 0 0 20px;">
        <el-row style="width: 100%">
          <el-tag v-for="(item,index) in bookingScheduleList" :key="item.id" @click="selectDate(item.workDate, index)" :type="index == activeIndex ? '' : 'info'" style="height: 60px;margin-right: 5px;margin-right:15px;cursor:pointer;">
            {{ item.workDate }} {{ item.dayOfWeek }}<br/>
            {{ item.depNumber == 0 ? '约满' : item.availableNumber }} / {{ item.reservedNumber }}
          </el-tag>

          <!-- 分页 -->
          <el-pagination
            :current-page="page"
            :total="total"
            :page-size="limit"
            class="pagination"
            layout="prev, pager, next"
            @current-change="getPage">
          </el-pagination>
        </el-row>

        <el-row style="margin-top: 20px;">
          <el-table
            v-loading="listLoading"
            :data="scheduleList"
            border
            fit
            highlight-current-row>

            <el-table-column
              label="序号"
              width="60"
              align="center">
              <template slot-scope="scope">
                {{ scope.$index + 1 }}
              </template>
            </el-table-column>
            <el-table-column label="职称" width="150">
              <template slot-scope="scope">
                {{ scope.row.title }} | {{ scope.row.docname }}
              </template>
            </el-table-column>
            <el-table-column label="号源时间" width="80">
              <template slot-scope="scope">
                {{ scope.row.workTime == 0 ? "上午" : "下午" }}
              </template>
            </el-table-column>
            <el-table-column prop="reservedNumber" label="可预约数" width="80"/>
            <el-table-column prop="availableNumber" label="剩余预约数" width="100"/>
            <el-table-column prop="amount" label="挂号费(元)" width="90"/>
            <el-table-column prop="skill" label="擅长技能" />
          </el-table>
        </el-row>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import departmentApi from '@/api/hosp/department'
import scheduleApi from '@/api/hosp/schedule'

export default {
  data() {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'depname'
      },

      activeIndex: 0,
      hoscode: null,
      depcode: null,
      depname: null,
      workDate: null,

      listLoading: false, // 数据是否正在加载
      bookingScheduleList: [],
      scheduleList: [],
      baseMap: {},
      nextWorkDate: null, // 下一页第一个日期
      preWorkDate: null, // 上一页第一个日期

      page: 1, // 当前页
      limit: 7, // 每页个数
      total: 0 // 总页码
    }
  },

  // 生命周期函数：内存准备完毕，页面尚未渲染
  created() {
    console.log('list created......')
    this.hoscode = this.$route.params.hoscode
    this.workDate = this.getCurDate()

    this.fetchData()
  },

  // 生命周期函数：内存准备完毕，页面渲染成功
  mounted() {
    console.log('list mounted......')
  },

  methods: {
    fetchData() {
      departmentApi.getTree(this.hoscode).then(response => {
        this.data = response.data

        if (this.data.length > 0) {
          this.depcode = this.data[0].children[0].depcode
          this.depname = this.data[0].children[0].depname

          this.getPage()
        }
      })
    },

    getPage(page = 1) {
      this.page = page
      this.workDate = null
      this.activeIndex = 0

      this.getScheduleRule()
    },

    getScheduleRule() {
      scheduleApi.getScheduleRule(this.page, this.limit, this.hoscode, this.depcode).then(response => {
        this.bookingScheduleList = response.data.bookingScheduleList
        this.total = response.data.total
        this.scheduleList = response.data.scheduleList
        this.baseMap = response.data.baseMap

        // 分页后workDate=null，默认选中第一个
        if (this.workDate == null) {
          this.workDate = this.bookingScheduleList[0].workDate
        }
        this.findScheduleList()
      })
    },

    findScheduleList() {
      scheduleApi.findScheduleList(this.hoscode, this.depcode, this.workDate).then(response => {
        this.scheduleList = response.data
      })
    },

    handleNodeClick(data) {
      // debugger
      // 科室大类直接返回
      if (data.children != null) return
      this.depcode = data.depcode
      this.depname = data.depname

      this.getScheduleRule()
    },

    selectDate(workDate, index) {
      this.workDate = workDate
      this.activeIndex = index

      this.getScheduleRule()
    },

    getCurDate() {
      var datetime = new Date()
      var year = datetime.getFullYear()
      var month = datetime.getMonth() + 1 < 10 ? '0' + (datetime.getMonth() + 1) : datetime.getMonth() + 1
      var date = datetime.getDate() < 10 ? '0' + datetime.getDate() : datetime.getDate()
      return year + '-' + month + '-' + date
    }
  }
}
</script>
<style>
  .el-tree-node.is-current > .el-tree-node__content {
    background-color: #409EFF !important;
    color: white;
  }

  .el-checkbox__input.is-checked+.el-checkbox__label {
    color: black;
  }
</style>
