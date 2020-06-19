<template>
  <div v-if="visible" class="mulo-imgupload-shadow flex center center-line">
    <div class="mulo-imgupload">
      <div class="icon-close">
        <img src="./imgs/close.png" alt srcset />
      </div>
      <div class="tip-top">{{tipMsg}}</div>

      <div class="handle-menu"></div>

      <div class="header">
        <div class="menu flex">
          <div
            class="item"
            v-for="(li,index) in menu"
            @click="menuActive=index"
            :key="index"
            :class="{active:index==menuActive}"
          >{{li.title}}</div>
        </div>
      </div>
      <div v-if="menuActive==0" class="manage">
        <!-- 筛选栏目 -->
        <div class="header-filter flex space-between">
          <div class="filter flex">
            <el-date-picker
              v-model="filter.date"
              type="daterange"
              align="right"
              unlink-panels
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              value-format="yyyy-MM-dd"
              :picker-options="pickerOptions"
            ></el-date-picker>
            <el-button @click="loadImg(true)">查询</el-button>
          </div>

          <div v-if="!editMode" class="handle flex">
            <el-button @click="editMode=true">编辑</el-button>

            <!--    :on-preview="handlePreview"
              :on-remove="handleRemove"
            :before-remove="beforeRemove"-->
            <el-upload
              class="upload-demo"
              action="http://s.zoo.cn/party/upload/webuploader?_ajax=1&app=wr"
              multiple
              :show-file-list="false"
              :on-success="uploadSuccess"
              :on-exceed="handleExceed"
              :on-progress="progress"
              :file-list="fileList"
            >
              <el-button type="success">上传图片</el-button>
            </el-upload>
          </div>
          <div v-else class="handle flex">
            <el-button @click="editMode=false" type="danger">取消编辑</el-button>
            <el-button type="danger">删除</el-button>
          </div>
        </div>

        <div class="manage-main flex">
          <div class="categoty flex column">
            <div class="item category-add flex center-line">
              <img class="icon-add" src="./imgs/add.png" />
              <div>添加分类</div>
            </div>
            <div
              v-for="(li,index) in categoryList"
              :key="index"
              class="item flex center-line"
              @click="categoryActiveId = li.id"
              :class="{active:li.id==categoryActiveId}"
            >{{li.title}}</div>
          </div>
          <!-- 图片列表-无限滚动 -->
          <div
            class="imgs-container flex wrap start"
            v-infinite-scroll="loadImg"
            infinite-scroll-disabled="loadImgDisable"
          >
            <div
              v-for="(img,index) in imgs"
              :key="index"
              @click="selectImg(index)"
              class="item flex center center-line"
              :class="{active:img.__selected}"
            >
              <img :src="img.url" alt srcset />
            </div>
            <div class="emptydata">还木有图片,快上传一张吧~</div>
          </div>
        </div>
      </div>

      <div v-if="menuActive==1" class="cloud-imgs">云图库</div>
      <!--  -->
      <div class="footer flex end">
        <template v-if="!editMode">
          <el-button @click="selectSuccess" type="success">确定</el-button>
        </template>
        <template v-else></template>
      </div>
    </div>
  </div>
</template>

<script>
import "mulo-ui/lib/css/index.css";

//infinite-scroll
import {
  Button,
  DatePicker,
  Popover,
  Upload,
  InfiniteScroll,
  Message
} from "element-ui";

export default {
  components: {
    [Button.name]: Button,
    [DatePicker.name]: DatePicker,
    [Popover.name]: Popover,
    [Upload.name]: Upload
  },
  name: "mulo-imgupload",
  directives: {
    InfiniteScroll
  },
  watch: {
    //分类改变
    categoryActiveId(newValue, oldValue) {
      this.loadImg(true);
    }
  },
  computed: {
    tipMsg() {
      if (this.editMode) {
        return "点击图片勾选要删除的图片哒~";
      }
      return "点击选择一张图片";
    },
    categoryList() {
      return [{ id: 0, title: "全部" }, ...this.category];
    },
    loadImgDisable() {
      return this.loading || this.loadAll;
    }
  },
  props: {
    //显示右上角关闭按钮
    closeVisible: {
      type: Boolean,
      default: true
    },
    //可选择选择张数 - 最大
    num: {
      type: Number,
      default: 1
    },
    //最少选择张数
    minNum: {
      type: Number,
      default: 1
    }

    //
  },

  created() {
    this.loadImg();
  },
  data() {
    return {
      //加载参数
      loading: false,
      disable: false,
      loadAll: false,
      //查询参数
      page: 1,
      psize: 1,

      //操作状态
      editMode: false,
      visible: true,

      //页面数据
      fileList: [],
      menu: [{ title: "我的图片" }, { title: "云图库" }],
      menuActive: 0,
      imgs: [
        // {
        //   url: "http://qiniu1.loqh.cn/mulo-turntable-2.png",
        //   selected: false
        // }
      ],
      //选中的图片列表 id:selected
      imgSelecteds: {},
      //选择时间
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            }
          }
        ]
      },
      filter: {
        date: ""
      },
      // 分类
      category: [],
      // 0 全部, 选中的分类的id
      categoryActiveId: 0
    };
  },
  methods: {
    /**
     * uploadSuccess
     *
     * @todo 上传成功保存图片
     *
     */
    uploadSuccess(res) {
      /**
       * 图片更新的回调
       * @tudo 新保存的图片后,更新显示的列表
       * @param {ImgInfo} img 一张图片
       *
       */
      let callback = img => {
        console.log("增加图片", img);

        this.imgs.splice(0, 0, img);
      };

      console.log("img-add", res);
      this.$emit("img-add", {
        uploadResult: res,
        callback,
        category_id: this.categoryActiveId
      });
    },
    //选择图片
    selectImg(index) {
      //选择状态
      this.$set(this.imgs[index], "__selected", !this.imgs[index].__selected);
    },
    selectList() {},
    //修改分类数据
    setCategory(data) {
      this.category = data;
    },
    /**
     * @param {Boolean} reset 重置分页
     *
     */
    loadImg(reset) {
      if (reset === true) {
        this.page = 1;
        this.loadAll = false;
      }
      this.loading = true;
      this.emitLoadAction();
    },
    selectedImgs() {
      let imgs = [];
      this.imgs.forEach((li, index) => {
        if (li.__selected) {
          imgs.push(li);
        }
      });
      return imgs;
    },
    close() {
      this.visible=false;
    },
    progress(event, file, fileList){
      console.log(event.percent);
      console.log(event,file,fileList)
    },
    selectSuccess() {
      let imgs = this.selectedImgs();
      if (imgs.length < this.minNum) {
        Message(`最少选择${this.minNum}张图片~`);
        return;
      }
      this.$emit("success", {
        //多张图片
        imgs: imgs,
        //单张图片方便取到
        img: imgs[0],
      });
    },
    /**
     * 加载的图片列表
     * @param {ImgsResult} res 加载接口返回的数据
     */
    renderImgs(res) {
      if (res.code != 200) {
        this.loading = false;
        return;
      }

      let { page, list } = res.data;

      //是否加载所有
      if (list.length < this.psize) {
        this.loadAll = true;
      }

      if (page == 1) {
        this.imgs = list || [];
      } else {
        this.imgs = this.imgs.concat(list);
      }
      this.page++;
      this.loading = false;
    },
    //提交加载事件
    emitLoadAction() {
      this.$emit("load", {
        //活动分类id
        category_id: this.categoryActiveId,
        page: this.page,
        filter: this.filter
      });
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePreview(file) {
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 3 个文件，本次选择了 ${
          files.length
        } 个文件，共选择了 ${files.length + fileList.length} 个文件`
      );
    },
    name() {},
    //编辑分类
    saveCate() {}
  }
};
</script>
<style lang="scss">
$padding: 15px;
$activeColor: #3399ff;
.mulo-imgupload {
  width: 1080px;
  font-size: 14px;
  background-color: white;
  padding: $padding;
  position: relative;

  .header {
    .menu {
      padding: $padding;
      > .item {
        padding: 0 $padding;
        color: black;
        &.active {
          color: $activeColor;
        }
      }
    }
  }

  .manage {
    //筛选栏目
    .header-filter {
      margin-bottom: $padding;
      // 编辑 , 上传
      .handle {
      }
    }
    // 中间主体
    .manage-main {
      // 分类
      .categoty {
        min-width: 140px;
        > .item {
          width: 100%;
          height: 40px;
          &.active {
            background-color: white;
            color: $activeColor;
          }
        }
        .category-add {
          color: #55b4e5;
          .icon-add {
            width: 15px;
            height: 15px;
          }
        }
      }
      .imgs-container {
        padding: 0 0;
        width: 100%;
        height: 300px;
        overflow-y: scroll;

        > .item {
          width: 160px;
          height: 160px;
          background-color: #999;
          margin: 0 7px;
          margin-bottom: 14px;
          position: relative;

          overflow: hidden;
          &.active {
            color: red;
            &::after {
              content: "";
              position: absolute;
              left: 0;
              right: 0;
              top: 0;
              bottom: 0;
              background-color: rgba(0, 0, 0, 0.7);
              background-image: url(./imgs/tick-white.png);
              background-repeat: no-repeat;
              background-size: 50% 50%;
              background-position: center center;
            }
          }

          img {
            max-width: 100%;
            max-height: 100%;
          }
        }
        .emptydata {
          text-align: center;
          width: 100%;
          color: #999;
        }
      }
    }
  }
  .footer {
    // padding:  0;
    margin-top: $padding;
  }
  .btn {
    background-color: #3399ff;
    padding: $padding $padding;
    display: inline-block;
    text-align: center;
    line-height: 1;
    border-radius: 5px;
    color: white;
  }
  .icon-close {
    position: absolute;
    right: 20px;
    top: 20px;

    img {
      width: 20px;
      height: 20px;
    }
  }
  .tip-top {
    position: absolute;
    width: 100%;
    left: 0;
    top: 0;
    padding-top: 10px;
    text-align: center;
    color: #999;
  }
}
.mulo-imgupload-shadow {
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  position: fixed;
  background-color: rgba(0, 0, 0, 0.7);
}

.handle-menu {
  background-image: url(./imgs/menu.png);
  background-size: cover;
  width: 25px;
  height: 25px;
  position: absolute;
  bottom: 15px;
  left: 15px;
}
</style>