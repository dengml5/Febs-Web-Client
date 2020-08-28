<template>
  <el-dialog
    :title="title"
    :width="width"
    top="50px"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :visible.sync="isVisible"
  >
    <el-form ref="form" :model="user" :rules="rules" label-position="right" label-width="100px">
      <el-form-item :label="$t('table.user.artifactId')" prop="artifactId">
        <el-input v-model="user.artifactId" />
      </el-form-item>
      <el-form-item :label="$t('table.user.cnvdId')" prop="cnvdId">
        <el-input v-model="user.cnvdId" />
      </el-form-item>
      <el-form-item :label="$t('table.user.groupId')" prop="groupId">
        <el-input v-model="user.groupId" />
      </el-form-item>
      <el-form-item :label="$t('table.user.remark')" prop="remark">
        <el-input v-model="user.remark" />
      </el-form-item>
      <el-form-item :label="$t('table.user.version')" prop="version">
        <el-input v-model="user.version" />
      </el-form-item>     
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button type="warning" plain :loading="buttonLoading" @click="isVisible = false">
        {{ $t('common.cancel') }}
      </el-button>
      <el-button type="primary" plain :loading="buttonLoading" @click="submitForm">
        {{ $t('common.confirm') }}
      </el-button>
    </div>
  </el-dialog>
</template>
<script>
import { validMobile } from '@/utils/my-validate'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'

export default {
  name: 'UserEdit',
  components: { Treeselect },
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      initFlag: false,
      user: this.initUser(),
      buttonLoading: false,
      screenWidth: 0,
      width: this.initWidth(),
      depts: [],
      roles: [],
      deptTree: [],
      rules: {
        
      }
    }
  },
  computed: {
    isVisible: {
      get() {
        return this.dialogVisible
      },
      set() {
        this.close()
        this.reset()
      }
    }
  },
  mounted() {
    this.initDept()
    this.initRoles()
    window.onresize = () => {
      return (() => {
        this.width = this.initWidth()
      })()
    }
  },
  methods: {
    initUser() {
      return {
        artifactId: '',
        cnvdId: '',
        groupId: '',
        remark: '',
        version: '',
        vulGrade: '',
      }
    },
    initWidth() {
      this.screenWidth = document.body.clientWidth
      if (this.screenWidth < 991) {
        return '90%'
      } else if (this.screenWidth < 1400) {
        return '45%'
      } else {
        return '800px'
      }
    },
    initDept() {
      this.$get('system/dept').then((r) => {
        this.depts = r.data.data.rows
        this.deptTree = this.depts
      }).catch((error) => {
        console.error(error)
        this.$message({
          message: this.$t('tips.getDataFail'),
          type: 'error'
        })
      })
    },
    // resetDeptTree() {
    //   this.$refs.deptTree.setCheckedKeys([])
    // },
    initRoles() {
      this.$get('system/role/options').then((r) => {
        this.roles = r.data.data
      }).catch((error) => {
        console.error(error)
        this.$message({
          message: this.$t('tips.getDataFail'),
          type: 'error'
        })
      })
    },
    setUser(val) {
      this.user.artifactId = val.artifactId
      this.user.cnvdId = val.cnvdId
      this.user.groupId = val.groupId
      this.user.remark = val.remark
      this.user.version = val.version
      this.user.vulGrade = val.vulGrade
      this.user.id = val.id
    },
    close() {
      this.$emit('close')
    },
    submitForm() {
      this.$refs.form.validate((valid) => {
        console.log(this.title)
        if(this.user.id){
          this.$put('/whitebox/whitebox', { ...this.user }).then(() => {
                this.buttonLoading = false
                this.isVisible = false
                this.$emit('success')
                this.$message({
                  message: this.$t('tips.updateSuccess'),
                  type: 'success'
                })
              })
        }else{
          this.$post('/whitebox/whitebox', { ...this.user }).then(() => {
              this.buttonLoading = false
              this.isVisible = false
              this.$message({
                message: this.$t('tips.createSuccess'),
                type: 'success'
              })
              this.$emit('success')
            })
        }
      })
    },
    reset() {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$refs.form.clearValidate()
      this.$refs.form.resetFields()
      this.user = this.initUser()
    }
  }
}
</script>
<style lang="scss" scoped>
</style>
