<template>
    <div class="accountList"  v-loading="loading">
      <div class="breadcrumb-con">
            <img class="left-icon" src="../../images/breadcrumb-left-icon.png" alt="">
            <div class="breadcrumb-info">
                <el-breadcrumb separator-class="el-icon-arrow-right">
                    <el-breadcrumb-item>互助金业务</el-breadcrumb-item>
                    <el-breadcrumb-item  class="breadcrumb-tit">开户列表</el-breadcrumb-item>
                </el-breadcrumb>
            </div>
          <el-button size="small" @click="initData()">刷新</el-button>
        </div>
        <div class="operate-con">
              <div class="operate-con clearfix">
                <div class="search-con">
                    <el-form class="clearfix"  label-width="65px" >
                        <el-form-item label="开户日期" style="width:auto">
                            <el-date-picker v-model="dateArr" type="daterange" :picker-options="pickerOptions" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" align="right"> </el-date-picker>
                        </el-form-item>
                        <el-button type="primary" size="small" :loading="loading" @click="search()">{{loading ? '获取数据中...' : '查询'}}</el-button>
                    </el-form>
                </div>
            </div>
          </div>
           <div class="btns-manage">
            <el-button size="mini" type="success" round  @click="getData" v-if="adminType == 3">管理我的合作社</el-button>
            <el-button size="mini" type="warning"  round @click="chooseCoopera" v-if="adminType != 0">查看下级合作社</el-button>
          </div>
          <div class="tip" :class="{'manage-tip':!selectName}" v-if="adminType != 0">
            <span v-if="selectName">正在查看下级【{{selectName}}】的数据</span>
            <span v-else>正在管理我的合作社</span>
          </div>
        <div class="table-con">
          <div>
              <el-table  :data="tableData" v-if="tableData && tableData.length > 0" border>
                  <el-table-column prop="relation_id" label="ID" width="80" align="center"></el-table-column>
                  <el-table-column prop="coopera_name" label="开户机构" align="center"></el-table-column>
                  <el-table-column prop="passbook_date" label="开户日期" align="center"></el-table-column>
                  <el-table-column prop="name" label="社员名称" align="center"></el-table-column>
                  <el-table-column prop="card" label="卡号" align="center"></el-table-column>
                  <el-table-column prop="idcard" label="身份证号" align="center"></el-table-column>
                  <el-table-column prop="passbook_status" label="卡状态" align="center"></el-table-column>
                  <template v-if="!selectName && (isGuashi || isBuka || isPrint || isDel)">
                      <el-table-column  label="操作" align="center" >
                        <template slot-scope="scope" >
                          <el-button type="text" size="small" @click="report(scope.row.card,scope.row.coopera_id,scope.row.passbook_status)" v-if="isGuashi">{{scope.row.passbook_status == '正常'?'挂失':'解挂'}}</el-button>
                          <el-button type="text" size="small" v-if="scope.row.passbook_status == '挂失' && isBuka" @click="updateCard(scope.row.relation_id,scope.row.passbook_num,scope.row.passbook_num_type)">补卡</el-button>
                          <el-button type="text" size="small" v-if="isPrint" @click="doPrint(scope.row)">打印存折</el-button>
                          <el-button type="text" size="small" @click="del(scope.row.card,scope.row.coopera_id)" v-if="isDel">删除</el-button>
                        </template>
                    </el-table-column>
                  </template>
              </el-table>
              <el-pagination
                  background
                  :current-page="curPage"
                  layout="prev, pager, next"
                  :total="totalPage*10"
                  @current-change="handleCurPageChange">
              </el-pagination>
          </div>
          <div v-if="!tableData || tableData.length == 0">
            <div class="no-data-con" >
              <div class="absolute-center">
                <div class="err-info-text ">暂无社员开户</div>
              </div>
            </div>
          </div>
        </div>
            
        <el-dialog title="补卡" :visible.sync="isUpdate">
          <el-form :model="ruleForm" :rules="rules" ref="ruleForm">
            <el-form-item label="卡号" :label-width="formLabelWidth" prop="card" v-if="ruleForm.passbook_num_type == 1">
              <el-input autocomplete="off" placeholder="请输入卡号" v-model="ruleForm.card">
                <!-- <template #append>
                    <span class="read-idCard" @click="readBankNo()">读卡号</span>
                </template> -->
              </el-input>
            </el-form-item>
            <el-form-item label="芯片标识:" :label-width="formLabelWidth" prop="uuid" v-if="ruleForm.passbook_num_type == 1">
                <el-input type="password" v-model="ruleForm.uuid" placeholder="请刷卡获取芯片标识" readonly>
                    <template #append>
                        <span class="read-idCard" @click="readBankNo()">刷卡</span>
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item label="存折号" :label-width="formLabelWidth" prop="card" v-if="ruleForm.passbook_num_type == 2">
              <el-input autocomplete="off" placeholder="请输入存折号" v-model="ruleForm.card"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="isUpdate = false">取 消</el-button>
            <el-button type="primary" @click="submitCard('ruleForm')" :loading="submitLoading">{{submitLoading ? '提交中...' : '确 定'}}</el-button>
          </div>
        </el-dialog>

        <div class="iframe-absolute-con" v-if="showReadIc" @click="showReadIc = false" style="z-index:9999">
            <iframe @click.stop=""  class="ic-iframe" id="readIcIframeId"  frameborder="0"></iframe>
        </div>

        <my-drawer :drawer="drawer" :cooperativeList="cooperativeList" :level2List="level2List" ></my-drawer>
    </div>
</template>
<script>
import myDrawer from "../../components/drawer.vue";
export default {
  name: "accountList",
  data() {
    return {
      tableData: [],
      curPage: 1,
      totalPage: 1,
      loading: false,
      tdWidth: 280,

      dateArr: '',
      pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }]
		},


      isUpdate: false,
      ruleForm: {
        card: "",
        relation_id: "",
        passbook_num: "",
        passbook_num_type:'',
        uuid:''// IC卡芯片卡号
      },
      rules: {
        card: [{ required: true, message: "请输入卡号", trigger: "blur" }],
        uuid: [{ required: true, message: "请刷卡获取芯片标识", trigger: "blur" }]
      },
      submitLoading: false,
      formLabelWidth: "220px",

      isGuashi: false,
      isBuka: false,
      isPrint: false,
      isDel: false,

      isAdmin: false, // 是否为管理员
      adminType: '', // 管理员类型
      cooperaId: "", //合作社id
      cooperativeList: [],

      showReadIc: false, // 读取卡号弹窗

      adminLoading: false,
      cooperationInfo: '',

      drawer: false,
      selectName: '',
      level2List: [],
    };
  },
  components: { myDrawer },
  created() {
    window.addEventListener("message",(event)=>{
         if(event.data && event.data.path && event.data.path == 'accountListReadIc') {
            if(event.data.type == 'data') {
                this.ruleForm.uuid = event.data.data
                // this.getBankNo()
                this.showReadIc = false	
            }
            if(event.data.type == 'close') {
                this.showReadIc = false	
                this.ruleForm.uuid = ''
            }
        }
    }, false)
  },
  activated() {
    this.initEventWatch()
	  this.selectName = '';
    this.level2List = []
    this.tableData = [];
    this.curPage = 1;
    this.loading = true;
    //  const start = new Date();
	  // const end = new Date();
	  // start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
    // this.dateArr = [start, end]
    this.dateArr = ''
    
    //判断是否为管理员
    let adminType = localStorage.getItem('is_admin')
    if(adminType && Number(adminType) >= 1) {
      this.isAdmin = true
      this.adminType = adminType
    }
    if (this.isAdmin) {
      this.cooperationInfo = localStorage.getItem('cooperationInfo') ? JSON.parse(localStorage.getItem('cooperationInfo')) : ''
      this.utils.getCooperativeList(this, data =>{
          if(data && data.length > 1 && adminType == 3) {
            this.cooperaId = data[1].coopera_id
          } else {
            this.cooperaId = ''
            this.selectName = data[0].coopera_name
            data[0].isSelect = true
          }
          this.cooperativeList = data
          this.getStaff()
        },err =>{
            this.loading = false
        })
    } else {
      this.getStaff();
    }
    this.handlePermission();
  },
  methods: {
    initData() {
      this.selectName = '';
      this.level2List = []
      this.tableData = [];
      this.curPage = 1;
      this.loading = true;
      this.cooperaId = "";
      //  const start = new Date();
      // const end = new Date();
      // start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
      // this.dateArr = [start, end]
      this.dateArr = ''

      if(this.cooperativeList && this.cooperativeList.length > 1 && this.adminType == 3) {
          this.cooperaId = this.cooperativeList[1].coopera_id
      } else {
          this.cooperaId = ''
          if(this.cooperativeList && this.cooperativeList.length > 0) {
            this.selectName = this.cooperativeList[0].coopera_name
          }
      }
      this.getStaff();
    },
     // 初始化 监听合作社选择组件事件
    initEventWatch(){
      eventWatch.$on('closeDrawer', res =>{
        this.drawer = false
      })
      eventWatch.$on('selectLevel1', row=>{
        this.cooperativeList.forEach(item =>{
          this.$set(item,'isSelect', false)
        })
        this.$set(row,'isSelect', true)
        this.level2List = []
        if(row.coopera_id == row.parent_id && row.count > 1) {
          this.getLevel2Data(row)
        } else {
          this.tableData = []
          this.curPage = 1
          this.totalPage = null
          this.cooperaId = row.coopera_id
          this.selectName = row.coopera_name
          this.getStaff()
        }
      })
      eventWatch.$on('selectLevel2', row=>{
        this.level2List.forEach(item =>{
          this.$set(item,'isSelect', false)
        })
        this.$set(row,'isSelect', true)

        this.tableData = []
        this.curPage = 1
        this.totalPage = null
        this.cooperaId = row.coopera_id
        this.selectName = row.coopera_name
        this.getStaff()
      })
    },
    // 获取我的合作社数据
      getData() {
      if(!this.selectName) return
          let cooperaInfo = localStorage.getItem('cooperationInfo') ? JSON.parse(localStorage.getItem('cooperationInfo')) : ''
          this.tableData = []
          this.curPage = 1
          this.totalPage = null
          this.cooperaId = cooperaInfo ? cooperaInfo.id : ''
          this.selectName = ''
          this.level2List = []
          this.getStaff()
      },
      // 展开选择合作社
      chooseCoopera() {
        this.drawer = true
        eventWatch.$emit('initData')
      },
      getLevel2Data(row,callBack){
        this.ajax("cooperativeLevelList",{ parent_id: row.coopera_id },"获取合作社列表失败",res => {
          if(res.errno == 0) {
            res.data.forEach(item  =>{
              item.disabled = true
            })
            this.level2List =res.data
            callBack && typeof callBack == 'function' && callBack()
          } else {
            this.level2List = []
          }
          },err => {
            this.level2List = []
          }
        );
      },
    search() {
      this.tableData = [];
      this.curPage = 1;
      this.getStaff();
    },
    // 判断当前页面都有什么权限
    handlePermission() {
      let that = this;
      that.utils.getPermissionList(that, data => {
        data.forEach(item => {
          if (item.title == "挂失") {
            that.isGuashi = true;
          }
          if (item.title == "补卡") {
            that.isBuka = true;
          }
          if (item.title == "打印存折") {
            that.isPrint = true;
          }
          if (item.title == "删除") {
            that.isDel = true;
          }
        });
        if(data.length === 4) {
          if(!that.isPrint) {
            that.tdWidth = 220
          } else {
            that.tdWidth = 240
          }
        } else if(data.length === 3) {
          if(!that.isPrint) {
            that.tdWidth = 180
          } else {
            that.tdWidth = 200
          }
        } else if(data.length === 2) {
          if(!that.isPrint) {
            that.tdWidth = 140
          } else {
            that.tdWidth = 160
          }
        } else if(data.length === 1) {
            that.tdWidth = 0
        }

      });
    },
     // 读卡号
    readBankNo() {
      this.showReadIc = true
      // this.ruleForm.card = ''
      setTimeout(() =>{
          document.getElementById('readIcIframeId').src='./static/autoGetCardSN.html?path=accountListReadIc';
      },100)
    },
    // 根据IC芯片卡号获取卡号
    getBankNo() {
      let that = this;
      that.ajax('searchICCard',{
        uuid:that.ruleForm.uuid
      },'获取卡号失败',res => {
          if (res.errno == 0) {
            that.ruleForm.card  = res.data.card;
            // that.findcard()
          }
        });
    },
    //获取存折号
    getCard() {
      this.ajax("getCard",{},"存折号生成失败",res => {
          if (res.errno == 0) {
            this.ruleForm.card = res.data
          }
        },err => {}
      );
    },
    //   社员列表
    getStaff() {
      this.loading = true
      this.ajax(
        "userRelationLists",
        {
          page: this.curPage,
          size: 10,
          coopera_id: this.cooperaId,
          start:this.dateArr ? (this.utils.dateFormat('yyyy-MM-dd', this.dateArr[0]) + ' 00:00:00') : '',
          end: this.dateArr ? (this.utils.dateFormat('yyyy-MM-dd', this.dateArr[1])+' 23:59:59') : ''
        },
        "获取社员列表失败",
        res => {
          this.loading = false;
          if (res.errno == 0) {
            this.drawer = false
            this.tableData = res.data.data;
            this.curPage = Number(res.data.current_page);
            this.totalPage = res.data.total;
            this.tableData.forEach(ele => {
              if (ele.passbook_date) {
                ele.passbook_date = ele.passbook_date.substring(0, 10);
              }
              if (ele.passbook_status == 0) {
                ele.passbook_status = "挂失";
              } else if (ele.passbook_status == 1) {
                ele.passbook_status = "正常";
              } else {
                ele.passbook_status = "正常";
              }
            });
          }
        },
        err => {
          this.loading = false;
        }
      );
    },
    // 切换分页
    handleCurPageChange(val) {
      this.curPage = val;
      this.getStaff();
    },
    // 删除
    del(card, coopera_id) {
      this.ajax(
        "searchCard",
        {
          card: card
        },
        "",
        res => {
          if (res.errno == 0) {
            if (
              Number(res.data.money) == 0 &&
              Number(res.data.regular_money) == 0 &&
              Number(res.data.stock_money) == 0 &&
              Number(res.data.loan) == 0
            ) {
              this.$confirm("此操作将删除社员, 是否继续?", "提示", {
                confirmButtonText: "确定",
                cancelButtonText: "取消",
                type: "warning"
              })
                .then(() => {
                  this.ajax(
                    "updateCardStatus",
                    {
                      card: card,
                      coopera_id: coopera_id,
                      passbook_status: 2
                    },
                    "删除失败",
                    res => {
                      if (res.errno == 0) {
                        this.$message("删除成功");
                        this.getStaff();
                      }
                    }
                  );
                })
                .catch(() => {});
            } else {
              this.$message.error("该卡余额不为0，不能删除！");
            }
          }
        },
        err => {}
      );
    },
    // 挂失
    report(card, coopera_id, status) {
      let sta = "";
      let msg = "";
      if (status == "正常") {
        sta = 0;
        msg = "挂失失败";
      } else {
        sta = 1;
        msg = "解挂失败";
      }
      this.ajax(
        "updateCardStatus",
        {
          card: card,
          coopera_id: coopera_id,
          passbook_status: sta
        },
        msg,
        res => {
          if (res.errno == 0) {
            if (status == "正常") {
              this.$message.success("挂失成功");
            } else {
              this.$message.success("解挂成功");
            }
            this.getStaff();
          }
        }
      );
    },

    // 打印存折
    doPrint(row) {
      let info = {};
      info.name = encodeURI(encodeURI(row.name));
      info.card = row.card;
      info.passbook_date = row.passbook_date;
      info.coopera_name = encodeURI(encodeURI(row.coopera_name));
      window.open(
        "../../../static/cunzhe_kaihu.html?info=" + JSON.stringify(info)
      );
    },

    // 补卡
    updateCard(relation_id, passbook_num, passbook_num_type) {
      this.isUpdate = true;
      this.ruleForm.relation_id = relation_id;
      this.ruleForm.passbook_num = passbook_num;
      this.ruleForm.passbook_num_type = passbook_num_type;
      if(passbook_num_type == 2){
        this.getCard()
      }
    },
    submitCard(formName) {
      this.ruleForm.card = this.ruleForm.card.trim()
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.submitLoading = true;
          this.ajax(
            "updateCard",
            this.ruleForm,
            "补卡失败",
            res => {
              this.submitLoading = false;
              if (res.errno == 0) {
                this.$message.success("补卡成功");
                this.isUpdate = false;
                this.$refs["ruleForm"].resetFields();
                this.getStaff();
              }
            },
            err => {
              this.submitLoading = false;
            }
          );
        } else {
          setTimeout(() => {
            let isError = document.getElementsByClassName("is-error");
            let firstErrInput = isError[0].querySelector("input");
            if (firstErrInput.type == "file") {
              document.querySelectorAll(
                ".el-scrollbar__wrap"
              )[1].scrollTop = 50;
            } else {
              firstErrInput.focus();
            }
          }, 100);
          return false;
        }
      });
    }
  }
};
</script>
<style lang="less">
.accountList {
  padding: 20px;
  .operate-con {
    padding: 10px 0;
    .search-con {
      float: left;
      width: 80%;
      .el-form-item {
        width: 28%;
        float: left;
        margin-bottom: 0;
        .el-form-item__label {
          font-size: 0.9rem;
        }
        .el-form-item__content {
          margin-right: 30px;
          .el-input {
            width: 100%;
          }
        }
      }
    }
    .btn-con {
      float: right;
    }
  }

  .read-idCard {
    color: #3b6af1;
    background: #f0f8ff;
    font-size: 0.75rem;
    cursor: pointer;
  }
  .el-table {
    background: transparent;
  }
  .el-table::before {
    height: 0;
  }
  .el-pagination {
    float: right;
    margin-top: 30px;
    margin-bottom: 30px;
  }

  .el-dialog {
    .el-input {
      width: 50%;
    }
  }
}
</style>