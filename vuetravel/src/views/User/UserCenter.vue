<template>
  <div class="person">
    <div class="jumbotron jumbotron-fluid" id="topPic">
      <div class="container">
        <h1 class="text-light mt-5">摄影约拍札记</h1>
        <br />
        <h3 class="text-light">更懂世界，更懂你！！！</h3>
      </div>
    </div>

    <div class="container">
      <div class="row mb-5">
        <!-- 左侧边栏 -->
        <div
          class="col-lg-3 d-flex flex-column align-items-center d-md-none d-lg-block border-right"
        >
          <!-- 圆形头像 -->
          <div class="mt-3 float-right mr-5 text-center" style="width:10rem;">
            <h4 style="color:#ff9d00">{{ userInfo[0].userName }}</h4>
          </div>
          <div id="relations" class="mt-3 float-right">
            <img src="..//../assets/images/origin2 (4).jpg" style="width:100%">
          </div>
        </div>
        <!-- 中间部分 -->
        <div class="col-md-12 col-lg-9" id="content">
          <UserCenterMiddle v-if="show"></UserCenterMiddle>
          <List v-else></List>
        </div>
        <!-- 右侧导航 -->
        <el-backtop :bottom="100"></el-backtop>
      </div>
    </div>
  </div>
</template>
<script>
import UserCenterMiddle from "./children/UserCenter_middle";
import List from "./children/List";

export default {
  name: "UserCenter",
  components: {
    UserCenterMiddle,
    List
  },
  data() {
    return {
      userInfo: [
        {
          headPic: ""
        }
      ],
      fans: [],
      attentions: [],
      show: true
    };
  },
  created() {
   
    $(function() {
      $('[data-toggle="popover"]').popover({
        container: "body"
      });
      $('input[type="file"]').change(function() {
        var fread = new FileReader();
        fread.onload = function(e) {
          $(".rounded-circle").css(
            "background-image",
            "url(" + e.target.result + ")"
          );
        };
        fread.readAsDataURL(this.files[0]);
      });
    });
  },
  methods: {
    getHeadPic(pic) {
      //给图片名加上服务器端访问路径
      let path = "";
      if (pic == null || pic == "" || pic == "headPic") {
        pic = "primaryHead.jpeg";
      }
      path = "http://localhost:3000/uploadHeadPic/" + pic;
      return path;
    },
    fanslist() {
      this.show = false;
      console.log(this.userInfo[0].userId);
      sessionStorage.setItem("strategyuserId", this.userInfo[0].userId);
      var fanslist = JSON.stringify(this.fans);
      var attentions = JSON.stringify(this.attentions);
      sessionStorage.setItem("fanslist", fanslist);
      sessionStorage.setItem("attentions", attentions);
    }
  }
};
</script>
<style scoped>
.jumbotron {
  background: url("../../assets/bgPic/indexPic1.jpg") no-repeat center center;
  background-size: cover;
}
.rounded-circle {
  background-repeat: no-repeat;
  background-position: center center;
  background-size: cover;
}

#headPic {
  height: 10rem;
  width: 10rem;
}

#headPic img {
  height: 10rem;
  width: 10rem;
}

#relations ul {
  list-style: none;
  width: 10rem;
  padding: 0;
  cursor: pointer;
}
#relations ul li {
  float: left;
  height: 3em;
  line-height: 3em;
  width: 80px;
}
#relations ul li span {
  color: #ff9d00;
}
/* #headPic:hover::before {
  content: "";
  display: block;
  width: 10rem;
  height: 10rem;
  border-radius: 75px;
  background: rgba(0, 0, 0, 0.5) url("../../assets/camera.png") 50% no-repeat;
  cursor: pointer;
} */

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
