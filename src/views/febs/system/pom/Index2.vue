<template>
  <div class="main">
      <el-table
        :data="this.pomData"
        style="width: 100%"
        max-height="100%">
        <el-table-column
          fixed
          prop="id"
          label="id"
          :filters="[{text: '1', value: '1'}]"
          :filter-method="filterHandler"
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
          label="createTime"
          width="130">
        </el-table-column>
        <el-table-column
          prop="groupId"
          label="groupId"
          width="130">
        </el-table-column>
        <el-table-column
          prop="remark"
          label="remark"
          width="120">
        </el-table-column>
        <el-table-column
          prop="version"
          label="version"
          width="120">
        </el-table-column>
      </el-table>
  </div>
</template>

<script>
export default {
    name: 'Pom',
    data() {
      return {
        pomData: []
      }
    },
    mounted() {
      this.test()
    },
    methods: {
      test() {
        console.log(123),
         this.$get('whitebox/whitebox').then((r) => {
             this.pomData = r.data.data.rows;
             console.log(r)
             console.log(this.pomData)
            })
      },
       filterHandler(value, row, column) {
        const property = column['property'];
        return row[property] === value;
      }
    }
}

</script>
<style scoped>
.main {
  display: flex;
  justify-content: center;
}
</style>