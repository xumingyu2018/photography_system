<template>
  <div>
    <div v-for="stra in strategy" :key="stra.id">
      <img :src="stra.posterurl" alt class="jumbotron" />
      <div class="container mt-3">
        <h3 id="artTitle">{{ stra.name }}</h3>
        <!-- <img style="height:400px;width:600px" :src="stra.posterurl"> -->
        <h3 id="artTitle" style="font-size:1rem" v-html="stra.actor"></h3>
        <div id="title">
          <div id="h2-right">
            <div class="ext-r row" style="justify-content:space-around;">
              <div style="cursor:pointer;color:red;float:left;" @click="gosheyin(stra)">摄影师：{{stra.userinfo.username}}</div>
              <div style="cursor:pointer" @click="buynow(stra)">立即预订</div>
              <div style="cursor:pointer" @click="go(stra.userId)">
                <span>
                  <i class="fa fa-map-marker mr-2" aria-hidden="true">
                  {{ stra.isshow }}</i
                 >
                </span>
              </div>
              <div
                @click="updateCollectionNum(stra)"
                class="operation"
                id="operation"
                style="cursor:pointer"
                v-if="stra.ssStatus < 0 ? false : true"
              >
                <i class="el-icon-star-off" aria-hidden="true" id="icon"
                  >点赞 {{ stra.playinfo }}</i
                >
              </div>
             
            </div>
          </div>
        </div>
      </div>
      <div class="container">
        <div id="article" v-html="stra.ssInfo"></div>
      </div>
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
                  <img src="../assets/touxiang.png" />
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
              <textarea
                v-model="newcommentContent"
                placeholder="说点什么吧..."
                id="textarea"
              ></textarea>
            </div>
            <div v-if="isShow === false" id="readonly" class="ml-5 mb-5">
              <h3 style="color:red;text-align:center">
                由于您的当前用户状态不正常，已禁止评论功能
              </h3>
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
    <el-backtop :bottom="100"></el-backtop>
  </div>
</template>
<script>
import jwt_decode from "jwt-decode";

export default {
  name: "fvstrategy",
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
      iscollect:false
    };
  },
  created() {
    //获取传来的攻略类型和id
    var info = JSON.parse(sessionStorage.getItem("info")); //info=[type,id]
    this.info = info;
    console.log(info)
    this.iscollect = false
      this.getdatadetail()

      //筛选评论
      this.getcomment()
    
      this.getCollect()
      /**判断该用户是否点赞该攻略 */
     
  },
  methods: {
    gosheyin(obj){
      console.log(obj)
      this.$router.push(`/index/takephoner/${obj.wish}`);
    },
      // 获取详情
    getdatadetail(){
    this.$axios
      .post("/api/getGoodsById", {
        id: this.info.id
      })
      .then(res => {
        console.log(res.data);
        this.strategy = res.data;
      })
      .catch(err => {
        console.log("错误信息" + err);
      });
    },
    getCollect(){
      let tuser =JSON.parse(localStorage.getItem("tuser")) 
      let data = {
        userid: tuser.id ,
        goodsid:this.info.id
      }
      this.$axios
        .post("/api/queryCollectByGU", data)
        .then(res => {
          if (res.data.length) {
            this.iscollect = true
            $("#operation").addClass("operated");
            $("#icon").removeClass("el-icon-star-off");
            $("#icon").addClass("el-icon-star-on");
          }
        });
    },
    getcomment(){
      this.$axios
        .post("/api/queryCommentByGoodsId", {
          goodsid: this.info.id
        })
        .then(res => {
          this.discuss = res.data;
          this.allpages = res.data.length;
        })
        .catch(err => {
          console.log("错误信息" + err);
        });
    
    },
    //立即预订
    buynow(element){
      console.log(element)
      let userinfo = JSON.parse(localStorage.getItem("tuser"));
      let temp = []
      let data = {
        id: element.id,
        name: element.name,
        posterurl: element.posterurl,
        type: element.type,
        score: element.score,
        actor: element.actor,
        screen: '1',
        isshow: element.isshow,
        wish: userinfo.username,
        takeuserid: element.wish,
        ordertype: 1,
        playinfo: element.playinfo
      };
      temp.push(data);
      console.log(temp)
      this.$axios
				.post("/api/addOrder",  { jsonparm: temp }
				)
				.then(response => {
          this.$message({
            showClose: true,
            message: "预订成功！",
            type: "success"
          });
				})
				.catch(function(error) {
					console.log(error);
				});

        
    },
    //更新点赞数
    updateCollectionNum() {       
      if (localStorage.getItem("mytoken")) {
        let tuser =JSON.parse(localStorage.getItem("tuser")) 
        let data = {
          userid: tuser.id ,
          goodsid:this.info.id
        }
        if(!this.iscollect){
           this.$axios
            .post("/api/addCollect", data)
            .then(res => {
              $("#operation").addClass("operated");
              $("#icon").removeClass("el-icon-star-off");
              $("#icon").addClass("el-icon-star-on");
              this.iscollect = true
              this.$message({
                showClose: true,
                message: "点赞成功！",
                type: "success"
              });
      this.getdatadetail()
            });
        }else{
          this.$axios
          .post("/api/delCollect", data)
          .then(res => {
              $("#operation").removeClass("operated");
              $("#icon").removeClass("el-icon-star-on");
              $("#icon").addClass("el-icon-star-off");
              this.iscollect = false
              this.$message({
                showClose: true,
                message: "取消点赞成功！",
                type: "success"
              });
      this.getdatadetail()
          });
        }
      } else {
        this.$message({
          showClose: true,
          message: "亲，请先登录呦！",
          type: "warning"
        });
      }
    },
    getHeadPic(pic) {
      //给图片名加上服务器端访问路径
      let path = "";
      if (pic == null || pic == "" || pic == "headPic") {
        pic = "primaryHead.jpeg";
      }
      path = "http://localhost:3000/uploadHeadPic/" + pic;
      return path;
    },
    //获取头像
    getCoverPic(pic) {
      //给图片名加上服务器端访问路径
      if (pic == "cover" || pic == null) {
        pic = "primaryCover.jpg";
      }
      let path = "http://localhost:3001/coverPic/" + pic;
      console.log(path);
      return path;
    },
    //添加评论
    addComment() {
      if (localStorage.getItem("mytoken")) {
      let tuser =JSON.parse(localStorage.getItem("tuser")) 
        let data = {
					username: tuser.username ,
					content: this.newcommentContent,
					creattime:"",
					goodsid: this.info.id,
					info: ""
				}
				console.log(data)
				this.$axios
				.post("/api/addComent", data
				)
				.then(response => {
          console.log(response)
          this.$message({
            showClose: true,
            message: "评论成功",
            type: "warning"
          });
          this.getcomment()
          this.newcommentContent = ""
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
   
    //筛选回复
    selReply(commentId, index) {
      var _this = this;
      console.log(index);
      this.$axios
        .post("http://localhost:3000/operation/selreply", {
          // strategyId: this.info.id,
          // strategyType: this.info.type
          commentId: commentId
        })
        .then(res => {
          // console.log("筛选回复", res);
          this.replys = res.data.data;
          // this.commentId=commentId
        })
        .catch(err => {
          console.log("错误信息" + err);
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
    go(userId) {
      if (userId == this.userId) {
        this.$router.push("/index/userCenter");
      } else {
        sessionStorage.setItem("strategyuserId", userId);
      }
    },
    current_change: function(currentPage) {
      this.currentPage = currentPage;
    },
    getHeadPic(pic) {
      //给图片名加上服务器端访问路径
      let path = "";
      if (pic == null || pic == "" || pic == "headPic") {
        pic = "primaryHead.jpeg";
      }
      path = "http://localhost:3000/uploadHeadPic/" + pic;
      return path;
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
#article {
  margin: 2rem;
}
#artTitle {
  color: #ff9d00;
}
#readonly h3 {
  color: #666;
}
.card-body h5 {
  padding-left: 10px;
  margin-bottom: 15px;
}
.main-show p {
  padding-left: 10px;
  margin-top: 15px;
}

#title #h2-right .img-span:not(:last-child):hover {
  color: #ff9d00;
  cursor: pointer;
}
#title #h2-right .img-span:last-child:hover {
  color: red;
  cursor: pointer;
}
.con-top-left {
  margin-left: 10px;
}
.con-top-left p {
  margin-left: 15px;
  margin-top: 10px;
}
.con-top-left .route {
  margin: 15px 10px 10px;
}
.operation :hover {
  color: #ff9d00;
}
.operated {
  color: #ff9d00;
}

#title {
  height: 80px;
  line-height: 80px;
}

#title h2 {
  display: inline-block;
  width: 60%;
  font-family: 楷体;
}

#title > div {
  float: right;
  /* display: inline-block; */
  width: 40%;
}

.container-fluid .container p span {
  background-color: #ff9d00;
}

.container .con-top .con-top-left {
  display: inline-block;
  width: 55%;
}

.con-top-left .route a {
  color: #666;
}

.con-top-left .route a:hover {
  color: #ff9d00;
}

.con-top-left p {
  color: #666;
}

.container .con-top .con-top-right {
  float: right;
  border: 1px solid #eee;
}

.con-main {
  clear: both;
  margin-top: 40px;
}

.con-main .main-show {
  margin: 0 auto;
  position: relative;
}

.con-main .main-show .show-img {
  background-color: white;
  margin: 25px;
  float: left;
  width: 300px;
}

.con-main .main-show .show-img:hover {
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.con-main .main-show .show-img img {
  width: 300px;
  height: 200px;
}

.con-main .main-show .show-img .img-span {
  font-size: 24px;
  position: relative;
  top: -42px;
  left: 12px;
  color: white;
}

.con-main .main-show .show-img .d-txt {
  text-align: center;
  padding: 10px 10px;
  padding-top: 0;
}

.con-main .traffic {
  clear: both;
}

.con-main .foods {
  clear: both;
}

.con-main .rooms {
  clear: both;
}

.con-main .main-show .show-img .d-txt .p-left {
  text-align: left;
  height: 40px;
}
/* 评论区style */
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
  /* margin: 0 auto; */
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

.com-box {
  position: relative;
  border-top: 1px solid #e5e5e5;
  margin: 0 auto;
}
.contain {
  background-color: #eee;
  width: 1200px;
  margin: 0 auto;
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
.com-box .info {
  width: 1000px;
  float: left;
}
.com-box .info a {
  color: #ff9d00;
  cursor: pointer;
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

li {
  list-style: none;
}

.com-box li .img {
  overflow: hidden;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  float: left;
  cursor: pointer;
}
.com-box li .img img {
  width: 48px;
  height: 48px;
}

.com-box .info {
  width: 1000px;
  float: left;
}

.com-box .info h3 {
  font-size: 18px;
  color: #333;
  font-weight: normal;
  line-height: 28px;
}

.com-box .info h4 {
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

.com-cont {
  margin-left: 48px;
}
.block {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translate(-50%, 0);
}

/deep/ .sc-user-input--picker-wrapper{
  display: none;
}
</style>
