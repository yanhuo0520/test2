<template>
    <div class="openAccount">
        <el-form ref="ruleForm" :model="ruleForm" :rules="rules" label-width="100px">
            <div class="breadcrumb-con">
                <img class="left-icon" src="../../images/breadcrumb-left-icon.png" alt="">
                <div class="breadcrumb-info">
                    <el-breadcrumb separator-class="el-icon-arrow-right">
                        <el-breadcrumb-item>股金业务</el-breadcrumb-item>
                        <el-breadcrumb-item  class="breadcrumb-tit">股金开户</el-breadcrumb-item>
                    </el-breadcrumb>
                </div>
            </div>
            <div class="tit-con">
                <div class="shu"></div>
                <span class="tit">股金开户</span>
                <div class="bg"></div>
            </div>
            <div class="form-item-con clearfix">
                <el-form-item label="社员名称:" prop="name">
                    <el-input v-model="ruleForm.name" placeholder="请输入社员名称" clearable></el-input>
                </el-form-item>
                <el-form-item label="身份证号码:" prop="idcard">
                    <el-input v-model="ruleForm.idcard" placeholder="请输入身份证号码" clearable maxlength="18">
                        <template #append>
                            <span class="read-idCard" @click="findIdCard()">验证身份信息</span>
                        </template>
                    </el-input>
                </el-form-item>
                <el-form-item label="开户类型:">
                    <span>股金</span>
                </el-form-item>
                <template v-if="!pay_pwd && ruleForm.relation_id">
                   <el-form-item label="密码:" prop="pay_pwd" >
                      <el-input v-model="ruleForm.pay_pwd" placeholder="请输入您的密码" clearable type="password"></el-input>
                  </el-form-item>
                  <el-form-item label="确认密码:" prop="repassword" >
                      <el-input v-model="ruleForm.repassword" placeholder="请再次确认您的密码" clearable type="password"></el-input>
                  </el-form-item>
                </template>
                <template v-if="ruleForm.relation_id && pay_pwd">
                  <el-form-item label="密码:" prop="pay_pwd" >
                      <span style="color:#E6A23C">密码已设置,与您在其他合作社设置的密码一致</span>
                  </el-form-item>
                </template>
                <el-form-item label="开户日期:">
                    <el-date-picker type="date" readonly value-format="yyyy-MM-dd" placeholder="请选择开户日期" v-model="stock_date" style="width: 80%;"></el-date-picker>
                </el-form-item>
                <!-- <el-form-item label="操作员:" prop="operate">
                    <el-input v-model="ruleForm.operate" placeholder="请输入操作员" clearable></el-input>
                </el-form-item> -->
            </div>
            <div class="btn-con">
                <el-button type="primary" round @click="submitForm('ruleForm')" :loading="submitLoading">{{submitLoading ? '提交中...' : '立即提交'}}</el-button>
            </div>
        </el-form>
    </div>
</template>
<script>
export default {
  name: "openAccount",
  data() {
    return {
      ruleForm: {
        name: "",//社员名称
        idcard: "",//社员身份证号
        pay_pwd: "",//密码
        repassword: "",//确认密码
        // operate: "",
        relation_id: ""//社员关系id
      },
        stock_date: "",//开户日期
      rules: {
        name: [{ required: true, message: "请输入社员名称", trigger: "blur" }],
        idcard: [
          { required: true, message: "请输入身份证号码", trigger: "blur" }
        ],
        pay_pwd: [
          { required: true, message: "请输入您的密码", trigger: "blur" }
        ],
        repassword: [
          { required: true, message: "请再次确认您的密码", trigger: "blur" }
        ],
        // stock_date: [
        //   { required: true, message: "请选择开户日期", trigger: "blur" }
        // ],
        // operate: [{ required: true, message: "请输入操作员", trigger: "blur" }]
      },
      pay_pwd:'',
      submitLoading: false,

    };
  },
  methods: {
    submitForm(formName) {
      this.ruleForm.pay_pwd = this.ruleForm.pay_pwd.trim()
      this.ruleForm.repassword = this.ruleForm.repassword.trim()
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.submitLoading = true;
          if(this.ruleForm.relation_id == '' || this.ruleForm.relation_id == null){
            this.$message.error('请验证当前合作社内，社员正确的身份证号');
            this.submitLoading = false;
            return;
          }
          if(this.ruleForm.pay_pwd != this.ruleForm.repassword){
            this.$message.error('请确认您的密码是否相同');
            this.submitLoading = false;
            return;
          }
          let param = JSON.parse(JSON.stringify(this.ruleForm))
          if(this.ruleForm.pay_pwd != ''){
            param.pay_pwd = this.pay_pwd ? this.pay_pwd : this.utils.recursiveMD5(this.ruleForm.pay_pwd, 1)
            param.repassword = this.pay_pwd ? this.pay_pwd :  this.utils.recursiveMD5(this.ruleForm.repassword, 1)
          }
          this.ajax(
            "openStockAccount",
            param,
            "股金开户失败",
            res => {
              this.submitLoading = false;
              if (res.errno == 0) {
                this.$message.success("股金开户成功,请去开户列表查看");
                this.resetForm();
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
    },
    // 重置form表单
    resetForm() {
      this.$refs["ruleForm"].resetFields();
    },
    // 查找身份证
    findIdCard() {
      this.ajax(
        "searchIdcard",
        {
          idcard: this.ruleForm.idcard
        },
        "暂无该社员，请完善社员档案",
        res => {
          this.submitLoading = false;
          if (res.errno == 0) {
            this.$message.success("信息验证完成");
            this.ruleForm.relation_id = res.data.relation_id;
            let tmpPwd = res.data.pay_pwd ? res.data.pay_pwd : ''
            this.pay_pwd = tmpPwd
            this.ruleForm.pay_pwd = tmpPwd
            this.ruleForm.repassword = tmpPwd
            this.ruleForm.name = res.data.name ? res.data.name : ''
          } else {
            this.ruleForm.relation_id = null
            this.pay_pwd = ''
            this.ruleForm.pay_pwd = ''
            this.ruleForm.repassword = ''
            this.ruleForm.name = ''
          }
        },
        err => {
          this.submitLoading = false;
        }
      );
    }
  },
  activated(){
    this.$refs["ruleForm"].resetFields();
    this.pay_pwd = ''
    this.ruleForm.pay_pwd = ''
    this.ruleForm.repassword = ''
    this.ruleForm.relation_id = null
    this.ruleForm.name = ''
    let date = new Date();
    this.stock_date = new Date().toLocaleDateString().replace(/\//g,'-')
  }
};
</script>
<style lang="less">
.openAccount {
  padding: 20px;
  .tit-con {
    width: 100%;
    display: flex;
    align-items: center;
    position: relative;
    .shu {
      width: 0.28rem;
      height: 1rem;
      background-color: #3b6af1;
    }
    .tit {
      color: #444444;
      padding: 0 0.8rem;
      line-height: 1rem;
    }
    .bg {
      flex: 1;
      background: url("../../images/baseInfo/tit-bg.png");
      height: 1rem;
      background-size: 100% 100%;
    }
    .del-family-icon {
      position: absolute;
      right: 0;
      top: 0.5rem;
      height: 1.5rem;
      cursor: pointer;
    }
  }
  .form-item-con {
    margin: 2rem 0;
    position: relative;
    .el-form-item {
      width: 50%;
      //   float: left;
      .el-form-item__label {
        font-size: 0.9rem;
      }
      .el-input {
        width: 80%;
        .read-idCard {
          color: #3b6af1;
          background: #f0f8ff;
          font-size: 0.75rem;
          cursor: pointer;
        }
        .read-idCard:active {
          opacity: 0.6;
        }
      }
    }
    .upload-con {
      width: 30%;
      position: absolute;
      top: 50%;
      right: 0;
      transform: translateY(-50%);
      display: flex;
      align-items: center;
      justify-content: center;
      .el-form-item__content {
        margin-left: 0 !important;
      }
      .img-con {
        max-width: 100%;
        border: 1px solid #e4edf6;
        border-bottom: 0;
        padding: 0.5rem;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 10rem;
        height: 15rem;
        cursor: pointer;
        img {
          max-width: 100%;
          max-height: 100%;
        }
      }

      .upload-btn {
        width: 100%;
        padding: 0.3rem 0;
        background: #e4edf6;
        display: flex;
        align-items: center;
        justify-content: center;
        color: #3b6af1;
        font-size: 0.9rem;
        cursor: pointer;
      }
      .upload-btn:hover {
        opacity: 0.6;
      }
    }
  }
  .btn-con {
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 5rem;
    padding-bottom: 2rem;
    .el-button {
      width: 23%;
      padding: 0.9rem;
    }
  }
}
</style>