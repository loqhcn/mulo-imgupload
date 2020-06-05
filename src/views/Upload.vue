<template>
  <div>
    <button @click="select">选择图片</button>

    <mulo-imgupload></mulo-imgupload>

  </div>
</template>

<script>
import Upload from "./../components/Upload";
console.log(Upload.name)
export default {
  components: {
    [Upload.name]: Upload
  },
  methods: {
    select() {}
  }
};
</script>

<style lang="scss" scoped>
</style>