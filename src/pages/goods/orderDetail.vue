<template>
    <div class="order-detail" v-loading="loading">
        <div class="breadcrumb-con">
            <img class="left-icon" src="../../images/breadcrumb-left-icon.png" alt="">
            <div class="breadcrumb-info">
                <el-breadcrumb separator-class="el-icon-arrow-right">
                    <el-breadcrumb-item  :to="{ path: '/orderList' }">订单列表</el-breadcrumb-item>
                    <el-breadcrumb-item  class="breadcrumb-tit">订单详情</el-breadcrumb-item>
                </el-breadcrumb>
            </div>
            <el-button style="margin-right:1rem" size="small" @click="initData()">刷新</el-button>
            <div class="back-con" @click="goBack">
                <img class="back-icon" src="../../images/breadcrumb-back-icon.png" alt="">
                <span class="back-text">返回上一页</span>
            </div>
        </div>
          <el-form ref="ruleForm" :model="ruleForm"  label-width="100px" style="margin-top: 0.5rem;min-height:200px" v-if="ruleForm">
            <div class="tit-con">
                <div class="shu"></div>
                <span class="tit">商品信息</span>
                <div class="bg"></div>
            </div>
            <div class="form-item-con clearfix">
                 <el-table :data="ruleForm.order_goods" border >
                    <el-table-column prop="goods_id" label="ID" width="70px" align="center"></el-table-column>
                    <el-table-column prop="goods_name" label="商品名称"   show-overflow-tooltip align="center" ></el-table-column>
                    <el-table-column prop="spcifiName" label="多规格"   align="center"></el-table-column>
                    <el-table-column prop="goods_num" label="购买数量"  align="center"></el-table-column>
                    <el-table-column prop="retail_price" label="商品单价"  align="center"></el-table-column>
                </el-table>
            </div>
             <div class="tit-con">
                <div class="shu"></div>
                <span class="tit">订单信息</span>
                <div class="bg"></div>
            </div>
            <div class="form-item-con clearfix">
                <!-- <el-form-item label="所属合作社:">
                    <el-input v-model="ruleForm.coopera_name" readonly></el-input>
                </el-form-item> -->
                <el-form-item label="商户订单编号">
                    <el-input v-model="ruleForm.order_no" readonly></el-input>
                </el-form-item>
                 <el-form-item label="总价格:">
                    <el-input v-model="ruleForm.total_price" readonly></el-input>
                </el-form-item>
                 <el-form-item label="实际总价:">
                    <el-input v-model="ruleForm.real_price" readonly></el-input>
                </el-form-item>
                <el-form-item label="运费:">
                    <el-input :value="ruleForm.post_fee ? ruleForm.post_fee : '0'" readonly></el-input>
                </el-form-item>
                 <el-form-item label="商品数量:">
                    <el-input v-model="ruleForm.goods_num" readonly></el-input>
                </el-form-item>
                 <el-form-item label="下单时间:">
                    <el-input v-model="ruleForm.add_time" readonly></el-input>
                </el-form-item>
                <el-form-item label="订单状态:">
                    <el-input v-model="ruleForm.order_status_name" readonly></el-input>
                </el-form-item>
                <el-form-item label="支付方式:" v-if="ruleForm.pay_type">
                    <el-input  :value="ruleForm.pay_type_name" readonly></el-input>
                </el-form-item>
                 <el-form-item label="支付时间:" v-if="ruleForm.pay_time">
                    <el-input v-model="ruleForm.pay_time" readonly></el-input>
                </el-form-item>
            </div>
            <div class="tit-con">
                <div class="shu"></div>
                <span class="tit">收货人信息</span>
                <div class="bg"></div>
            </div>
            <div class="form-item-con clearfix">
                <el-form-item label="姓名:">
                    <el-input v-model="ruleForm.name" readonly></el-input>
                </el-form-item>
                <el-form-item label="手机号">
                    <el-input v-model="ruleForm.mobile" readonly></el-input>
                </el-form-item>
                 <el-form-item label="地址:">
                     <el-tooltip class="item" effect="dark" :content="ruleForm.zong_address" placement="top">
                        <el-input v-model="ruleForm.zong_address" readonly></el-input>
                    </el-tooltip>
                </el-form-item>
                 <el-form-item label="备注:">
                    <el-input v-model="ruleForm.note" readonly></el-input>
                </el-form-item>
            </div>
            <template v-if="ruleForm.seed">
                <div class="tit-con">
                    <div class="shu"></div>
                    <span class="tit">种子/农药备注信息</span>
                    <div class="bg"></div>
                </div>
                <div class="form-item-con seed-item-con clearfix">
                    <el-form-item class="seed-item" label="身份证:" v-if="ruleForm.seed.idcard_url">
                        <div class="idcard-img-con" @click="showImgView(ruleForm.seed.idcard_url)">
                            <img class="idcard-img" :src="ruleForm.seed.idcard_url" alt="">
                        </div> 
                    </el-form-item>
                    <el-form-item label="种植地点:">
                        <el-input v-model="ruleForm.seed.address" readonly></el-input>
                    </el-form-item>
                    <el-form-item label="电话:">
                        <el-input v-model="ruleForm.seed.phone" readonly></el-input>
                    </el-form-item>
                    <el-form-item label="身份证号:">
                        <el-input v-model="ruleForm.seed.idcard" readonly></el-input>
                    </el-form-item>
                </div>
            </template>
        </el-form>
        <div v-if="!ruleForm" style="height:300px"></div>
        <el-image-viewer v-if="showViewer"  :on-close="closeImgView"  :url-list="viewerImgs" />
    </div>
</template>
<script>
import ElImageViewer from 'element-ui/packages/image/src/image-viewer'
export default {
  name: "orderDetail",
  data() {
    return {
      orderId: '',
      ruleForm: '',

      showViewer: false,
      viewerImgs: [],

      loading: true,

      isRefresh: 1, // 返回上一页是否刷新 1-不刷新 2-刷新
      isAdmin: false, // 是否为管理员
      adminType: '', // 管理员类型
    };
  },
  components: { ElImageViewer },
  activated() {
    //判断是否为管理员
    let adminType = localStorage.getItem('is_admin')
    if(adminType && Number(adminType) >= 1) {
        this.isAdmin = true
        this.adminType = adminType
    }
    this.initData()
  },	
  methods: {
      // 初始化信息
    initData() {
        this.ruleForm = ''
        this.orderId = this.$route.query.id
        this.loading = true
        this.getOrderDetail()
    },
    // 返回上一页
    goBack() {
        this.$router.push({
            path: '/orderList',
            query:{
                isRefresh: this.isRefresh
            }
        })
    }, 
    getOrderDetail() {
      let that = this
      that.ajax('orderDetail',{
        order_id: that.orderId,
      },'获取订单详情失败',res =>{
        that.loading = false
        if(res.errno == 0) {
            that.ruleForm = res.data
        }
      }, err =>{
        that.loading = false
      })
    },
    // 显示图片大图
    showImgView(url) {
        if(url) {
            this.showViewer = true
            this.viewerImgs = [url]
        }
    },
    // 关闭图片大图
    closeImgView() {
        this.showViewer = false
        this.viewerImgs = []
    },
  }
};
</script>
<style lang="less">
.order-detail {
    padding: 20px;
    .tit-con {
        width: 100%;
        display: flex;
        align-items: center;
        position: relative;
        .shu {
            width: 0.28rem;
            height: 1rem;
            background-color: #3B6AF1;
        }
        .tit {
            color: #444444;
            padding: 0 0.8rem;
            line-height: 1rem;
        }
        .bg {
            flex: 1;
            background: url('../../images/baseInfo/tit-bg.png');
            height: 1rem;
            background-size: 100% 100%;
        }
        .del-family-icon {
            position:absolute;
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
            width: 35%;
            float: left;
            .el-form-item__label {
                font-size: 0.9rem;
            }
            .el-input {
                width: 100%;
            }
        }
    }
    .seed-item-con {
        .el-form-item {
            width: 35%;
             .el-input {
                width: 60%;
            }
        }
        .seed-item {
             width: 35%;
             height: auto;
            .idcard-img-con {
                width: 20rem;
                height: 12.6rem;
                border: 1px solid #ddd;
                padding: 10px;
                .idcard-img {
                    width: 100%;
                    height: 100%;
                }
            }
        }
        
    }
    .user-list-con {
        position: relative;
        border-bottom: 1px solid #dcdfe6; 
        margin-top: 1rem;
        .user-form-item-con {
            margin: 0;
            .el-form-item {
                width: 22%;
                float: left;
                .el-form-item__label {
                    font-size: 0.9rem;
                }
                .el-input {
                    width: 100%;
                }
            }
            .assets-form-item {
                width: 100%;
                margin-bottom: 0;
                .pic-item {
                    float: left;
                    width: 8rem;
                    height: 8rem;
                    padding: 0.5rem;
                    border: 1px solid #dcdfe6;
                    border-radius: 8px;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    margin-right: 1rem;
                    margin-bottom: 1rem;
                    .pic {
                        max-width: 100%;
                        max-height: 100%;
                    }
                }
            }
        }
    }
    .user-list-con:first-of-type {
        margin-top: 0;
    }
     .btn-con {
        display: flex;
        align-items: center;
        justify-content: center;
        padding-top: 5rem;
        padding-bottom: 2rem;
        .el-button {
            width: 15%;
            padding: 0.9rem;
        }
    }
}

</style>