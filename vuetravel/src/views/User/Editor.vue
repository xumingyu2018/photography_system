<template>
  <div>
    <!-- 封面 -->
    <div class="jumbotron">
      <el-container class>
        <el-main>
          <el-row>
            <!-- 表单 -->
            <el-form
              :model="articleForm"
              :rules="rules"
              ref="ruleForm"
              class="demo-ruleForm"
            >
              <!--elementui的上传图片的upload组件-->
              <el-row :gutter="20">
                <el-col :span="12">
                  <div class="mt-3">
                    <h4 class="display-5 text-right">设置效果封面</h4>
                    <p class="lead text-right mt-5">
                      图片建议选择尺寸大于1680px的高清大图，如相机原图
                    </p>
                  </div>
                </el-col>
                <el-col :span="12">
                   <el-upload
                    class="avatar-uploader"
                    ref="upload"
                    :action="local"
                    :show-file-list="false"
                    :on-success = "uploadsucess"
                    :on-progress="uploadAva"
                    :data="uploadData"
                    name="file">
                    <i  style="height:50px;width:50px;font-size:60px" class="el-icon-plus"></i>
                  </el-upload>
                </el-col>
              </el-row>
              <el-col :span="12" :offset="6">
                <el-row :gutter="40">
                  <el-col :span="7">
                    <el-form-item
                      label="摄影地区"
                      class="text-right"
                      prop="type"
                    >
                      <el-select
                        v-model="articleForm.type"
                        placeholder="请选择地区"
                      >
                        <el-option
                          v-for="type in types"
                          :key="type.value"
                          :label="type.label"
                          :value="type.value"
                        ></el-option>
                      </el-select>
                    </el-form-item>
                    <el-form-item label="标题" class="text-right" prop="title">
                      <el-input
                        v-model="articleForm.title"
                        name="title"
                        style="width:200px;"
                      ></el-input>
                    </el-form-item>
                    <el-form-item
                      label="城市名称"
                      class="text-right"
                      prop="cityName"
                    >
                      <el-input
                        v-model="articleForm.cityName"
                        name="cityName"
                        style="width:200px;"
                      ></el-input>
                    </el-form-item>
                  </el-col>
                </el-row>
                <!-- 提交按钮 -->
                <el-row>
                  <el-col :offset="3">
                    <el-form-item>
                      <el-button @click="resetForm('ruleForm')">重置</el-button>
                    </el-form-item>
                  </el-col>
                </el-row>
              </el-col>
            </el-form>
          </el-row>
        </el-main>
      </el-container>
    </div>
    <div class="container">
      <div class="row">
        <!-- 富文本框 -->
        <div class="col-md-12">
          <div class="clearfix mb-5">
            <!-- 富文本编辑框 -->
            <div id="editorMenu"></div>
            <div id="editor" class="mt-3" style="z-index:1"></div>
          </div>
          <!-- 发表按钮 -->
          <button
            type="button"
            class="btn btn-outline-primary float-right mt-3 mb-5 mr-5"
            :plain="true"
            @click="upload('ruleForm')"
          >
            发表约拍单
          </button>
        </div>
        <!-- 返回顶部 -->
        <el-backtop :bottom="200"></el-backtop>
      </div>
    </div>
  </div>
</template>
<script>
import E from "wangeditor";
export default {
  data() {
    return {
      local: '/api/cover',
      imageUrl: '',
      imageurl2:"",
      phoneEditor: "",
      name: "",
      uploadData: {
        userId: localStorage.getItem('userId')
      },
      fileList: [],
      articleForm: {
        type: "",
        cover: "",
        title: "",
        cityName: "",
        dayNum: "",
        season: "",
        crowdType: ""
      },
      params: "", // 表单最后提交的参数对象
      types: [
        {
          value: "国内",
          label: "国内"
        },
        {
          value: "日本",
          label: "日本"
        },
        {
          value: "欧洲",
          label: "欧洲"
        },
        {
          value: "海岛",
          label: "海岛"
        },
        {
          value: "泰国",
          label: "泰国"
        }
      ],
      days: [
        {
          value: "1",
          label: "一日游"
        },
        {
          value: "2",
          label: "二日游"
        },
        {
          value: "3",
          label: "三日游"
        },
        {
          value: "4",
          label: "四日游"
        },
        {
          value: "5",
          label: "五日游"
        },
        {
          value: "7",
          label: "七日游"
        },
        {
          value: "0",
          label: "多日游"
        }
      ],
      seasons: [
        {
          value: "春季",
          label: "春季"
        },
        {
          value: "夏季",
          label: "夏季"
        },
        {
          value: "秋季",
          label: "秋季"
        },
        {
          value: "冬季",
          label: "冬季"
        }
      ],
      crowdTypes: [
        {
          value: "活力行",
          label: "活力行"
        },
        {
          value: "夕阳游",
          label: "夕阳游"
        },
        {
          value: "亲子游",
          label: "亲子游"
        },
        {
          value: "情侣游",
          label: "情侣游"
        },
        {
          value: "蜜月行",
          label: "蜜月行"
        },
        {
          value: "伙伴游",
          label: "伙伴游"
        }
      ],
      rules: {
        type: [
          { required: true, message: "请选择攻略类型", trigger: "change" }
        ],
        title: [
          { required: true, message: "请输入标题", trigger: "blur" },
          { min: 1, max: 80, message: "长度在 1 到 80 个字符", trigger: "blur" }
        ],
        cityName: [
          { required: true, message: "请输入城市名称", trigger: "blur" },
          { min: 1, max: 16, message: "长度在 1 到 16 个字符", trigger: "blur" }
        ],
        dayNum: [
          { required: true, message: "请选择几日游", trigger: "change" }
        ],
        season: [
          { required: true, message: "请选择适宜季节", trigger: "change" }
        ],
        crowdType: [
          { required: true, message: "请选择面向人群", trigger: "change" }
        ]
      }
    };
  },
  created() {
    console.log("Asd")
    var editor = null;
  },
  methods: {
    uploadAva(response, file, fileList) {
      this.imageUrl = URL.createObjectURL(file.raw);
      $(".jumbotron").css(
        "background-image",
        "url(" + this.imageUrl + ")"
      );
    },
    uploadsucess(response, file, fileList){
      this.imageurl2 = response
      console.log(this.imageurl2)
    },
    // 重置表单按钮
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    //当上传文件组件submit之前触发执行
    beforeupload(file) {
      // 准备表单上传需要的参数对象
      console.log(file)
      this.param = new FormData();
      this.params.append(`cover`, file);
      return true;
    },
    // 设置保存发送后台数据事件
    upload(formName) {
      let tuser =JSON.parse(localStorage.getItem("tuser")) 
      if(!localStorage.getItem("mytoken")){
         this.$message({
            showClose: true,
            message: "用户未登录",
            type: "success"
          });
          return;
      }
      let _this = this;
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.params = new FormData();
          this.params.append("article", this.phoneEditor.txt.html());
          this.params.append("type", this.articleForm.type);
          this.params.append("title", this.articleForm.title);
          this.params.append("cityName", this.articleForm.cityName);
          this.params.append("dayNum", this.articleForm.dayNum);
          this.params.append("season", this.articleForm.season);
          this.params.append("crowdType", this.articleForm.crowdType);
          let data = {
            rowname:this.articleForm.title,
            content:this.phoneEditor.txt.html(),
            place: this.articleForm.type,
            address:this.articleForm.cityName,
            prCollectionNum: 1,
            userid:tuser.id,
            url:this.imageurl2
          }
          console.log(data)
          let thatt =this
          // this.$refs.upload.submit();
          //设置提交请求头，适用于上传文件
         this.$axios
          .post("/api/addrow",data)
          .then(res => {
               this.$alert('恭喜你，发布成功！', '温馨提示', {
              confirmButtonText: '确定',
              callback: action => {
                  
                  }
            });
           
          })
          .catch(err => {
            console.log("错误信息" + err);
          })
          
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    }
  },
  mounted() {
    // wangeditor
    this.phoneEditor = new E("#editorMenu", "#editor");
    // 上传图片到服务器，base64形式
    this.phoneEditor.customConfig.uploadImgShowBase64 = true;
    this.phoneEditor.customConfig.uploadImgServer = "/api/cover2/";
    this.phoneEditor.customConfig.uploadFileName = 'files'
    // 隐藏网络图片
    this.phoneEditor.customConfig.showLinkImg = true;
    // 创建一个富文本编辑器
    this.phoneEditor.create();
    // 富文本内容
    this.phoneEditor.txt.html();
  }
};
</script>
<style scoped>
#editor{
  z-index: 100 !important;
}
.el-message-box__wrapper{
  z-index: 999999999;
}
.el-message-box{
  z-index: 99999999;
}
.jumbotron {
  background-image: url("../../assets/bgPic/editorCover.jpg");
  background-repeat: no-repeat;
  background-position: center center;
  background-size: cover;
}

#editor {
  height: 1000px;
  /* height: auto; */
  width: auto;
  border: 1px solid #ccc;
}
.w-e-toolbar {
  background-color: #f1f1f1;
  border: 1px solid #ccc;
}

.el-row {
  margin-bottom: 20px;
}
.el-col {
  border-radius: 4px;
}
.bg-purple-dark {
  background: #99a9bf;
}
.bg-purple {
  background: #d3dce6;
}
.bg-purple-light {
  background: #e5e9f2;
}
.grid-content {
  border-radius: 4px;
  min-height: 36px;
}
.row-bg {
  padding: 10px 0;
  background-color: #f9fafc;
}
</style>
