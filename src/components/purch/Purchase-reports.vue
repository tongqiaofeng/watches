<template>
  <div class="Purchase-report" id="Purchase-report">
    <div class="report-center1" v-if="againclick.times == 0">
      <div class="report-heade">
        <!-- <span class="heade-title">采购报表</span> -->
        <img src="../../../src/assets/imgs/chart.png" alt="" class="bar-chart" @click="barchart" title="采购报表柱状图"
          style="cursor: pointer;" />
      </div>
      <div class="report-center2">
        <div class="report-form">
          <el-form label-width="100px" id="report-input" ref="form" :label-position="labelPosition">
            <div class="report-input">
              <!-- 管理员 -->
              <el-form-item label="采购员:" style="width: 50%;margin:10px 0px 6px;"
                :rules="[{ required: true, message: '采购员不能为空' }]" v-if="authorityList[32] == 1">
                <el-select v-model="backnicks.name" filterable placeholder="请选择" style="width: 90%;"
                  @change="currentSel">
                  <el-option v-for="item in adminlist" :key="item.id" :label="item.nick" :value="item.id"
                    style="width: 500px;" @click="currentSel">
                  </el-option>
                </el-select>
              </el-form-item>
              <!-- 采购员 -->
              <el-form-item label="采购员:" style="width: 50%;margin:10px 0px 6px;"
                :rules="[{ required: true, message: '采购员不能为空' }]" v-if="authorityList[32] != 1">
                <el-select v-model="buyerlimit.nick" filterable placeholder="请选择2" style="width: 90%;"
                  @change="currentSel">
                  <el-option :key="buyerlimit.id" :label="buyerlimit.nick" :value="buyerlimit.nick"
                    style="width: 500px;" @click="currentSel">
                  </el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="采购日期:" style="width: 50%;margin:10px 0px 6px;"
                :rules="[{ required: true, message: '采购日期不能为空' }]">
                <el-date-picker v-model="mytime" type="daterange" range-separator="~" @change="getMyDateTime2"
                  value-format="yyyy-MM-dd" start-placeholder="开始日期" end-placeholder="结束日期" style="width: 90%;">
                </el-date-picker>
              </el-form-item>
            </div>
            <!-- 采购状态 -->
            <div>
              <el-form-item label="货物状态:" style="width: 50%; padding-bottom: 10px; margin-bottom: 0;"
                :rules="[{ required: true, message: '采购日期不能为空' }]">
                <el-select v-model="value" filterable placeholder="请选择" style="width: 90%;" @change="saleType">
                  <el-option v-for="item in saletype" :key="item.type" :label="item.name" :value="item.type"
                    style="width: 500px;" @click="saleType">
                  </el-option>
                </el-select>
              </el-form-item>
            </div>

          </el-form>
        </div>

        <div class="report-showlist">
          <div class="report-num">
            <div class="watch-nums">
              <div class="watch-salenums">
                <p class="watch-nums-title">手表数量 :</p>
                <p class="watch-nums-total">{{ reportlist.watchTotal }}</p>
              </div>

              <div class="watch-sale-type" v-show="saletypenum==0">
                <div class="watch-sale">
                  <p>已出售 :</p>
                  <p>{{formatNumberRgx(reportlist.sellTotal)}}</p>
                </div>
                <div class="watch-sale">
                  <p>未出售 :</p>
                  <p>{{formatNumberRgx(reportlist.unSellTotal)}}</p>
                </div>
              </div>
            </div>
            <div class="watch-nums">
              <div class="watch-salenums">
                <p class="watch-nums-title">采购金额:</p>
                <p class="watch-nums-total">
                  {{ formatNumberRgx(reportlist.allMoney) + " " + "HKD" }}
                </p>
              </div>
              <div class="watch-sale-type" v-show="saletypenum==0">
                <div class="watch-sale">
                  <p>已出售金额 :</p>
                  <p>{{formatNumberRgx(reportlist.sellMoney) + " " + "HKD" }}</p>
                </div>
                <div class="watch-sale">
                  <p>未出售金额 :</p>
                  <p>{{formatNumberRgx(reportlist.unSellMoney) + " " + "HKD" }}</p>
                </div>
              </div>
            </div>
            <div class="excel">
              <a :href="eurl" class="urlimg" title="导出excel表格"></a>
            </div>
          </div>
          <div v-show="reportlist.watchTotal == 0" style="text-align: center;">
            <p>{{ hintMsg }}</p>
          </div>
          <div class="report-watch" id="report-watch" v-if="reportlist.watchTotal !== 0">
            <table>
              <tr class="table-tr">
                <th>图片</th>
                <th>品牌</th>
                <th>型号</th>
                <th>采购价格</th>
                <th>采购时间</th>
                <th>查看详情</th>
              </tr>
              <tr v-for="(item,index) in imgurl" :key="index" class="imgurlbox">
                <td>
                  <img v-image-preview
                    :src=" item.watchPic == null || item.watchPic == '' ? '' : img + '/img/watch/' + item.watchPic.split('|')[0] "
                    class="watch-img" />
                </td>
                <td>
                  <p>{{ item.watchBrand }}</p>
                </td>
                <td>
                  <P>{{ item.watchModel }}</P>
                </td>
                <td>
                  <P>{{formatNumberRgx(item.buy_watchPrice) + "  " + "HKD"}}</P>
                </td>
                <td>
                  <p>{{ item.buy_date }}</p>
                </td>
                <td class="salebox">
                  <img style="cursor:pointer" src="../../assets/imgs/details.png" alt=""
                    @click="detailspage(item.buy_id)" title="查看详情" />
                  <img v-show="item.flag==1" src="../../assets/imgs/sale.png" alt="" class="saleimg">
                </td>
              </tr>
            </table>
          </div>
        </div>
      </div>
      <!-- 分页 -->
      <div style="width: 100%;height: 50px;">
        <div style="margin:15px 0;position:absolute;right:6%;">
          <el-pagination @current-change="handleCurrentChange" :current-page="page"
            layout="total, prev, pager, next, jumper" :total="total">
          </el-pagination>
        </div>
      </div>
    </div>

    <Details :detailsSelect="detailsSelect" @backbuy="backbuy" v-if="againclick.times == 2">
    </Details>

    <div class="chartbox" v-if="againclick.times == 1">
      <div class="returnlastpage" style="margin-top: 0; " @click="backto">
        <div>
          <img src="../../assets/imgs/goback.png" />
        </div>
        <span class="font">返回</span>
      </div>

      <Report-xchart v-if="authorityList[32] == 1" :datanews="datanews" @backtime="backtime" @backnick="backnick">
      </Report-xchart>
      <Report-echart v-if="authorityList[32] != 1" :buyerlimit="buyerlimit"></Report-echart>
    </div>
  </div>
</template>
<script>
  export default {
    data() {
      return {
        hintMsg: "数据加载中...",
        img: this.$store.state.baseUrl,
        mytime: [],
        mytime1: "",
        mytime2: "",
        datanews: {
          startT: "",
          endT: "",
          nick: "公司",
          id: 0,
        },

        labelPosition: "left",
        watchdetails: {
          num: 0,
        },
        detailsSelect: {
          id: 1,
          num: 0,
          buy_commState: 0,
        },

        backnicks: {
          name: "公司",
          id: "",
        },
        // backnicksid:"",
        adminlist: {
          id: "",
          nick: "",
        },
        reportlist: {
          watchTotal: "",
          allMoney: "",
          watchList: [],
        },
        imgurl: {
          id: "",
          watchBrand: "",
          watchBrand: "",
          buy_watchPrice: "",
          buy_date: "",
          watchPic: "",
        },
        imgs: {
          id: "",
          watchBrand: "",
          watchBrand: "",
          buy_watchPrice: "",
          buy_date: "",
          watchPic: "",
        },
        watchList: [],
        tableData: [],
        eurl: "",
        buyerid: 0,
        nowDate: "", // 当前日期
        nowDatemonth: "", //本月1号
        total: 0,
        page: 1,
        pageNum: 10,
        buyerlimit: {
          id: 0,
          nick: "",
        },
        value: "全部（已出售+未出售）",
        saletype: [{
          type: 0,
          name: "全部（已出售+未出售）"
        }, {
          type: 1,
          name: "已出售"
        }, {
          type: 2,
          name: "未出售"
        }],
        saletypenum: 0,
        authorityList: [],
      };
    },
    props: ["againclick"],
    created() {
      this.authorityList = sessionStorage.getItem("authority").split("|");
      console.log(this.authorityList);
      this.$store.state.id = sessionStorage.getItem("id");
      console.log(this.$store.state.id);
      console.log(this.againclick.times);
    },
    mounted() {
      // 获取当前时间---默认时间
      this.adddate();
      this.mytime = [this.mytime1, this.mytime2];
      this.buyersid();
      if (this.authorityList[32] == 1) {
        this.getMyDateTime();
      }
    },
    methods: {
      saleType(val) {
        this.saletypenum = val;
        this.page = 1;
        this.getMyDateTime();
      },
      //采购员列表
      buyersid() {
        this.$axios
          .post(this.$store.state.baseUrl + "/purchaserList")
          .then((res) => {
            this.adminlist = res.data;

            console.log(res.data);
            if (this.authorityList[32] == 0) {
              for (let num = 0; num < this.adminlist.length; num++) {
                if (this.$store.state.id == this.adminlist[num].id //非管理员--打开页面只有自己的数据
                ) {
                  this.buyerlimit = this.adminlist[num];
                  this.buyerid = this.buyerlimit.id; //获取登陆用户的id（非管理员）
                  this.getMyDateTime();
                  break;
                }
              }
            }
          });
      },

      backto() {
        // this.isshow=false;
        this.againclick.times = 0;
        // this.backtime();
        if (this.authorityList[32] == 1) {
          this.buyerid = this.backnicks.id;
        }
        this.getMyDateTime();
      },
      backtime(val, newv) {
        this.mytime = [val, newv];

        console.log(this.mytime);
      },
      backnick(nickval, idval) {
        this.backnicks.name = nickval;
        this.backnicks.id = idval;
        // console.log(this.backnicks.name,this.backnicks.id);
      },
      // 点击手表跳转至详情页面
      detailspage(id) {
        this.againclick.times = 2;
        // console.log("跳转至详情页面");
        sessionStorage.setItem("buy_id", id);
        // console.log(this.detailsSelect.num);
      },
      // 分页
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.page = val;
        this.getMyDateTime();
        (function smoothscroll() {
          var currentScroll =
            document.documentElement.scrollTop || document.body.scrollTop;
          if (currentScroll > 0) {
            window.requestAnimationFrame(smoothscroll);
            window.scrollTo(0, currentScroll - currentScroll / 5);
          }
        })();
      },
      backbuy(val) {
        // console.log(val);

        this.againclick.times = val;
        this.detailsSelect.num = 0;
        // console.log(this.againclick.times);
      },
      // 获取默认时间
      adddate() {
        let yy = new Date().getFullYear();
        let mm = new Date().getMonth() + 1;
        let dd = new Date().getDate();
        mm = mm < 10 ? "0" + mm : mm;
        dd = dd < 10 ? "0" + dd : dd;
        this.mytime1 = yy + "-" + mm + "-" + "01";
        this.mytime2 = yy + "-" + mm + "-" + dd;
      },
      // 采购员ID  selval对应value值--id
      currentSel(selval) {
        this.buyerid = selval;
        this.page = 1;
        this.getMyDateTime();

        let obj = {};
        obj = this.adminlist.find((item) => {
          //面遍历的数据源
          return item.id === selval; //筛选出匹配数据
        });
        this.datanews.nick = obj.nick;
        this.datanews.id = this.buyerid;
      },
      // 时间变化
      getMyDateTime2() {
        this.page = 1;
        this.getMyDateTime();
      },
      getMyDateTime() {
        this.mytime1 = this.mytime[0];
        this.mytime2 = this.mytime[1];
        console.log(this.mytime);

        this.$axios
          .post(this.$store.state.baseUrl + "/purchaseReport", {
            id: this.buyerid,
            startTime: this.mytime1,
            endTime: this.mytime2,
            page: this.page,
            pageNum: this.pageNum,
            saleType: this.saletypenum
          })
          .then((res) => {
            console.log(this.saletypenum);
            this.reportlist = res.data; //报表
            this.imgurl = res.data.watchList;
            this.total = res.data.watchTotal;
            // console.log(this.reportlist);
            // console.log(res.data.watchList.length);
            // console.log(this.mytime2, this.mytime1);
            if (this.reportlist.watchTotal == 0) {
              this.hintMsg = "啊哦~ 暂无数据";
            }
          })
          .catch((err) => {
            console.log("reporlist method错误");
          });

        // 调用导出excel表格
        this.exportdata();
      },
      exportdata() {
        // 导出手动选择日期的excel表

        this.$axios
          .post(this.$store.state.baseUrl + "/exportData", {
            id: this.buyerid,
            startTime: this.mytime1,
            endTime: this.mytime2,
          })
          .then((res) => {
            this.tableData = res.data;
            this.eurl = this.$store.state.baseUrl + this.tableData.excelUrl;
          });
      },
      barchart() {
        this.againclick.times = 1;
        console.log("echart跳转");
        this.datanews.startT = this.mytime1;
        this.datanews.endT = this.mytime2;
      },
    },
  };
</script>
<style lang="scss" scoped>
  .Purchase-report {
    width: 100%;
  }

  .report-center1 {
    width: 95%;
    margin: 0 auto;
    padding: 50px 0 30px 0;
    font-size: 16px;
    background-color: #fff;
    border-radius: 30px;
    font-family: "微软雅黑";
  }

  .chartbox {
    width: 95%;
    margin: 0 auto;
    border-radius: 30px;
    background-color: #fff;
  }

  .report-center2 {
    width: 90%;
    margin: 0 auto;
  }

  .returnlastpage {
    width: 75px;
    height: 45px;
    margin: 23px;
    line-height: 45px;
    padding-top: 18px;
    display: flex;
    justify-content: space-between;
    cursor: pointer;
  }

  .returnlastpage img {
    width: 30px;
    height: 25px;
  }

  .returnlastpage div {
    margin-top: 5px;
  }

  .report-form {
    width: 100%;
    margin: 0 30px;
    background-color: #d7ebe7;
  }

  #Purchase-report .el-form-item__label {
    font-size: 16px;
    text-align: right;
  }

  .report-input {
    width: 100%;
    /* height: 80px; */

    display: block;
    display: flex;
    flex-wrap: nowrap;
    align-items: center;
  }

  .report-heade {
    display: flex;
    justify-content: space-between;
    flex-wrap: nowrap;
    width: 90%;
    margin: 0 auto;
    height: 64px;
    line-height: 64px;
    padding: 0 30px 20px;
    color: #2c3e50;
  }

  .report-heade img {
    width: 20px;
    height: 20px;
  }

  .urlimg {
    background-image: url(../../assets/imgs/excel.png);
    width: 20px;
    height: 20px;
    display: block;
    background-size: contain;
  }

  .salebox {
    position: relative;
  }

  .saleimg {
    position: absolute;
    top: 0px;
    right: 0px;
  }

  .excel {
    margin-right: 20px;
  }

  .excel img {
    width: 20px;
    height: 20px;
  }

  .heade-title {
    font-size: 18px;
    margin-left: 30px;
    font-weight: bold;
  }

  .report-time {
    display: flex;
    flex-wrap: nowrap;
    width: 90%;
  }

  .report-num {
    display: flex;
    flex-wrap: nowrap;
    margin-right: 8px;
    align-items: center;
  }

  .watch-nums {
    margin: 0 auto 16px;
    /* display: flex;
  flex-wrap: nowrap; */
    margin-left: 40px;
  }

  .watch-salenums {
    display: flex;
    flex-wrap: nowrap;

  }

  .watch-sale-type {
    margin-top: -30px;
  }

  .watch-sale {
    display: flex;
    flex-wrap: nowrap;
    margin-top: -20px;
  }

  .watch-sale>p {
    font-size: 12px;
  }

  .watch-sale-type {}

  .buyer {
    text-align: justify;
  }

  .watch-nums-title {
    font-size: 18px;
    font-weight: bold;
    color: #2c3e50;
    height: 50px;
    line-height: 50px;
  }

  .watch-nums-total {
    color: #2c3e50;
    height: 50px;
    line-height: 50px;
    margin-left: 20px;
  }

  .waech-nums-price {
    height: 40px;
    line-height: 40px;
  }

  .report-showlist {
    width: 100%;
    display: block;
    margin: 0 30px;
  }

  #report-watch table>tr {
    text-align: center;
  }

  .table-tr {
    height: 40px;
    line-height: 40px;
    border-bottom: 0;
  }

  #report-watch table>tr>td {
    background-color: #f3fbf9;
    border-bottom: 1px solid #d7ebe7;
    padding: 20px;
    width: 16%;
  }

  .watch-img {
    width: 100px;
    height: 100px;
    border-radius: 30px;
    -o-object-fit: cover;
    object-fit: cover;
  }
</style>