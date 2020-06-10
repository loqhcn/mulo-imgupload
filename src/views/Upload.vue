<template>
  <div>
    <button @click="select">选择图片</button>

    <mulo-imgupload
      ref="up"
      @on-select="onSelect"
      @load="loadImg"
      save-cate="saveCate"
      @img-add="imgAdd"
    ></mulo-imgupload>
  </div>
</template>

<script>
import Upload from "./../components/Upload";
import axios from "axios";
const http = axios.create({
  baseURL: "http://s.zoo.cn"
});
import { Message } from "element-ui";
console.log(Upload.name);
export default {
  components: {
    [Upload.name]: Upload
  },
  mounted() {
    //分类获取
    http.get("/party/upload/categories").then(res => {
      res = res.data;
      this.$refs.up.setCategory(res.data.list);
    });
  },
  created() {},
  methods: {
    onSelect(data) {
      console.log("选中图片", data);
    },
    /**
     * 新增一张图片
     *
     *
     */
    imgAdd(data) {
      let { uploadResult, category_id, callback } = data;

      http
        .post("/party/upload/addImg", {
          url: uploadResult.data.url,
          category_id: category_id
        })
        .then(res => {
          res = res.data;
          if (res.code != 200) {
            Message(res.msg);
            return;
          }
          callback(res.data.info);
        });
    },
    /**
     * 加载图片列表
     *
     * @param {Object} param 加载参数, -page -date -cate
     *
     */
    loadImg(param) {
      http.post("/party/upload/imgs", param).then(res => {
        res = res.data;
        this.$refs.up.renderImgs(res);
      });
      //
    },
    // 选择了图片
    select() {},
    // 保存分类
    saveCate() {}
    //
  }
};
</script>

<style lang="scss" scoped>
</style>