<template>
    <div class="post-list"  v-loading="loading">
		<div class="breadcrumb-con">
			<img class="left-icon" src="../../../images/breadcrumb-left-icon.png" alt="">
			<div class="breadcrumb-info">
				<el-breadcrumb separator-class="el-icon-arrow-right">
					<el-breadcrumb-item>农资商城</el-breadcrumb-item>
					<el-breadcrumb-item  class="breadcrumb-tit">部门列表</el-breadcrumb-item>
				</el-breadcrumb>
			</div>
			<el-button size="small" @click="initData()">刷新</el-button>
		</div>
		 <el-alert title="未设置权限的部门,将无法在添加员工页面选择此部门" type="warning"></el-alert>
		<div class="operate-con clearfix">
			<div class="btn-con" v-if="!loading && isAdd">
				<el-button slot="trigger" size="small"  type="primary" @click="addEditPost(1)" >添加部门</el-button>
			</div>
		</div>
        <div class="table-con">
            <template v-if="data && data.length > 0">
                <el-table :data="data" border >
                    <el-table-column prop="post_id" label="ID" width="60" align="center"></el-table-column>
                    <el-table-column prop="post_code" label="部门编号" align="center"></el-table-column>
                    <el-table-column prop="post_name" label="部门名称" align="center"></el-table-column>
                    <el-table-column prop="post_desc" label="部门描述" align="center"></el-table-column>
                    <el-table-column prop="add_time" label="工作部门设置时间" align="center"></el-table-column>
                    <el-table-column label="操作" width="220px" align="center" v-if="isSetPermission || isEdit || isDel" >
                        <template slot-scope="scope">
                            <el-button plain size="small" :type="scope.row.goods_mod_id ? '' : 'warning'" @click="showSetPermissionDialog(scope.row)" v-if="isSetPermission" >{{scope.row.goods_mod_id ? '编辑' : '设置'}}权限</el-button>
                            <el-button plain size="small" type="primary" @click="addEditPost(2,scope.row,scope.$index)" v-if="isEdit">编辑</el-button>
                            <!-- <el-button size="small" type="danger" @click="delPost(scope.row,scope.$index)" v-if="isDel">删除</el-button> -->
                        </template>
                    </el-table-column>
                </el-table>
            </template>
            <template v-if="!data || data.length == 0">
                <div class="no-data-con" >
                    <div class="absolute-center">
                        <div class="err-info-text ">暂无部门列表</div>
                    </div>
                </div>
            </template>
            <el-pagination @current-change="handleCurPageChange" v-if="totalPage && totalPage > 0" background :current-page="curPage" layout="prev, pager, next" :total="totalPage*10"> </el-pagination>
        </div>

        <el-dialog :title="postType == 1 ? '添加部门' : '编辑部门'" :visible.sync="postFormDialog" class="post-dialog">
			<el-form  ref="postForm"  :model="postForm" :rules="postRules" label-width="100px" class="post-form clearfix" >
				<el-form-item label="部门名称:" prop="post_name">
					<el-input v-model="postForm.post_name" placeholder="请输入部门名称"></el-input>
				</el-form-item>
                <el-form-item label="部门描述:" >
					<el-input v-model="postForm.post_desc" placeholder="请输入部门描述"></el-input>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="postFormDialog = false">取 消</el-button>
				<el-button type="primary" @click="postSubmit('postForm')" :loading="postSubmitLoading">确 定</el-button>
			</div>
		</el-dialog>

		<el-dialog title="设置商城权限" :visible.sync="setPermissionDialog"  @close="closePermissionDialog()">
			<el-tree :check-strictly="isStrictly" :data="permissionData" show-checkbox default-expand-all node-key="mod_id" ref="tree" highlight-current :props="defaultProps"> </el-tree>
			<div slot="footer" class="dialog-footer">
				<el-button @click="setPermissionDialog = false">取 消</el-button>
				<el-button type="primary" @click="permissionSubmit()" :loading="permissionSubmitLoading">确 定</el-button>
			</div>
		</el-dialog>

    </div>
</template>
<script>
export default {
  name: "postList",
  data() {
    return {
      data: [],
      curPage: 1,
      totalPage: 0,
      loading: false,
      
      postType: '',
	  postFormDialog: false, //部门form表单弹窗
	  postIndex: '', // 当前操作编辑部门列表的下标
	  postForm: {
		  
	  },
	  postRules: {
		  post_name: [
            { required: true, message: '请输入部门名称', trigger: 'blur' }
		  ],
		//    post_desc: [
        //     { required: true, message: '请输入部门描述', trigger: 'blur' }
		//   ],
	  },
	  postSubmitLoading: false,

	curPostId: '', // 当前设置权限选中的post_id
	setPermissionDialog: false, //设置权限dialog弹窗
	curModIdArr: [],
	permissionData: [],
	defaultProps: {
		children: 'child',
		label: 'title'
	},

	permissionSubmitLoading: false,
	isStrictly: true, // 设置权限选中复选框是否禁止父子级联动选择

	isEdit: false,// 是否有编辑权限
	isDel: false, //是否有删除权限
	isAdd: false, //是否有添加权限
	isSetPermission: false, //是否有设置权限 权限
    };
  },
  activated() {
	  this.initData()
  },	
  methods: {
	// 初始化信息
	initData() {
      this.data = []
      this.curPage = 1
	  this.getPostList()
	  this.handlePermission()
	},
	// 判断当前页面都有什么权限
	handlePermission() { 
		let that = this;
		that.utils.getPermissionList(that,data =>{
			data.forEach(item =>{
				if(item.title == '编辑') {
					that.isEdit = true
                }	
                if(item.title == '删除') {
					that.isDel = true
                }	
                if(item.title == '添加') {
					that.isAdd = true
				}	
				if(item.title == '设置权限') {
					that.isSetPermission = true
				}
			})
		},'','goods_mod_id')
	},
	// 获取部门列表
	getPostList(isAddEdit) {
        let that = this
        if(isAddEdit) {
            that.loading = false;
        } else {
            that.loading = true;
        }
		that.ajax('postLists',{
            page: that.curPage,
			size: 10,
        },'获取部门列表失败',res =>{
			that.loading = false
			if(res.errno == 0) {
                that.curPage = Number(res.data.current_page)
				that.totalPage = res.data.total
				if(res.data.data) {
					res.data.data.forEach((item,index) =>{
						if(item.post_id == 6) {
							res.data.data.splice(index,1)
						}
					})
				}
				that.data = res.data.data
				that.getModuleLists()
			}
		}, err =>{
			that.loading = false
		})
	},
	// 获取权限列表
	getModuleLists() {
		let that = this
		let userInfo = JSON.parse(localStorage.getItem('userInfo'))
		that.ajax('moduleLists',{
			post_id: userInfo.post_id,
			type: 2
			// post_id: 0,	
		},'获取权限列表失败',res =>{
			if(res.errno == 0) {
				that.permissionData = res.data
			}
		}, err =>{
		})
	},
    // 处理分页
	handleCurPageChange(val) {
		this.curPage = val;
		this.getPostList()
    },
    // 添加编辑产品
	addEditPost(type,row,index) { // type 1-添加 2-编辑
		if(type == 1) {
			this.postForm = {
                post_name: '',
				post_desc: '',
				post_id: '',

			}
		} else {
			this.postIndex = index
			this.postForm = {
                post_id: row.post_id,
                post_name: row.post_name,
                post_desc: row.post_desc,
			}
			
		}
		this.postType = type
		this.postFormDialog = true
	},
	// 添加编辑部门提交
	postSubmit(formName) {
		let that = this;
		let type = that.postType
		that.$refs[formName].validate((valid) => {
          if (valid) {
              that.postSubmitLoading = true
			  let params = JSON.parse(JSON.stringify(that.postForm))
			//   params.post_id = ''
              that.ajax('postAdd',params,(type == 1 ? '添加' : '编辑')+ '失败',res =>{
                that.postSubmitLoading = false
                if(res.errno == 0) {
					that.$message.success((type == 1 ? '添加' : '编辑')+'成功')
					that.getPostList(true)
					that.resetForm(formName)
					that.postFormDialog = false
                }
              }, err =>{
                   that.postFormDialog = false
              })
          } else {
            setTimeout(()=>{
                let isError= document.getElementsByClassName("is-error");
                let firstErrInput = isError[0].querySelector('input')
				firstErrInput.focus();
            },100);
            return false;
          }
        });
	},
	//删除部门
	delPost(row,index) {
		let that = this;
		that.$confirm('此操作将永久删除该部门, 是否继续?', '提示', {
			confirmButtonText: '确定',
			cancelButtonText: '取消',
			type: 'warning',
			center: true
			}).then(() => {
				that.ajax('postDel',{
					post_id: row.post_id
				},'删除失败',res =>{
					if(res.errno == 0) {
						that.$message.success('删除成功')
						that.data.splice(index,1)
						if(that.data && that.data.length == 0) {
							that.getPostList()
						}	
					}
				}, err =>{
				})
				
			}).catch(() => {});
	},
	// 显示设置权限dialog弹窗
	showSetPermissionDialog(row) {
		this.curModIdArr = row.goods_mod_id ? row.goods_mod_id.split(',') : []
		this.setPermissionDialog = true
		this.curPostId = row.post_id
		setTimeout(() =>{
			this.$refs.tree.setCheckedKeys(this.curModIdArr);
			this.isStrictly = false
		})

	},
	// 关闭设置权限dialog弹窗
	closePermissionDialog() {
		this.isStrictly = true
		this.$refs.tree.setCheckedKeys([]);
	},
	// 设置权限确定按钮
	permissionSubmit() {
		let that = this;
		that.permissionSubmitLoading = true
		that.ajax('goodsAddModule',{
			post_id: that.curPostId,
			goods_mod_id: that.$refs.tree.getCheckedKeys().join(',')
		},'设置权限失败',res =>{
			that.permissionSubmitLoading = false
			if(res.errno == 0) {
				that.$message.success('设置成功')
				that.$refs.tree.setCheckedKeys([]);
				that.setPermissionDialog = false
				that.getPostList(true)
				that.isStrictly = true
			}
		}, err =>{
			that.permissionSubmitLoading = false
		})
	},
    // 重置form表单
      resetForm(formName) {
        this.$refs[formName].resetFields();
      }

  }
};
</script>
<style lang="less">
.post-list {
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
			.read-idCard {
				color: #3B6AF1;
				background: #F0F8FF;
				font-size: 0.75rem;
				cursor: pointer;
			}
		}
		}
		
	}
    }
    .btn-con {
      float:right;
    }
  }
    .table-con {
	
	}
	.post-dialog {
		.el-dialog {
			width: 30%;
			.el-form {
				padding: 0 2rem;
				.el-form-item__content {
					.el-select {
						.el-input {
							width: 100%;
						}
					}
					.el-input {
						width: 100%;
					}
				}
			}
		}
	}
	


 
}

</style>