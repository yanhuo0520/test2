<template>
    <div class="fullForm" v-loading="loading">
        <div class="breadcrumb-con">
            <img class="left-icon" src="../../images/breadcrumb-left-icon.png" alt="">
            <div class="breadcrumb-info">
                <el-breadcrumb separator-class="el-icon-arrow-right">
                    <el-breadcrumb-item>监管查询报表</el-breadcrumb-item>
                    <el-breadcrumb-item class="breadcrumb-tit">盈余及盈余分配表</el-breadcrumb-item>
                </el-breadcrumb>
            </div>
          <el-button size="small" @click="initData()">刷新</el-button>
        </div>

        <div class="search-con">
            <el-form class="clearfix"  label-width="80px">
                <el-form-item label="机构名称">
                    <el-input placeholder="请输入机构名称" v-model="coopera_name"></el-input>
                </el-form-item>
                <el-form-item label="年份">
                    <el-date-picker
                        v-model="year"
                        type="year"
                        placeholder="选择年">
                        </el-date-picker>
                </el-form-item>
                <el-form-item label="填报日期">
                    <el-date-picker
                    v-model="date"
                    type="date"
                    value-format="yyyy-MM-dd"
                    placeholder="选择日期">
                    </el-date-picker>
                </el-form-item>
                <el-button type="primary" size="small" :loading="loading">{{loading ? '获取数据中...' : '查询'}}</el-button>
            </el-form>
        </div>

        <div class="table-con">
            <div>
              <el-table :data="tableData" v-if="tableData && tableData.length > 0"  default-expand-all border>
                <el-table-column type="expand">
                    <template slot-scope="props">
                        <el-table :data="props.row.children" :show-header="false" default-expand-all border>
                            <el-table-column type="expand">
                                <template slot-scope="prop">
                                    <el-table :data="prop.row.children" :show-header="false" border>
                                        <el-table-column width="50" align="center"></el-table-column>
                                        <el-table-column prop="title" label="项目名称" align="center">
                                            <template slot-scope="scope">
                                                <span v-if="scope.row.num">({{scope.row.num}})</span>{{scope.row.title}}
                                            </template>
                                        </el-table-column>
                                        <el-table-column prop="benyue" label="本月数" align="center" ></el-table-column>
                                        <el-table-column prop="bennian" label="本年累计数" align="center"></el-table-column>
                                    </el-table>
                                </template>
                            </el-table-column>
                            <el-table-column prop="title" label="项目名称" align="center">
                                <template slot-scope="scope">
                                    <span v-if="scope.row.num">({{scope.row.num}})</span>{{scope.row.title}}
                                </template>
                            </el-table-column>
                            <el-table-column prop="benyue" label="本月数" align="center"></el-table-column>
                            <el-table-column prop="bennian" label="本年累计数" align="center"></el-table-column>
                        </el-table>
                    </template>
                </el-table-column>
                <el-table-column prop="title" label="项目名称" align="center">
                    <template slot-scope="scope">
                        {{scope.row.title}}
                    </template>
                </el-table-column>
                <el-table-column prop="benyue" label="本月数" align="center"></el-table-column>
                <el-table-column prop="bennian" label="本年累计数" align="center"></el-table-column>
              </el-table>

              <!-- <el-pagination
              background
              :current-page="curPage"
              layout="prev, pager, next"
              :total="totalPage*10"
              @current-change="handleCurPageChange">
              </el-pagination> -->
          </div>
          <div v-if="!tableData || tableData.length == 0">
              <div class="no-data-con" >
                  <div class="absolute-center">
                      <div class="err-info-text ">暂无信息</div>
                  </div>
              </div>
          </div>
        </div>
    </div>
</template>
<script>
export default {
  name: "fullForm",
  data() {
    return {
      tableData: [
        {
          title: "一、营业收入",
          benyue: 0,
          bennian: 0,
          num: "",
          children: [
            {
              title: "资金占用费收入",
              benyue: 0,
              bennian: 0,
              num: "501",
              children: [
                {
                  title: "占用费收入",
                  benyue: 0,
                  bennian: 0,
                  num: "50101"
                },
                {
                  title: "消费借款占用费收入",
                  benyue: 0,
                  bennian: 0,
                  num: "50102"
                }
              ]
            },
            {
              title: "银行存款利息收入",
              benyue: 0,
              bennian: 0,
              num: "502"
            },
            {
              title: "其他收入",
              benyue: 0,
              bennian: 0,
              num: "503"
            }
          ]
        },
        {
          title: "二、营业支出",
          benyue: 0,
          bennian: 0,
          num: "",
          children: [
            {
              title: "互助金分红支出",
              benyue: 0,
              bennian: 0,
              num: "521",
              children: [
                {
                  title: "活期分红支出",
                  benyue: 0,
                  bennian: 0,
                  num: "52101"
                },
                {
                  title: "定期分红支出",
                  benyue: 0,
                  bennian: 0,
                  num: "52102"
                }
              ]
            },
            {
              title: "管理费用",
              benyue: 0,
              bennian: 0,
              num: "522",
              children: [
                {
                  title: "差旅费",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
                {
                  title: "折旧费",
                  benyue: 0,
                  bennian: 0,
                  num: "52200"
                },
                {
                  title: "管理费用",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
                {
                  title: "基础工资",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
                {
                  title: "办公费用",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
                {
                  title: "通讯费",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
                {
                  title: "招待费",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
                {
                  title: "水电费",
                  benyue: 0,
                  bennian: 0,
                  num: ""
                },
              ]
            },
            {
              title: "其他支出",
              benyue: 0,
              bennian: 0,
              num: "529",
              children:[
                  {
                      title:"短期投资",
                      benyue:0,
                      bennian:0,
                      num:""
                  },
                  {
                      title:"财务费用",
                      benyue:0,
                      bennian:0,
                      num:""
                  },
                  {
                      title:"交易性金融资产",
                      benyue:0,
                      bennian:0,
                      num:""
                  }
              ]
            }
          ]
        },
        {
            title:"三、净利润总额",
            benyue:0,
            bennian:0,
            num:""
        }
      ],
      curPage: 1,
      totalPage: 1,
      loading: false,
      coopera_name: "",
      year: "",
      date:""
    };
  },
  methods: {
    handleCurPageChange(val) {}
  }
};
</script>
<style lang="less">
.fullForm {
  padding: 20px;
  .el-pagination {
    float: right;
    margin-top: 30px;
    margin-bottom: 30px;
  }

  .search-con {
    margin-bottom: 10px;
    .el-form {
      width: 100%;
      display: flex;
    }
    .el-form-item {
      width: 20%;
      // float: left;
      margin-bottom: 0;
      .el-form-item__label {
        font-size: 0.9rem;
      }
      .el-form-item__content {
        margin-right: 30px;
        .el-input {
          width: 100%;
          .read-idCard {
            color: #3b6af1;
            background: #f0f8ff;
            font-size: 0.75rem;
            cursor: pointer;
          }
        }
      }
    }
  }
  .el-table__expanded-cell[class*="cell"] {
    padding: 0;
  }
}
</style>