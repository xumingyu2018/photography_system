<template>
  <el-row type="flex" class="row-bg" justify="center">
    <el-col :span="22">
      <el-container>
        <el-header height class="mt-3">
          <h5>所查询到的摄影内容如下:</h5>
        </el-header>
        <el-main height>
          <el-row :gutter="40">
            <el-col :span="16">
              <div class="card-deck flex-column mb-5">
                <div v-if="show">
                  <div
                    class="card d-flex flex-row mb-2 shadow-sm p-3 bg-white rounded"
                    v-for="article in articles.slice(
                      (currentPage - 1) * pagesize,
                      currentPage * pagesize
                    )"
                    :key="article.strategyId"
                  >
                    <img
                      class="card-img-top"
                      :src="article.posterurl"
                      alt="Card image cap"
                      style="width: 14rem;height: 11rem;"
                    />
                    <div class="card-body">
                      <h5 class="card-title">
                        <a
                          href="#"
                          class="card-link"
                          @click="go(article.type, article.id)"
                          >{{ article.name }}</a
                        >
                      </h5>
                      <p
                        class="card-text overflow-hidden"
                        style="height: 4em;text-overflow:ellipsis;"
                        v-html="article.ssInfo"
                      ></p>
                      <p class="text-muted">
                        <i class="fa fa-map-marker mr-3" aria-hidden="true">{{
                          article.isshow
                        }}</i>
                        <i class="el-icon-star-off mr-2" style="float:right">{{
                          article.playinfo
                        }}</i>
                      </p>
                    </div>
                  </div>
                  <div class="block mt-5">
                    <el-pagination
                      :page-size="pagesize"
                      :pager-count="11"
                      layout="prev, pager, next"
                      :total="allpages"
                      @current-change="current_change"
                    ></el-pagination>
                  </div>
                </div>
                <div v-else>
                  <h1>暂无数据</h1>
                </div>
              </div>
            </el-col>
            <el-col :span="8">
              <div class="rightTitle">
                <h4>作品展览</h4>
              </div>
              <div class="rightPic">
                <div class="partPic">
                  <img
                    src="..//../assets/images/origin2 (3).jpg"
                    alt
                    width="200px"
                    height="110px"
                  />
                  婚纱高大上流
                </div>
              </div>
              <div class="rightPic">
                <div class="partPic">
                  <img
                    src="..//../assets/images/origin2 (18).jpg"
                    alt
                    width="200px"
                    height="110px"
                  />
                  黑白现代流
                </div>
              </div>
              <div class="rightPic">
                <div class="partPic">
                  <img
                    src="..//../assets/images/origin2 (19).jpg"
                    alt
                    width="200px"
                    height="110px"
                  />
                  欧风
                </div>
              </div>
            </el-col>
          </el-row>
        </el-main>
      </el-container>
    </el-col>
    <el-backtop :bottom="100"></el-backtop>
  </el-row>
</template>
<script>
export default {
  data() {
    return {
      keyWord: "",
      articles: [],
      show: false,
      pagesize: 5,
      currentPage: 1,
      allpages: ""
    };
  },
  created() {
    this.keyWord = this.$route.params.keyWord;
    this.$axios
      .post("/api/queryGoodsByName", { name: this.keyWord })
      .then(res => {
        if (res.data) {
          this.show = true;
          this.articles = res.data;
          this.allpages = res.data.length;
        } else {
          this.show = false;
        }
      })
      .catch(err => {
        console.log("错误信息" + err);
      })
  },
  methods: {
    getCoverPic(pic) {
      //给图片名加上服务器端访问路径
      if (pic == "cover" || pic == null) {
        pic = "primaryCover.jpg";
      }
      let path = "http://localhost:3000/coverPic/" + pic;
      return path;
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
    go(type, id) {
      console.log(type,id)
      var strategy = { type, id };
      var info = JSON.stringify(strategy);
      sessionStorage.setItem("info", info);
      this.$router.push("/index/fvstrategy");
    },
    current_change(currentPage) {
      this.currentPage = currentPage;
    }
  }
};
</script>
<style scoped>
.contain {
  width: 1200px;
  margin: 0 auto;
}

.total::after {
  content: "";
  display: block;
  clear: both;
}

.left {
  float: left;
  width: 900px;
}

.right {
  float: right;
  width: 200px;
}

li {
  list-style: none;
}

.pic {
  display: inline-block;
  margin: 0 auto;
}

h3 {
  font-size: 25px;
  font-family: 宋体;
  margin-bottom: -20px;
  display: block;
}

h4 {
  text-align: center;
}

.partPic {
  background-color: cornsilk;
  height: 135px;
  margin-top: 30px;
}
h1 {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #ccc;
}
.block {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translate(-50%, 0);
}
.card-body {
  padding-bottom: 0;
  width: 10rem;
  height: 5rem;
}
.text-muted {
  margin: 0;
  height: 1rem;
  line-height: 1rem;
}
.text-muted img {
  width: 20px;
  height: 20px;
}
.card {
  color: #666;
}
.card a {
  font-size: 20px;
  color: #333;
}
.card:hover a {
  color: #ff9d00;
}
</style>
