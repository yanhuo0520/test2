<template>
    <div class="order-list"  v-loading="loading">
		<div class="breadcrumb-con">
			<img class="left-icon" src="../../../images/breadcrumb-left-icon.png" alt="">
			<div class="breadcrumb-info">
				<el-breadcrumb separator-class="el-icon-arrow-right">
					<el-breadcrumb-item>合作商城</el-breadcrumb-item>
					<el-breadcrumb-item  class="breadcrumb-tit">订单列表</el-breadcrumb-item>
				</el-breadcrumb>
			</div>
			<el-button size="small" @click="initData()">刷新</el-button>
		</div>
		<div class="operate-con clearfix">
			<div class="search-con" >
				<el-form class="clearfix"  label-width="65px">
					<el-form-item label="订单状态">
						<el-select v-model="orderType" filterable placeholder="请选择订单状态">
							<el-option v-for="(item,index) in orderTypeOptions" :key="index" :label="item.label" :value="item.value"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="支付方式">
						<el-select v-model="payType" filterable placeholder="请选择支付方式">
							<el-option v-for="(item,index) in payTypeOptions" :key="index" :label="item.label" :value="item.value"></el-option>
						</el-select>
					</el-form-item>
					<el-form-item label="订单日期">
						<el-date-picker v-model="dateArr" type="daterange" :picker-options="pickerOptions" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" align="right"> </el-date-picker>
					</el-form-item>
					<el-button type="primary" size="small" :loading="loading" @click="search()">{{loading ? '获取数据中...' : '查询'}}</el-button>
				</el-form>
			</div>
		</div>
		<div class="table-con">
			<template v-if="data && data.length > 0">
				<el-table :data="data" border >
					<el-table-column prop="order_id" label="ID" width="70px" align="center"></el-table-column>
					<el-table-column prop="order_no" label="订单编号"   show-overflow-tooltip align="center" ></el-table-column>
					<!-- <el-table-column prop="coopera_name" label="所属合作社"  width="200" show-overflow-tooltip align="center" ></el-table-column> -->
					<el-table-column prop="contents" label="订单状态"  width="160" align="center">
						<template slot-scope="scope">
							<el-tag v-if="scope.row.order_status == 1" type="info" effect="dark">未付款</el-tag>
							<el-tag v-if="scope.row.order_status == 2" type="warning" effect="dark">待发货</el-tag>
							<el-tag v-if="scope.row.order_status == 3" type="danger" effect="dark">支付失败</el-tag>
							<el-tag v-if="scope.row.order_status == 4" type="warning" effect="dark">待支付</el-tag>
							<el-tag v-if="scope.row.order_status == 5" type="info" effect="dark">已取消</el-tag>
							<el-tag v-if="scope.row.order_status == 6" type="info" effect="dark">已退款</el-tag>
							<el-tag v-if="scope.row.order_status == 7" type="" effect="dark">待收货</el-tag>
							<el-tag v-if="scope.row.order_status == 8" type="warning" effect="dark">退换货</el-tag>
							<el-tag v-if="scope.row.order_status == 9" type="info" effect="dark">待评价</el-tag>
							<el-tag v-if="scope.row.order_status == 10" type="success" effect="dark">已完成</el-tag>

							<el-tag v-if="scope.row.order_status == 11" type="info" effect="dark">单个商品退换货</el-tag>
							<el-tag v-if="scope.row.order_status == 12" type="info" effect="dark">单个商品已退款</el-tag>
							<el-tag v-if="scope.row.order_status == 13" type="info" effect="dark">订单超时</el-tag>
						</template>
					</el-table-column>
					<el-table-column prop="real_price" label="实付金额" width="100" align="center"></el-table-column>
					<el-table-column label="支付方式" width="190" align="center" >
						<template slot-scope="scope">
							<el-tag v-if="scope.row.pay_type == 1" type="" effect="dark">支付宝扫码</el-tag>
							<el-tag v-if="scope.row.pay_type == 2" type="" effect="dark">微信扫码</el-tag>
							<el-tag v-if="scope.row.pay_type == 3" type="" effect="dark">现金支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 4" type="" effect="dark">建行聚合支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 5" type="" effect="dark">记账支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 6" type="" effect="dark">其他支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 7" type=""  effect="dark">建行二维码</el-tag>
							<el-tag v-if="scope.row.pay_type == 8" type="" effect="dark">pos支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 9" type="" effect="dark">银联公众号支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 10" type="" effect="dark">银联二维码</el-tag>
							<el-tag v-if="scope.row.pay_type == 11" type="" effect="dark">银联btoc支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 12" type="" effect="dark">微信二维码支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 13" type="" effect="dark">支付宝二维码支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 20" type="" effect="dark">微信app支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 21" type="" effect="dark">支付宝app支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 22" type="" effect="dark">微信app银联渠道支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 23" type="" effect="dark">支付宝app银联渠道支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 24" type="" effect="dark">云闪付支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 25" type="" effect="dark">农行微信分账支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 26" type="" effect="dark">农行支付宝分账支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 27" type="" effect="dark">农行app支付</el-tag>
							<el-tag v-if="scope.row.pay_type == 28" type="" effect="dark">纯余额支付</el-tag>
						</template>
					</el-table-column>	
					<el-table-column prop="goods_num" label="商品数量" width="80" align="center"></el-table-column>
					<el-table-column prop="name" label="收货人姓名" width="100" show-overflow-tooltip align="center"></el-table-column>
					<el-table-column prop="mobile" label="收货人电话" width="120" show-overflow-tooltip align="center"></el-table-column>
					<el-table-column prop="zong_address" label="收货地址"  show-overflow-tooltip align="center"></el-table-column>
					<el-table-column prop="add_time" label="下单时间" width="150" show-overflow-tooltip align="center"></el-table-column>
					<el-table-column label="操作" width="240" fixed="right" align="center">
						<template slot-scope="scope">
							<el-button size="small" plain type="" @click="detail(scope.row)" >查看</el-button>
							<el-button v-if="scope.row.order_status == 2 && scope.row.is_send == 2 && isSend" size="small" plain type="primary" @click="sendGoods(scope.row,scope.$index)" :loading="sendLoading">发货</el-button>
							<el-button  v-if="scope.row.order_status == 8 && isRefund"  size="small" plain type="warning" @click="orderRefund(scope.row,scope.$index)" :loading="orderRefundLoading">退换货</el-button>
						</template>
					</el-table-column>
				</el-table>
			</template>
			<template v-if="!data || data.length == 0">
				<div class="no-data-con" >
					<div class="absolute-center">
						<div class="err-info-text ">暂无订单列表</div>
					</div>
				</div>
			</template>
			<el-pagination @current-change="handleCurPageChange" v-if="totalPage && totalPage > 0" background :current-page="curPage" layout="prev, pager, next" :total="totalPage*10"> </el-pagination>
		</div>
    </div>
</template>
<script>
export default {
  name: "orderList",
  data() {
    return {
      data: [],
	  curPage: 1,
	  totalPage: null,
	  dateArr: '',
	  orderType: '',
	  orderTypeOptions:[{
		  label: '全部',
		  value: '',
	  },{
		  label: '未付款',
		  value: 1,
	  },{
		  label: '待发货',
		  value: 2,
	  },{
		  label: '支付失败',
		  value: 3,
	  },{
		  label: '待支付',
		  value: 4,
	  },{
		  label: '已取消',
		  value: 5,
	  },{
		  label: '已退款',
		  value: 6,
	  },{
		  label: '待收货',
		  value: 7,
	  },{
		  label: '退换货',
		  value: 8,
	  },{
		  label: '待评价',
		  value: 9,
	  },{
		  label: '已完成',
		  value: 10,
	  },{
		  label: '单个商品退换货',
		  value: 11,
	  },{
		  label: '单个商品已退款',
		  value: 12,
	  },{
		  label: '订单超时',
		  value: 13,
	  }],
	  payType: '',
	  payTypeOptions:[{
		  label: '全部',
		  value: '',
	  },{
		  label: '支付宝扫码',
		  value: 1,
	  },{
		  label: '微信扫码',
		  value: 2,
	  },{
		  label: '现金支付',
		  value: 3,
	  },{
		  label: '建行聚合支付',
		  value: 4,
	  },{
		  label: '记账支付',
		  value: 5,
	  },{
		  label: '其它支付',
		  value: 6,
	  },{
		  label: '建行二维码',
		  value: 7,
	  },{
		  label: 'pos支付',
		  value: 8,
	  },{
		  label: '银联公众号支付',
		  value: 9,
	  },{
		  label: '银联二维码',
		  value: 10,
	  },{
		  label: '银联btoc支付',
		  value: 11,
	  },{
		  label: '微信二维码支付',
		  value: 12,
	  },{
		  label: '支付宝二维码支付',
		  value: 13,
	  },{
		  label: '微信app支付',
		  value: 20,
	  },{
		  label: '支付宝app支付',
		  value: 21,
	  },{
		  label: '微信app银联渠道支付',
		  value: 22,
	  },{
		  label: '支付宝app银联渠道支付',
		  value: 23,
	  },{
		  label: '云闪付支付',
		  value: 24,
	  },{
		  label: '农行微信分账支付',
		  value: 25,
	  },{
		  label: '农行支付宝分账支付',
		  value: 26,
	  },{
		  label: '农行app支付',
		  value: 27,
	  },{
		  label: '纯余额支付',
		  value: 28,
	  }],
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
	  loading: false,
	  sendLoading: false, // 发货loading
	  orderRefundLoading:false, //退换货loading

	  isSend: false,// 是否有发货权限
	  isRefund: false,// 是否有退款权限
    };
  },
  activated() {
	let isRefresh = this.$route.query.isRefresh // 1不刷新 2或者undefined刷新
	if(isRefresh && isRefresh == 1) return
	this.loading = true
	this.initData()
  },	
  methods: {
	// 初始化信息
	initData() {
	//   const start = new Date();
	//   const end = new Date();
	//   start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
	//   this.dateArr = [start, end];
	  this.dateArr = ''
      this.data = [];
	  this.curPage = 1;
	  this.totalPage = null;
	  this.sendLoading = false;
	  this.orderRefundLoading = false;
	  this.orderType = '';
	  this.payType = '';
	  this.getOrderList();
	   this.handlePermission();
	},
	// 判断当前页面都有什么权限
    handlePermission() { 
      let that = this;
      that.utils.getPermissionList(that,data =>{
        data.forEach(item =>{
          if(item.title == '发货') {
            that.isSend = true
          }	
          if(item.title == '退款') {
            that.isRefund = true
          }	
        })
      },'','goods_mod_id')
    },
	// 查询
	search(){
	 	this.data = []
		this.curPage = 1
		this.totalPage = null
		this.sendLoading = false
		this.orderRefundLoading = false
		this.getOrderList()
	},
	// 获取订单列表
	getOrderList() {
		let that = this
		that.loading = true
		that.ajax('orderLists',{
            page: that.curPage,
			size: 10,
			order_status: that.orderType,
			pay_type: that.payType,
			start_time: that.dateArr ? (that.utils.dateFormat('yyyy-MM-dd',that.dateArr[0]) + ' 00:00:00') : '',
			end_time: that.dateArr ? (that.utils.dateFormat('yyyy-MM-dd',that.dateArr[1]) +' 23:59:59') : '',
        },'获取订单列表失败',res =>{
			that.loading = false
			if(res.errno == 0) {
                that.curPage = Number(res.data.current_page)
				that.totalPage = res.data.total
				res.data.data.forEach(item =>{
					item.add_time = that.utils.dateFormat('yyyy-MM-dd HH:mm:ss', new Date(item.add_time*1000))
				})
				that.data = res.data.data
			}
		}, err =>{
			that.loading = false
		})
    },
    // 处理分页
	handleCurPageChange(val) {
		this.curPage = val;
		this.getOrderList()
	},
	// 订单详情
	detail(row) {
		this.$router.push({
			path: '/cooperateOrderDetail',
			query: {
				id: row.order_id
			}
		})
	},
	// 发货
	sendGoods(row,index) {
		this.$confirm('确定发货吗?', '提示', {
			confirmButtonText: '确定',
			cancelButtonText: '取消',
			type: 'warning',
			center: true
			}).then(() => {
				this.sendGoodsApi(row)
			}).catch(() => {});
	},
	// 请求发货api
	sendGoodsApi(row) {
		let that = this
        that.sendLoading = true;
		that.ajax('send',{
            order_id: row.order_id,
			order_status: 7,
        },'发货失败',res =>{
			that.sendLoading = false
			if(res.errno == 0) {
				that.$message.success('发货成功')
				that.$set(row,'order_status', 7)
			}
		}, err =>{
			that.sendLoading = false
		})
	},
	// 退换货
	orderRefund(row,index) {
		this.$confirm('确定退款吗?', '提示', {
			confirmButtonText: '确定',
			cancelButtonText: '取消',
			type: 'warning',
			center: true
		}).then(() => {
			this.orderRefundApi(row)
		}).catch(() => {});
	},
	// 请求退换货api
	orderRefundApi(row) {
		let that = this
        that.orderRefundLoading = true;
		that.ajax('orderRefund',{
            order_id: row.order_id,
        },'退款失败',res =>{
			that.orderRefundLoading = false
			if(res.errno == 0) {
				that.$message.success('退款成功')
				that.$set(row,'order_status', 6)
			}
		}, err =>{
			that.orderRefundLoading = false
		})
	},
	// // 删除订单
	// delOrderItem(row,index) {
	// 	 this.$confirm('此操作将永久该订单, 是否继续?', '提示', {
    //       confirmButtonText: '确定',
    //       cancelButtonText: '取消',
	// 	  type: 'warning',
	// 	  center: true
    //     }).then(() => {
         
    //     }).catch(() => {
                   
    //     });
	// },
	// // delOrder
	// delOrderApi(row,index) {
	// 	let that = this
    //     that.delLoading = true;
	// 	that.ajax('del',{
    //         order_id: row.order_id,
	// 		order_status: 7,
    //     },'发货失败',res =>{
	// 		that.delLoading = false
	// 		if(res.errno == 0) {
	// 			that.$message.success('发货成功')
	// 			that.$set(row,'order_status', 7)
	// 		}
	// 	}, err =>{
	// 		that.delLoading = false
	// 	})
	// },
  }
};
</script>
<style lang="less">
.order-list {
  padding: 20px;
  .el-pagination{
        float: right;
        margin-top: 30px;
        margin-bottom: 30px;
	}
	.operate-con {
	  padding: 10px 0;
    .search-con {
	  float: left;
	  width: 100%;
	  .el-form-item {
		width: 25%;
		float: left;
		margin-bottom: 0;
		.el-form-item__label {
			font-size: 0.9rem;
		}
		.el-form-item__content {
			margin-right: 0;
			.el-input {
			width: 100%;
			.read-idCard {
				color: #3B6AF1;
				background: #F0F8FF;
				font-size: 0.75rem;
				cursor: pointer;
			}
		}
		}
		.el-range-editor {
			width: 90%;
		}
		
	}
	}
	.admin-search-con {
		.el-form-item {
			width: 18%;
			.el-range-editor {
				width: 100%;
			}
		}
		.el-form-item:last-of-type{
			width: 25%;
		}
	}
    .btn-con {
      float:right;
    }
  }
    .table-con {
		.inner-table-tit-con {
			display: flex;
			align-content: center;
			justify-content: space-between;
			.inner-table-tit {
				flex: 1;
				display: flex;
				align-items: center;
				justify-content: center;
				font-weight: bold;
				font-size: 1.2rem;
				padding-bottom: 1.25rem;
				text-align: center;
				.btn-con {
					display: flex;
					align-items: center;
					justify-content: center;
				}
			}
		}	
	}
}

.admin-tmp-con { 
  padding: 0;
  width: 100%; 
  height: 100%; 
  display: flex;
  flex-direction: column;
  .breadcrumb-con {
    margin-bottom: 0;
  }
  .admin-con {
    display: flex;
    width: 100%;
    height: 100%;
    .org-menu-con {
      width: 16rem;
      height: 100%;
      background: #F8FCFF;
      .level2-title {
          color: #8F9BA7;
      }
      .level2-title::after {
        content: '';
        position: absolute;
        top: 50%;
        left: 35px;
        width: 5px;
        height: 5px;
        border-radius: 50%;
        background: #8F9BA7;
        transform: translateY(-50%);
      }
      .is-opened {
        .el-icon-arrow-down {
          color: #409EFF;
        }
        .is-active {
          .level2-title {
            color: #409EFF;
          }
          .level2-title::after {
            background: #409EFF;
          }
        }
        
      }
      .is-active {
        background: #fff!important;
      }
      .menu-title {
        width: 10rem;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      .menu-item-title {
        width: 7.5rem;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
	  }
	  .no-menu-con {
        position: absolute;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        .menu-absolute-center {
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%,-50%);
          .text {
            font-size: 0.9rem;
            color: #999;

          }
        }
      }
    }
    .right-con {
      position: relative;
      width: calc(100% - 16rem);
      height: 99%;
      padding: 10px;
      .info-con {
        position: relative;
        width: 100%;
        padding-top: 10px;
      }
      .operate-con {
        padding: 0;
      }
      .operate-con+.el-table {
        margin-top: 10px;
      }
    }
    
    .hide-menu {
      width: 54px;
    }
    .page-con {
      position: absolute;
      bottom: 0;
      right: 5rem;
    }
  }
}

</style>