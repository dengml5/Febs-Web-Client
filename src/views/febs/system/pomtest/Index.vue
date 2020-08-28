<template>
<div>
  <div>
     <el-upload
        class="upload"
        :action="uploadAction"
        :headers="headers"
        list-type="picture"
        :show-file-list="false"
        :before-upload="beforeUpload"
        :on-success="uploadSuccess"
        :on-error="uploadError"
        :on-progress="uploadProgress"
        style="margin-left:50px; margin-top:5px "
      >
        <el-button class="filter-item" icon="el-icon-upload2" type="success" plain>
          {{ $t('table.import') }}
        </el-button>
      </el-upload>
  </div>
  <div class="shuttle">
    <div class="shuttle_item">
      <span>pom列表</span>
      <div class="school" style="margin-top: 15px">
        <el-table 
        ref="table"
        :key="tableKey"
        v-loading="loading"
        v-show="this.data.length>0"
        :data="this.data"
        border
        fit
        >
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="key"
            label="key"
            align="center"
            >
          </el-table-column>
          <el-table-column
          width="60px"
          label="操作">
            <template slot-scope="{row,$index}">
                <img v-show="row.children !== undefined" style="width:30px;" src="@/assets/rightarrow.png"  @click="childList($index)" alt="">
            </template>
          </el-table-column>
        </el-table>
    </div>
    </div>

    <div class="shuttle_item" style="margin-left:20px">
      <span>pom详情</span>
      <ul class="school">
        <list :list="childli"></list>
      </ul>
    </div>

    <div class="shuttle_item"  style="margin-left:20px">
      <span>问题详情</span>
      <ul class="school">
        <li v-for="(item,index) in leader" :key="index">
          <input type="checkbox" :id="item.userId" :value="item.userId"
                 v-model="teamLeader"
                 :disabled="schoolsNames.length>0||teamName.length>0?true:false"
          >
          <label :for="item.userId">{{item.username}}</label>
        </li>
      </ul>
    </div>
  </div>
</div>
</template>
 
<script>
import { getToken } from '@/utils/auth'
import { getFileType } from '@/utils'
import NProgress from 'nprogress'
import 'nprogress/nprogress.css'
import arrow from '@/assets/rightarrow.png'

import List from "./List";

  export default {
    name: "shuttle",
    components: { List },
    data() {
      return {
        allSchool: [{userId:1,username:'北京',test:'阿拉蕾'}, {userId:2,username:'上海'},{userId:3,username:'深圳'},],//所有学校
        allTeam: [],  //所有小组
        leader:[],// 组长
        schoolsNames: [],   //所有学校 选中的
        teamName: [],   //小组成员  选中的
        teamLeader: [],   //小组组长  选中的
        fileName: null,
        leftList: [],
        childli:[],
        
        uploadAction: `${process.env.VUE_APP_BASE_API}whitebox/projJavaInfo/importFile`,
        dialogVisible: false,
        tableKey: 0,
        loading: false,
        list: null,
        total: 0,
        pagination: {
          size: 10,
          num: 1
        },
        fileList: [],
        headers: {
         Authorization: `bearer ${getToken()}`
         },
        data: [],
        error: [],
        time: '0 ms',
      }
    },
    props: ["schoolInfo","teamInfo","leaderInfo"],
    created() {
    },
    mounted() {
    },
    watch: {
      schoolInfo(val) {   //  编辑时用来回显，添加时的默认数据
        this.schoolInfo = val;
        this.allSchool = val;
      },
      teamInfo(val){   //  编辑时用来回显
        this.teamInfo = val;
        this.allTeam=val;
      },
      leaderInfo(val){   //  编辑时用来回显
        this.leaderInfo=val;
        this.leader=val;
      },
    },
    methods: {
      childList(index) {
        console.log(index)
        console.log(123)
        this.childli=this.data[index].children
        console.log(this.data[index].children)
      },
      handleNodeClick(data) {
          console.log(data);
      },
     beforeUpload(file) {
      const type = getFileType(file.name)
      // if (type !== 'xlsx') {
      //   this.$message({
      //     message: this.$t('tips.onlySupportXlsx'),
      //     type: 'error'
      //   })
      //   return false
      // } else {
      //   return true
      // }
      return true
    },
    uploadError() {
      console.log("失败")
      this.$message({
        message: this.$t('tips.uploadFailed'),
        type: 'error'
      })
      NProgress.done()
    },
    uploadSuccess(response) {
      console.log(response)
      this.data = response
      this.$message({
        message: this.$t('tips.uploadSuccess'),
        type: 'success'
      })
      NProgress.done()
    },
    uploadProgress() {
      NProgress.start()
    },

      async toRightTeam() {
        let moveName= await this.matching(this.allSchool,this.schoolsNames); //两数组相同
        let allArr= await this.myFilter(this.allSchool,moveName); //两数组删除相同
        this.allSchool = allArr; //删除之后的数组
        for(let i=0;i<moveName.length;i++){
          this.allTeam.push(moveName[i])
        }
        this.schoolsNames=[];
        await this.putParentsTeams();
      },
      async toLeftSchools() {
        let moveName= await this.matching(this.allTeam,this.teamName);
        let allArr =await this.myFilter(this.allTeam,moveName);
        this.allTeam = allArr;
        for(let i=0;i<moveName.length;i++){
          this.allSchool.push(moveName[i])
        }
        this.teamName=[];
        await this.putParentsTeams();
      },
      async toRightLeader() {
        // if (this.leader.length >= 1||this.teamName.length>1) {
        //   this.$message({
        //     message: "小组组长只能选一个",
        //     type: "error"
        //   });
        //   return false;
        // }
        let moveName= await this.matching(this.allTeam,this.teamName);
        let allArr =await this.myFilter(this.allTeam,moveName);
        this.allTeam = allArr;
        for(let i=0;i<moveName.length;i++) {
          this.leader.push(moveName[i])
        }
        this.teamName = [];
        await this.putParentsTeams();
      },
      async toLeftTeam() {
        let moveName= await this.matching(this.leader,this.teamLeader);
        let allArr =await this.myFilter(this.leader,moveName);
        this.leader = allArr;
        for(let i=0;i<moveName.length;i++){
          this.allTeam.push(moveName[i])
        }
        this.teamLeader=[];
        await this.putParentsTeams();
      },
      // 过滤 相同选项
      async myFilter(allArr, selArr) {
        let ary03 = [];
        for (let i = 0; i < allArr.length; i++) {
          for (let j = 0; j < selArr.length; j++) {
            if (allArr[i].userId == selArr[j].userId) {
              allArr.splice(i,1);
            }
          }
        }
        return allArr;
      },
      //  匹配 移动的 选项
      async matching(allArr, matchArr) {
        let matArr = [];
        for (let i = 0; i < allArr.length; i++) {
          for (let j = 0; j < matchArr.length; j++) {
            if (allArr[i].userId == matchArr[j]) {
              matArr.push(allArr[i]);
            }
          }
        }
        return matArr;
      },
      //    向父组件传组员名称 和组长id   由于这是单独的一个穿梭框组件，数据会和父组件里面的动态绑定，提交按钮也在父组件，所以需要此步骤，
      async putParentsTeams() {
        this.$emit('teamNames', this.allTeam,this.leader);
      },
    },
 
  }
</script>
 
<style lang="scss" scoped>
  ul li {
    list-style: none;
  }
 
  .shuttle {
    display: flex;
    flex-direction: row;
    justify-content: center;
 
    .shuttle_item {
      width: 30%;
 
      span {
        font-size: 16px;
        margin-left: 135px;
      }
    }
 
    .shuttle_arrow {
      width: 10%;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      margin: 0 30px 0 20px;
 
      .go_left {
        margin-left: 0;
        margin-top: 15px;
      }
    }

 
    .school {
      border: 1px solid #c8c9cc;
      padding: 0;
      border-radius: 5px;
      height: 450px;
      overflow: auto;
 
      li {
        padding-top: 10px;
      }
    }
  }
</style>
