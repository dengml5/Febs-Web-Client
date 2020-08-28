<template>
  <div class="app-container">
    <el-button class="filter-item" type="success" @click="add()" style="float:right;margin-right:60px">
        新增
    </el-button>
    <el-table
      ref="table"
      :key="tableKey"
      v-loading="loading"
      :data="this.list"
      border
      fit
      style="width: 100%"
      @selection-change="onSelectChange"
      @sort-change="sortChange"
    >
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          prop="id"
          label="id"
          width="100">
        </el-table-column>
        <el-table-column
          prop="artifactId"
          label="artifactId"
          width="120">
        </el-table-column>
        <el-table-column
          prop="cnvid"
          label="cnvid"
          width="120">
        </el-table-column>
        <el-table-column
          prop="createTime"
          :label="$t('table.user.createTime')"
          width="140">
        </el-table-column>
        <el-table-column
          prop="groupId"
          label="groupId"
          width="130">
        </el-table-column>
        <el-table-column
          prop="version"
          label="version"
          :filters="this.versionName"
          :filter-method="filterHandler"
          width="110">
        </el-table-column>
        <el-table-column
          prop="vulGrade"
          label="vulGrade"
          :filters="this.vulGradeName"
          :filter-method="filterHandler2"
          width="110">
        </el-table-column>
        <el-table-column
          prop="remark"
          label="remark"
          width="120">
        </el-table-column>
        <el-table-column :label="$t('table.operation')" align="center" width="150px" class-name="small-padding fixed-width">
          <template slot-scope="{row}">
            <i v-hasPermission="['user:update']" class="el-icon-setting table-operation" style="color: #2db7f5;" @click="edit(row)" />
            <i v-hasPermission="['user:delete']" class="el-icon-delete table-operation" style="color: #f50;" @click="singleDelete(row)" />
            <el-link v-has-no-permission="['user:view','user:update','user:delete']" class="no-perm">
              {{ $t('tips.noPermission') }}
            </el-link>
          </template>
        </el-table-column>
    </el-table>
    <pagination v-show="total>0" :total="total" :page.sync="pagination.num" :limit.sync="pagination.size" @pagination="search" />
    <user-edit
      ref="edit"
      :dialog-visible="dialog.isVisible"
      :title="dialog.title"
      @success="editSuccess"
      @close="editClose"
    />
  </div>
</template>
<script>
import Pagination from '@/components/Pagination'
import UserEdit from './Edit'
import UserView from './View'

export default {
  name: 'Pom',
  components: { Pagination, UserEdit, UserView  },
  data() {
    return {
      dialog: {
        isVisible: false,
        title: ''
      },
      tableKey: 0,
      loading: false,
      list: null,
      total: 0,
      queryParams: {},
      pagination: {
        size: 10,
        num: 1
      },
      datasourcesName: [],
      versionName: [],
      versionSet: [],
      vulGradeName: [],
      vulGradeSet: [],
      flag: false,
      selection: [],
    }
  },
  mounted() {
    this.fetch()
    // this.getDatasources()
    // this.versionName = this.uniqueTest(this.versionName)
  },
  methods: {
    search() {
      console.log('搜索成功')
      this.fetch({
        ...this.queryParams
      })
    },
    getDatasources() {
      console.log(123)
      this.$get('whitebox/whitebox').then((r) => {
        this.datasourcesName = r.data.data.rows
        console.log(this.datasourcesName)
      })
    },
    gen(row) {
      this.$download('generator', {
        name: row.name,
        datasource: this.queryParams.datasource,
        remark: row.remark
      }, `${row.name}_code.zip`)
    },
    reset() {
      this.queryParams = {}
      this.search()
    },
    fetch(params = {}) {
      this.loading = true
      params.pageSize = this.pagination.size
      params.pageNum = this.pagination.num
      this.$get('whitebox/whitebox', { ...params }).then((r) => {
        const data = r.data.data
        this.total = data.total
        this.list = data.rows
        if(!this.flag){
          for(var i=0;i<this.list.length;i++){
            this.versionSet.push(this.list[i].version);
          }
          this.versionSet = [...new Set(this.versionSet)]
          for(var i=0; i<this.versionSet.length; i++){
            this.versionName.push({text:this.versionSet[i],value:this.versionSet[i]});
          }

          for(var i=0;i<this.list.length;i++){
            this.vulGradeSet.push(this.list[i].vulGrade);
          }
          this.vulGradeSet = [...new Set(this.vulGradeSet)]
          for(var i=0; i<this.vulGradeSet.length; i++){
            this.vulGradeName.push({text:this.vulGradeSet[i],value:this.vulGradeSet[i]});
          }
          this.flag = true
        }
        this.loading = false
      })
    },
    filterHandler(value, row) {
         return row.version === value;
    },
    filterHandler2(value, row) {
         return row.vulGrade === value;
    },
    editClose() {
      this.dialog.isVisible = false
    },
    editSuccess() {
      this.search()
    },
    edit(row) {
        this.$refs.edit.setUser(row)
        this.dialog.title = this.$t('common.edit')
        this.dialog.isVisible = true
    },
     singleDelete(row) {
      this.$refs.table.toggleRowSelection(row, true)
      this.batchDelete()
    },
    batchDelete() {
      console.log('abc')
      if (!this.selection.length) {
        this.$message({
          message: this.$t('tips.noDataSelected'),
          type: 'warning'
        })
        return
      }
      let contain = false
      this.$confirm(this.$t('tips.confirmDelete'), this.$t('common.tips'), {
        confirmButtonText: this.$t('common.confirm'),
        cancelButtonText: this.$t('common.cancel'),
        type: 'warning'
      }).then(() => {
        const ids = []
        this.selection.forEach((u) => {
          ids.push(u.id)
        })
        if (contain) {
          this.$message({
            message: this.$t('tips.containCurrentUser'),
            type: 'warning'
          })
          this.clearSelections()
        } else {
          this.delete(ids)
        }
      }).catch(() => {
        this.clearSelections()
      })
    },
    clearSelections() {
      this.$refs.table.clearSelection()
    },
    delete(ids) {
      console.log(123)
      this.loading = true
      this.$delete(`/whitebox/whitebox/${ids}`).then(() => {
        this.$message({
          message: this.$t('tips.deleteSuccess'),
          type: 'success'
        })
        this.search()
      })
    },
    onSelectChange(selection) {
      this.selection = selection
    },
    add() {
      this.dialog.title = this.$t('common.add')
      this.dialog.isVisible = true
    },
    sortChange(val) {
      this.sort.field = val.prop
      this.sort.id = val.id
      this.search()
    }
  }
}
</script>
