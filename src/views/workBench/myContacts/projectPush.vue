<template>
  <!--项目推送人脉入口-->
  <div id="projectPush">
    <el-dialog :visible="dialogPush" :before-close="handleClose">

     <span slot="title" class="dialog-title clearfix">
        <div class="lines fl"></div>
        <div class="title fl">项目推送</div>
        <div class="lost fl">今日剩余推送<i>{{pushCount}}</i>次</div>
        <div class="img fl"><img src="../../../assets/images/why.png"></div>
      </span>

       <el-form label-position="top" :model="email2" label-width="80px" style="position: relative" ref="email2">
        <el-form-item label="推送人脉"
                      prop="nameEmail"
                      :rules="emailRule">
          <el-input v-model="email2.nameEmail" placeholder="项目及BP将以邮箱的方式推送，请输入邮箱"></el-input>
        </el-form-item>
      </el-form>
      <div class="message">
        <span class="message_innder fl" v-if="userMessage.user_real_name!=''">{{userMessage.user_real_name}}</span>
        <span class="message_innder fl" v-else>&#45;&#45;</span>
        <span class="lines fl"></span>
        <span class="message_innder fl" v-if="userMessage.user_company_career!=''">{{userMessage.user_company_career}}</span>
        <span class="message_innder fl" v-else>&#45;&#45;</span>
        <span class="lines fl"></span>
        <span class="message_innder fl" v-if="userMessage.user_company_name!=''">{{userMessage.user_company_name}}</span>
        <span class="message_innder fl" v-else>&#45;&#45;</span>
      </div>

            <el-form label-position="top" label-width="80px">
              <el-form-item label="推送项目">
                <el-select v-model="projectList" filterable
                           remote placeholder="请输入项目关键词"
                           multiple @remove-tag="removeTag"
                           :remote-method="remoteMethod"  popper-class="popper">
                  <el-option v-for="item in projectAll" :key="item.value" :label="item.label" :value="item.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-form>
      <div class="select">
              <el-tabs v-model="activeName">
                <el-tab-pane label="我的项目" name="first"></el-tab-pane>
              </el-tabs>
            </div>
            <div class="top-lists" style="background: #f3f4f8;cursor: pointer;margin-bottom: 29px;" >
              <el-table
                v-loading="loading"
                element-loading-text="拼命加载中"
                ref="multipleTable"
                :data="tableData3"
                tooltip-effect="dark"
                style="width: 100%"
                max-height="430"
                :row-class-name="tableRowClassName">
                <el-table-column
                  width="64">
                  <template scope="scope">
                    <el-radio class="radio" v-model="projectRadio" :label="scope.row.project_id"></el-radio>
                  </template>
                </el-table-column>
                <el-table-column
                  label="项目介绍"
                  min-width="570">
                  <template scope="scope">
                    <el-tooltip placement="top" :disabled="scope.row.pro_intro.length > 30 ? false:true">
                      <div slot="content">
                        <div class="tips-txt">{{scope.row.pro_intro}}</div>
                      </div>
                      <div>
                        {{scope.row.pro_intro}}
                      </div>
                    </el-tooltip>
                  </template>
                </el-table-column>
                <el-table-column
                  prop="match_weight"
                  label="匹配度"
                  min-width="80">
                  <template scope="scope">
                    <div class="origin">
                      {{scope.row.match_weight}}%
                    </div>
                  </template>
                </el-table-column>
              </el-table>
              <el-pagination
                class="pagination fr"
                small
                v-if="totalMatchProject!=0"
                @current-change="filterChangeMatchProject"
                :current-page.sync="currentPageMatchProject"
                layout="prev, pager, next"
                :page-size="10"
                :total="totalMatchProject">
              </el-pagination>
            </div>

            <el-form label-position="top" label-width="80px" ref="email" :model="email">
              <el-form-item label="标题" prop="title"
              :rules="[{max: 40, message: '长度不能大于40个字符', trigger: 'blur' }]">
                <el-input v-model="email.title" placeholder="请输入邮件标题"></el-input>
              </el-form-item>
              <el-form-item label="正文"
                            prop="body"
                            :rules="[{max: 500, message: '长度不能大于500个字符', trigger: 'blur' }]">
                <el-input type="textarea"
                          v-model="email.body"
                          placeholder="请输入邮件正文"
                          :autosize="{ minRows: 4, maxRows: 7}"></el-input>
              </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
              <el-button @click="preview">预览</el-button>
              <el-button type="primary" @click="push(2)">继续推送</el-button>
              <el-button type="primary" @click="push(1)">推送</el-button>
            </span>
    </el-dialog>

  </div>
</template>

<script type="text/ecmascript-6">

export default {
  props: ["dialogPush", "userMessage", "userEmail"],
  data () {
    var checkEmail = (rule, value, callback) => {
      if (this.$tool.getNull(value)) {
        return callback(new Error('邮箱不能为空'));
      }
      setTimeout(() => {
        if (!this.$tool.checkEmail(value)) {
          callback(new Error('请输入正确的邮箱'));
        } else {
          callback();
        }
      }, 300);


    };//邮箱判断
    var checkTitle = (rule, value, callback) => {
      if (this.$tool.getNull(value)) {
        return callback(new Error('不能为空'));
      } else {
        callback();
      }
    };//不为空判断

    return {
      emailRule: {validator: checkEmail, trigger: 'blur'},
      titleRule: {validator: checkTitle, trigger: 'blur'},
      close: false,//默认关闭
      loading: false,//加载动画
      activeName: 'first',
//      dialogPush:false,//控制显不显示

      email: {
        title: '',//邮件标题
        body: '',//邮件正文
      },
      email2: {
        nameEmail: '',//人脉的邮箱(一个)
      },
      user: {
        user_real_name: '',
        user_company_career: '',
        user_company_name: '',
        card_id: ""
      },//推送的用户消息
      project:{
        pro_id:"",//项目id
        pro_intro:"",//项目介绍
      },//推送的项目消息
      projectList: [],//推送的项目列表
      projectAll: [],//项目列表下拉框基本是不用的

      tableData3: [
        /*          {
        pro_intro: '项目的一句话介绍，字数可能会有点长，但不管怎样，就显示一行，如果显示不下那但不管怎样，就显示一行，如果显示不下那但不管怎样，就显示一行，如果显示不下那',
        match_weight : '100',
        project_id:1
      }*/
      ],
      projectRadio:'',//绑定当前项目数据,单选框的数据(project_id)
      firstInData:{
        user:{},
        email2:{},
        email:{}
      },//保存数据
      pushCount:0,//剩余推送次数
      totalMatchProject: 0,//项目加载总页数
      currentPageMatchProject:1,//当前第几页

    }
  },
  methods: {
    preview(){
      if(this.pushCount!=0) {

        if (this.projectRadio == '' || this.projectRadio==undefined) this.$tool.error("请选择要推送的项目")
        else if (!this.$tool.checkEmail(this.email2.nameEmail)) this.$tool.error("请输入正确的邮箱")
        else if(this.email.title.length>40) this.$tool.error("标题不能大于40个字")
        else if(this.email.body.length>500) this.$tool.error("正文不能大于500个字")
        else {
          this.$store.state.pushProject.project_id = this.projectRadio;
          this.$store.state.pushProject.user = this.user;
          this.$store.state.pushProject.pushMessage.user_id = localStorage.user_id;
          this.$store.state.pushProject.pushMessage.card_id = this.user.card_id;
          this.$store.state.pushProject.pushMessage.email = this.email2.nameEmail;
          this.$store.state.pushProject.pushMessage.title = this.email.title;
          this.$store.state.pushProject.pushMessage.body = this.email.body;
          this.$store.state.pushProject.pushMessage.project_ids = new Array;
          this.$store.state.pushProject.pushMessage.project_ids.push(this.projectRadio);
          this.$store.state.pushProject.email.title = this.email.title;
          this.$store.state.pushProject.email.body = this.email.body;
          this.$emit('changeall', false);
        }
      }else{
        this.$tool.warning("您今日的推送次数已用完")
      }
    },//推送预览
    push(type){
      if(this.pushCount!=0){
        this.firstInData.email=this.email;
        let check1 = this.submitForm('email');
        let check2 = this.submitForm('email2');
        if(this.projectRadio=='' || this.projectRadio==undefined) this.$tool.error("请选择要推送的项目")
        else if(!this.$tool.checkEmail(this.email2.nameEmail)) this.$tool.error("请输入正确的邮箱")
        else if(this.email.title.length>40) this.$tool.error("标题不能大于40个字")
        else if(this.email.body.length>500) this.$tool.error("正文不能大于500个字")
        else if(type ==1){ //关闭
          if(check1 && check2) {
            console.log("推送啦1")
            let pushData=new Object;
            pushData.user_id= localStorage.user_id;
            pushData.card_id=this.user.card_id;
            pushData.email=this.email2.nameEmail;
            pushData.title=this.email.title;
            pushData.body=this.email.body;
            pushData.project_ids=new Array;
            pushData.project_ids.push(this.projectRadio);
            this.$http.post(this.URL.pushUser, pushData)
              .then(res => {
                let data=res.data.data;
//              this.$tool.console(res);
//              this.$emit('changeall',false);
                this.$tool.success("推送成功");
                this.$emit('changeCloseProjectpush',false);
                this.getpushCount();
              })
              .catch(err => {
                this.$tool.console(err);
                this.$tool.success("推送失败");
              })
          }
        }else if(type==2){ //继续
          let pushData=new Object;
          pushData.user_id= localStorage.user_id;
          pushData.card_id=this.user.card_id;
          pushData.email=this.email2.nameEmail;
          pushData.title=this.email.title;
          pushData.body=this.email.body;
          pushData.project_ids=new Array;
          pushData.project_ids.push(this.projectRadio);
          this.$http.post(this.URL.pushUser, pushData)
            .then(res => {
              let data=res.data.data;
//            this.$tool.console(res);
              this.user={};
              this.email2.nameEmai="";
              this.projectList=[];
              this.tableData3 =[];
              this.email=this.firstInData.email;
              this.user=this.firstInData.user;
              this.email2=this.firstInData.email2;
              this.$tool.success("推送成功");
              this.remoteMethod("");
              this.getpushCount();
            })
            .catch(err => {
              this.$tool.console(err);
              this.$tool.success("推送失败");
            })
        }
      }else{
          this.$tool.warning("您今日的推送次数已用完")
      }
    },//推送按钮1推送1次,2继续推送
    remoteMethod(query) {
      if(query=="") this.projectRadio="";
      this.loading=true;
      this.currentPageMatchProject=1;
      this.$http.post(this.URL.matchProject,{
        user_id: localStorage.user_id,
        card_id: this.user.card_id,
        pro_intro: query,
        page:1})
        .then(res=>{
          let data = res.data.data;
//          this.$tool.console(data);
          this.tableData3=data.projects;
          this.projectAll=this.setProjectAll(data.projects);
          this.totalMatchProject=data.count;
          this.loading=false;
        })
        .catch(err =>{
          this.$tool.console(err,2);
          this.loading=false;
        })
    },//项目搜索
    filterChangeMatchProject(page){
      this.loading=true;
      this.$http.post(this.URL.matchProject,{
        user_id: localStorage.user_id,
        card_id: this.user.card_id,
        pro_intro: "",
        page:page})
        .then(res=>{
          let data = res.data.data;
//          this.$tool.console(data);
          this.tableData3=data.projects;
          this.projectAll=this.setProjectAll(data.projects);
          this.loading=false;
          this.totalMatchProject=data.count;
        })
        .catch(err =>{
          this.$tool.console(err,2);
          this.loading=false;
        })
    },//页码控制
    removeTag(e){
      this.projectRadio='';
    },//删除标签
    setProjectAll(data){
      let arr = [];
      data.forEach((x)=>{
        let obj = {};
        obj.label = x.pro_intro;
        obj.value = x.project_id;
        arr.push(obj);
      });
      return arr
    },//设置项目下拉框,虽然没什么卵用
    tableRowClassName(row, index) {
      if (index%2 === 1) {
        return 'info-row';
      }
      return '';
    },//控制列表颜色
    getIntroduce(id){
      if(id!=""){
        this.projectList=[];
        this.projectList.push(id);
        let arr = this.tableData3;
        for(let i=0; i<arr.length; i++){
          if(arr[i].project_id==id){
            this.$store.state.pushProject.pro_intro=arr[i].pro_intro;

          }
        }
      }
    },//获取项目详情
    submitForm(formName) {
      let check = true;
      this.$refs[formName].validate((valid) => {
        if (valid) {
          return
        } else {
          check = false;
        }
      });
      return check;
    },
    handleClose(){
      this.$emit('changeCloseProjectpush',false);
    },
    getpushCount(){
      this.$http.post(this.URL.pushCount,{
        user_id: localStorage.user_id})
        .then(res=>{
          let data = res.data.data;
          this.pushCount=data.push_count.remain_times;
//          this.$tool.console(data.push_count);
        })
        .catch(err =>{
          this.$tool.console(err,2);
          this.loading=false;
        })
    },//获取剩余推送次数
  },
  watch : {
    projectRadio : function(e){
      this.getIntroduce(e);
    },
    dialogPush : function (e) {
        if(e){
          this.user={};
          this.email2.nameEmai="";
          this.projectList=[];
          this.tableData3 =[];
          this.projectRadio="";
          this.user =this.userMessage || this.$store.state.pushProject.user;
          this.project = this.$store.state.pushProject.projectMessgae || {};
          this.email2.nameEmail =this.userEmail;
          this.firstInData.user =this.userMessage;
          this.firstInData.email2.nameEmail = this.userEmail;
          this.firstInData.project = this.$store.state.pushProject.projectMessgae || {};
          /*      this.email.title=this.$store.state.pushProject.email.title || '';
           this.email.body=this.$store.state.pushProject.email.body || '';
           this.projectRadio=this.$store.state.pushProject.project_id || '';*/
          if(this.firstInData.project.pro_id!=""){
            this.remoteMethod(this.firstInData.project.pro_intro);
            setTimeout(()=>{this.projectRadio=this.firstInData.project.pro_id;},200)

          }else{
            this.remoteMethod("");
          }
        }
        this.getpushCount();
    },

  },
  created(){

  }
}
</script>

<style lang="less">
  @import '../../../assets/css/mycontacts';
.popper{
display: none;
}
</style>
