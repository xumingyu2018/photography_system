<template>
  <div>
    <div v-for="stra in strategy" :key="stra.Id">
      <img
        :src="stra.url"
        alt
        class="jumbotron"
        style="height:400px; width:800px;margin-left:100px"
      />
      <div class="container-fluid" style="background-color: #fafafa;">
        <div class="container" style="padding-top:10px;padding-bottom:10px;">
          <span class="con-span">约拍地点</span>
          <span>></span>
          <span class="con-span">{{ stra.address }}</span>
          <div id="title">
            <h2>{{ stra.title }}</h2>
            <div id="h2-right">
              <div class="ext-r row" style="justify-content:space-around;">
                <div
                  class="img-span"
                  @click="updateCollectionNum()"
                  id="operation"
                  v-if="stra.ssStatus < 0 ? false : true"
                ></div>
              </div>
            </div>
          </div>
          <p>初次访问{{ stra.address }}的蜂蜂会选择这条线路</p>
        </div>
      </div>

      <div class="container">
        <div class="main">
          <h5>路线概览</h5>
          <div class="route" v-html="stra.content">
            <!-- 改名了 -->
          </div>
        </div>
        <!-- 评论内容 -->
        <div class="container">
          <div class="com-box mt-5">
            <h2>评论</h2>
            <div>
              <ul id="comments" data-page="1" data-id="0">
                <li
                  class="clearfix comment_item"
                  data-id="1203904"
                  data-replied="0"
                  v-for="dis in discuss.slice(
                  (currentPage - 1) * pagesize,
                  currentPage * pagesize
                )"
                  :key="dis.commentId"
                >
                  <div class="img mr-2" @click="go(dis.userId)">
                    <img style="height:50px;width:50px;border-radius:50%" src="../../../assets/touxiang.png" />
                  </div>
                  <div class="info">
                    <a @click="go(dis.userId)">{{ dis.username }}:</a>
                    <span class="com-cont ml-1">{{ dis.content }}</span>
                    <br />
                    <div class="info-span">
                      <h4>{{ dis.creattime }}</h4>
                    </div>
                  </div>
                </li>
              </ul>
              <div class="block">
                <el-pagination
                  :page-size="pagesize"
                  :pager-count="11"
                  layout="prev, pager, next"
                  :total="allpages"
                  @current-change="current_change"
                ></el-pagination>
              </div>
            </div>
          </div>
          <!-- 最后的插入评论 -->
          <div class="clearfix com-form">
            <div class="fm-tare user-log">
              <div v-if="isShow === true">
                <textarea v-model="newcommentContent" placeholder="说点什么吧..." id="textarea"></textarea>
              </div>
              <div v-if="isShow === false" id="readonly" class="ml-5 mb-5">
                <h3 style="color:red;text-align:center">由于您的当前用户状态不正常，已禁止评论功能</h3>
              </div>
              <el-form v-if="isShow === true">
                <el-form-item>
                  <el-button @click="addComment()" id="toke">评论</el-button>
                </el-form-item>
              </el-form>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- 返回顶部 -->
    <el-backtop :bottom="200"></el-backtop>
  </div>
</template>
<script>
import jwt_decode from "jwt-decode";

export default {
  name: "linestrategy",
  data() {
    return {
      info: [],
      strategy: [],
      hotarticles: [],
      // strategyType: "personalrow"
      //登录用户信息
      role: [],
      //全部评论
      discuss: [],
      //插入评论内容
      newcommentContent: "",
      //全部回复
      replys: [],
      //插入回复
      // newReplyContent: "",
      //当前登录用户id
      userId: "",
      loading: false,
      isShow: true,
      currentPage: 1,
      pagesize: 10,
      allpages: 0,
      userStatus: ""
    };
  },
  created() {
    //获取传来的攻略类型和id
    var info = JSON.parse(sessionStorage.getItem("info")); //info=[type,id]
    this.info = info;
    console.log(this.info);
    this.getrowdetail();
    this.getRowComment()
  },
  methods: {
    getrowdetail() {
      console.log(this.info);
      this.$axios
        .post("/api/queryRowByGoodsId", {
          Id: this.info.id
        })
        .then(res => {
          this.strategy = res.data;
        })
        .catch(err => {
          console.log("错误信息" + err);
        });
    },
    getRowComment(){
      console.log(this.info.id)
        this.$axios
        .post("/api/queryCommentByGoodsId2", {
          goodsid: this.info.id
        })
        .then(res => {
          console.log(res)
          this.discuss = res.data;
          this.allpages = res.data.length;
        })
        .catch(err => {
          console.log("错误信息" + err);
        });
    },
    //更新点赞数
    updateCollectionNum() {
      if (localStorage.getItem("mytoken")) {
        var judge;
        this.$axios
          .post("http://localhost:3000/operation/collect", {
            strategyId: this.info.id,
            strategyType: this.info.type,
            userId: this.userId
          })
          .then(res => {
            console.log(res);
            judge = parseInt(res.data.data);
            if (judge == 1) {
              this.strategy[0].prCollectionNum =
                parseInt(this.strategy[0].prCollectionNum) + 1;
              this.$message({
                showClose: true,
                message: "点赞成功！",
                type: "success"
              });
              $("#operation").addClass("operated");
              $("#icon").removeClass("el-icon-star-off");
              $("#icon").addClass("el-icon-star-on");
            } else if (judge == -1) {
              this.strategy[0].prCollectionNum =
                parseInt(this.strategy[0].prCollectionNum) - 1;
              if (this.strategy[0].prCollectionNum < 0) {
                this.strategy[0].prCollectionNum = 0;
              }
              this.$message({
                showClose: true,
                message: "取消点赞成功！",
                type: "warning"
              });
              $("#operation").removeClass("operated");
              $("#icon").removeClass("el-icon-star-on");
              $("#icon").addClass("el-icon-star-off");
            }
          })
          .catch(err => {
            console.log("错误信息" + err);
          });
      } else {
        this.$message({
          showClose: true,
          message: "亲，请先登录呦！",
          type: "warning"
        });
      }
    },
    //更新点赞数
    updateLikeNum() {
      if (localStorage.getItem("mytoken")) {
        var judge;
        this.$axios
          .post("http://localhost:3000/operation/like", {
            strategyId: this.info.id,
            strategyType: this.info.type,
            userId: this.userId
          })
          .then(res => {
            console.log(res);
            judge = parseInt(res.data.data);
            if (judge == 1) {
              this.strategy[0].prLikeNum =
                parseInt(this.strategy[0].prLikeNum) + 1;
              console.log(this.strategy.prLikeNum);
              this.$message({
                showClose: true,
                message: "点赞成功！",
                type: "success"
              });
              $("#operation1").addClass("operated");
            } else if (judge == -1) {
              this.strategy[0].prLikeNum =
                parseInt(this.strategy[0].prLikeNum) - 1;
              console.log(this.strategy.prLikeNum);
              this.$message({
                showClose: true,
                message: "取消点赞成功！",
                type: "warning"
              });
              $("#operation1").removeClass("operated");
            }
          })
          .catch(err => {
            console.log("错误信息" + err);
          });
      } else {
        this.$message({
          showClose: true,
          message: "亲，请先登录呦！",
          type: "warning"
        });
      }
    },
    //获取头像
    getHeadPic(pic) {
      //给图片名加上服务器端访问路径
      let path = "";
      if (pic == null || pic == "" || pic == "headPic") {
        pic = "primaryHead.jpeg";
      }
      path = "http://localhost:3000/uploadHeadPic/" + pic;
      return path;
    },
    getCoverPic(pic) {
      //给图片名加上服务器端访问路径
      if (pic == "cover" || pic == null) {
        pic = "primaryCover.jpg";
      }
      let path = "http://localhost:3000/coverPic/" + pic;
      console.log(path);
      return path;
    },
    // 跳转到作者首页
    goFocus(strategyuserId) {
      var strategyuserId = JSON.stringify(strategyuserId);
      sessionStorage.setItem("strategyuserId", strategyuserId);
      this.$router.push("/index/focus");
    },

    //添加评论
    addComment() {
      if (localStorage.getItem("mytoken")) {
        let tuser = JSON.parse(localStorage.getItem("tuser"));
        let data = {
          username: tuser.username,
          content: this.newcommentContent,
          creattime: "",
          goodsid: this.info.id,
          info: ""
        };
        console.log(data);
        this.$axios
          .post("/api/addRowComent", data)
          .then(response => {
            console.log(response);
            this.$message({
              showClose: true,
              message: "评论成功",
              type: "warning"
            });
            this.getRowComment();
            this.newcommentContent = "";
          })
          .catch(function(error) {
            console.log(error);
          });
      } else {
        this.$message({
          showClose: true,
          message: "亲，请先登录呦！",
          type: "warning"
        });
      }
    },
    // 删除评论
    delComment(commentId) {
      // if(confirm()){}
      this.$confirm("此操作将删除该评论, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$axios
            .post("http://localhost:3000/operation/deldiscuss", {
              commentId: commentId,
              strategyId: this.info.id,
              // userId:, 在后台token获取
              strategyType: this.info.type
              // commentContent,strategyId,userId,commentTime,strategyType
            })
            .then(res => {
              //删除评论后实时刷新评论
              this.$axios
                .post("http://localhost:3000/operation/seldiscuss", {
                  strategyId: this.info.id,
                  strategyType: this.info.type
                })
                .then(res => {
                  this.discuss = res.data.data;
                  this.allpages = res.data.data.length;
                  if (this.allpages % this.pagesize == 0) {
                    this.currentPage = this.currentPage - 1;
                  }
                })
                .catch(err => {
                  console.log("错误信息" + err);
                });
            })
            .catch(err => {
              console.log("错误信息" + err);
            });
        })
        .catch(() => {
          // this.$message({
          //   type: 'info',
          //   message: '已取消删除'
          // });
        });
    },
    //显示提示框
    openVn() {
      const h = this.$createElement;
      this.$message({
        message: h("p", null, [
          h("span", null, "内容可以是 "),
          h("i", { style: "color: teal" }, "VNode")
        ])
      });
    },
    current_change: function(currentPage) {
      this.currentPage = currentPage;
    }
  }
};
</script>
<style scoped>
* {
  margin: 0;
  padding: 0;
}
.jumbotron {
  height: 40rem;
  width: 100%;
}
.container {
  margin: 0 auto;
}
.container-fluid .container .con-span {
  color: #888;
  font-size: 12px;
}

.container-fluid .container .con-span:hover {
  color: #ff9d00;
}

#title {
  height: 80px;
  line-height: 80px;
}

#title h2 {
  display: inline-block;
  width: 60%;
}

#title > div {
  float: right;
  /* display: inline-block; */
  width: 40%;
}
#title #h2-right .img-span:not(:last-child):hover {
  color: #ff9d00;
  cursor: pointer;
}
#title #h2-right .img-span:last-child:hover {
  color: red;
  cursor: pointer;
}
.operated {
  color: #ff9d00;
}
.container-fluid .container p span {
  background-color: #ff9d00;
}

.con-main {
  clear: both;
  text-indent: 2em;
}
/* 评论栏 */
.l-comment {
  margin-top: 85px;
}

.com-form .fm-tare textarea {
  height: 200px;
  width: 1000px;
  padding: 14px;
  border: 1px solid #e5e5e5;
  resize: none;
  outline: 0;
  border-radius: 5px;
  font-size: 14px;
  color: #666;
}

.com-form .user-log textarea {
  border: 1px solid #ffa200;
}

button,
input,
optgroup,
select,
textarea {
  color: inherit;
  font: inherit;
  margin: 0;
}

.com-box ul li {
  border-top: 1px solid #e5e5e5;
  padding: 15px 0;
}

.com-box h2 {
  font-size: 24px;
  color: #333;
  font-weight: normal;
  line-height: 24px;
  margin: 36px 0;
}

.com-box h2 {
  font-size: 24px;
  color: #333;
  font-weight: normal;
  line-height: 24px;
  margin: 36px 0;
}

.com-box {
  border-top: 1px solid #e5e5e5;
}

.com-form {
  margin-top: 50px;
}
.com-form .fm-tare button {
  width: 114px;
  height: 40px;
  background: #ffa200;
  border: 0;
  outline: 0;
  cursor: pointer;
  display: block;
  margin: 20px 0;
  border-radius: 5px;
  font-size: 16px;
  color: rgb(255, 255, 255);
  text-align: center;
  padding: 0;
  line-height: 30px;
  margin-left: 850px;
}

li {
  list-style: none;
}

.com-box li .img {
  border-radius: 50%;
  /* display: inline-block; */
  float: left;
  cursor: pointer;
}
.img span {
  color: #ff9d00;
}

.com-box .info {
  width: 1000px;
  float: left;
}

.com-box .info a {
  color: #ff9d00;
  cursor: pointer;
}
.com-box .info #reply-h3 {
  font-size: 18px;
  color: blue;
  font-weight: normal;
  line-height: 28px;
}

.com-box .info h4 {
  display: inline;
  font-size: 12px;
  color: #999;
  font-weight: normal;
  line-height: 14px;
}

.com-box .info .com-cont {
  font-size: 14px;
  color: #666;
  line-height: 28px;
  margin-top: 8px;
}
.com-box .info .info-span {
  margin-top: 20px;
  width: 100%;
}
.com-box .info .info-span span {
  float: right;
  margin-right: 20px;
  cursor: pointer;
}
.com-box .info .info-span span:hover {
  color: #ff9d00;
}
.com-box .info ul {
  clear: both;
  margin-left: 80px;
}
.com-box .info #replys li {
  display: none;
}
.com-box .info:hover #replys li {
  display: block;
}
.com-box {
  position: relative;
}
.block {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translate(-50%, 0);
}
</style>
