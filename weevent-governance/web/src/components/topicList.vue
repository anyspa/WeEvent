<template>
<div class='event-table topic'>
  <div class='refresh'>
    <el-button type='primary' size='small' icon='el-icon-plus' @click='addNewOne'>{{$t('common.add')}}</el-button>
    <div class='search_part'>
      <el-input v-model.trim='topicName'
        :placeholder="$t('tableCont.searchTopic')"
        size='small'
        clearable
      ></el-input>
      <el-button type='primary' size='small' @click='searchTopic'>{{$t('common.search')}}</el-button>
    </div>
  </div>
  <el-table
    :data="tableData"
    v-loading='loading'
    element-loading-spinner='el-icon-loading'
    :element-loading-text="$t('common.loading')"
    element-loading-background='rgba(256,256,256,0.8)'
    style="width: 100%"
    @row-click='rowClick'
    @expand-change='readDetail'
    ref='table'
    >
    <el-table-column type="expand">
      <template slot-scope="props">
        <el-form label-position="left" inline class="demo-table-expand">
          <el-form-item label="Topic :">
            <span>{{ props.row.detail.topicName }}</span>
          </el-form-item><br/>
          <el-form-item :label="$t('tableCont.timestamp')  + ' :'">
            <span>{{ props.row.detail.createdTimestamp }}</span>
          </el-form-item><br/>
           <el-form-item :label="$t('tableCont.sequenceNumber')  + ' :'">
            <span>{{ props.row.detail.sequenceNumber }}</span>
          </el-form-item><br/>
           <el-form-item :label="$t('tableCont.newBlockNumber')  + ' :'">
            <span>{{ props.row.detail.blockNumber }}</span>
          </el-form-item><br/>
          <el-form-item :label="$t('tableCont.lastTimestamp')  + ' :'">
            <span>{{ props.row.detail.lastTimestamp }}</span>
          </el-form-item><br/>
          <el-form-item :label="$t('tableCont.address')  + ' :'" v-show="props.row.detail.topicAddress">
            <span>{{ props.row.detail.topicAddress }}</span>
          </el-form-item>
        </el-form>
      </template>
    </el-table-column>
    <el-table-column
      label="Topic"
      prop="topicName"
      :formatter="checkName">
    </el-table-column>
    <el-table-column
      :label="$t('tableCont.creater')"
      prop="creater"
      :formatter="checkCreater">
    </el-table-column>
     <el-table-column
      :label="$t('tableCont.timestamp')"
      prop="createdTimestamp"
      :formatter="checkTime">
    </el-table-column>
  </el-table>
  <el-pagination
    @current-change="indexChange"
    @size-change='sizeChange'
    :current-page="pageIndex"
    :page-sizes="[10, 20, 50]"
    layout="sizes,total, prev, pager, next, jumper"
    :total="total">
  </el-pagination>
  <el-dialog :title="$t('tableCont.addTopic')" :visible.sync="dialogFormVisible" center width='450px' :close-on-click-modal='false'>
    <el-form :model="form" :rules="rules" ref='form'>
      <el-form-item :label="$t('common.name') + ' :'" prop='name'>
        <el-input v-model="form.name" autocomplete="off"></el-input>
      </el-form-item>
      <el-form-item :label="$t('common.detail') + ' :'">
        <el-input v-model="form.describe" type='textarea' autocomplete="off"></el-input>
      </el-form-item>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click='addTopic(form)'>{{$t('common.ok')}}</el-button>
      <el-button @click="dialogFormVisible = false">{{$t('common.cancel')}}</el-button>
    </div>
  </el-dialog>
 </div>
</template>
<script>
import API from '../API/resource.js'
import { getDateDetail } from '../utils/formatTime'
export default {
  data () {
    var name = (rule, value, callback) => {
      if (value === '') {
        callback(new Error(this.$t('tableCont.noName')))
      } else {
        callback()
      }
    }
    return {
      topicName: '',
      loading: false,
      dialogFormVisible: false,
      tableData: [],
      pageIndex: 1,
      pageSize: 10,
      total: 0,
      form: {
        name: '',
        describe: ''
      },
      rules: {
        name: [
          { required: true, validator: name, trigger: 'blur' }
        ]
      },
      creater: ''
    }
  },
  methods: {
    // 数据获取
    getLastPage () {
      const vm = this
      vm.loading = true
      const data = {
        pageIndex: 0,
        pageSize: vm.pageSize,
        brokerId: Number(localStorage.getItem('brokerId')),
        groupId: Number(localStorage.getItem('groupId'))
      }
      API.topicList(data).then(res => {
        if (res.status === 200) {
          vm.total = res.data.data.total
          const last = Math.ceil(res.data.data.total / vm.pageSize)
          this.pageIndex = last
          const data = {
            pageIndex: last - 1,
            pageSize: vm.pageSize,
            brokerId: Number(localStorage.getItem('brokerId')),
            groupId: Number(localStorage.getItem('groupId'))
          }
          API.topicList(data).then(res => {
            if (res.status === 200) {
              const listData = res.data.data.topicInfoList.reverse()
              const det = {
                topicName: '',
                createdTimestamp: '',
                topicAddress: '',
                lastTimestamp: ''
              }
              listData.forEach(item => {
                vm.$set(item, 'detail', det)
              })
              vm.tableData = [].concat(listData)
            }
          })
        }
        vm.loading = false
      }).catch(e => {
        vm.loading = false
      })
    },
    getLsitData () {
      const vm = this
      vm.loading = true
      const data = {
        pageIndex: vm.pageIndex - 1,
        pageSize: vm.pageSize,
        brokerId: Number(localStorage.getItem('brokerId')),
        groupId: Number(localStorage.getItem('groupId'))
      }
      API.topicList(data).then(res => {
        if (res.status === 200) {
          vm.total = res.data.total
          const listData = res.data.topicInfoList.reverse()
          const det = {
            topicName: '',
            createdTimestamp: '',
            topicAddress: '',
            lastTimestamp: ''
          }
          listData.forEach(item => {
            vm.$set(item, 'detail', det)
          })
          vm.tableData = [].concat(listData)
        }
        vm.loading = false
      }).catch(e => {
        vm.loading = false
      })
    },
    refresh () {
      sessionStorage.removeItem('topic')
      this.loading = true
      setTimeout(fun => {
        this.getLastPage()
      }, 1000)
    },
    readDetail (e) {
      var vm = this
      const url = '?brokerId=' + localStorage.getItem('brokerId') + '&groupId=' + localStorage.getItem('groupId') + '&topic=' + e.topicName
      API.topicState(url).then(res => {
        const time = getDateDetail(res.data.data.createdTimestamp)
        res.data.createdTimestamp = time
        res.data.lastTimestamp = getDateDetail(res.data.data.lastTimestamp)
        vm.$set(e, 'detail', res.data)
      })
    },
    indexChange (e) {
      this.pageIndex = e
      this.getLsitData()
    },
    sizeChange (e) {
      this.pageSize = e
      this.getLastPage()
    },
    // check formart
    checkName (e) {
      if (!e.topicName) {
        return '—'
      } else {
        return e.topicName
      }
    },
    checkCreater (e) {
      if (!e.creater || (e.creater === 'unknow')) {
        return '—'
      } else {
        return e.creater
      }
    },
    checkTime (e) {
      const time = e.createdTimestamp
      return getDateDetail(time)
    },
    addNewOne () {
      this.dialogFormVisible = true
    },
    addTopic (form) {
      const vm = this
      vm.$refs.form.validate((valid) => {
        if (valid) {
          const data = {
            topic: vm.form.name,
            creater: localStorage.getItem('user'),
            brokerId: Number(localStorage.getItem('brokerId')),
            groupId: Number(localStorage.getItem('groupId')),
            description: vm.form.describe
          }
          API.openTopic(data).then(res => {
            if (res.data.status === 200) {
              vm.$message({
                type: 'success',
                message: this.$t('common.addSuccess')
              })
              vm.refresh()
            } else if (res.data.status === 100109) {
              vm.$store.commit('set_Msg', vm.$message({
                type: 'warning',
                message: this.$t('tableCont.exitTopic'),
                duration: 0,
                showClose: true
              }))
            } else {
              vm.$store.commit('set_Msg', vm.$message({
                type: 'warning',
                message: res.data.message,
                duration: 0,
                showClose: true
              }))
            }
            vm.dialogFormVisible = false
          }).catch(e => {
            vm.$store.commit('set_Msg', vm.$message({
              type: 'warning',
              message: this.$t('common.addFail'),
              duration: 0,
              showClose: true
            }))
          })
          vm.dialogFormVisible = false
        } else {
          return false
        }
      })
    },
    searchTopic () {
      var vm = this
      if (vm.topicName) {
        vm.tableData = []
        const url = '?brokerId=' + localStorage.getItem('brokerId') + '&groupId=' + localStorage.getItem('groupId') + '&topic=' + vm.topicName
        API.topicInfo(url).then(res => {
          const time = getDateDetail(res.data.createdTimestamp)
          res.data.createdTimestamp = time
          const item = {
            topicName: res.data.topicName,
            creater: '——',
            createdTimestamp: time,
            detail: {}
          }
          vm.tableData.push(item)
          vm.total = 1
        })
      } else {
        vm.getLastPage()
      }
    },
    rowClick (e) {
      this.$refs.table.toggleRowExpansion(e)
      this.readDetail(e)
    }
  },
  mounted () {
    // if the data is exit so it means click form subscribtion list
    if (sessionStorage.getItem('topic')) {
      var vm = this
      vm.tableData = []
      const url = '?brokerId=' + localStorage.getItem('brokerId') + '&groupId=' + localStorage.getItem('groupId') + '&topic=' + sessionStorage.getItem('topic')
      API.topicInfo(url).then(res => {
        const time = getDateDetail(res.data.createdTimestamp)
        res.data.createdTimestamp = time
        const item = {
          topicName: res.data.topicName,
          creater: '——',
          createdTimestamp: time,
          detail: {}
        }
        vm.tableData.push(item)
        vm.total = 1
      })
    } else {
      this.getLastPage()
    }
  },
  computed: {
    brokerId () {
      return this.$store.state.brokerId
    },
    groupId () {
      return this.$store.state.groupId
    }
  },
  watch: {
    dialogFormVisible (nVal) {
      if (!nVal) {
        this.form.name = ''
        this.form.describe = ''
        this.$refs.form.resetFields()
      }
    },
    groupId (nVal) {
      if (nVal !== '-1') {
        this.tableData = []
        this.topicName = ''
        this.refresh()
      }
    }
  },
  destroyed () {
    sessionStorage.removeItem('topic')
  }
}
</script>
