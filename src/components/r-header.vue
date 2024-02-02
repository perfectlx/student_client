<template>
  <div>
  
      <el-button type="text" icon="el-icon-switch-button" style="color: #333333; font-size: 18px" @click="out()">
        退出登录
      </el-button>
      <el-divider direction="vertical" style="color: #333333"></el-divider>
      <i class="el-icon-date" style="margin-right: 18px"></i>
      <span>第</span>{{ currentTerm }}学期</span>
      <el-divider direction="vertical"></el-divider>
      <i class="el-icon-s-custom" style="margin-right: 18px"></i>
      <span>{{ name }}</span>
  </div>
</template>

<script>
export default {
  name: "r-header",
  data() {
    return {
      name: null,
      currentTerm: null,
    };
  },
  created() {
    this.name = sessionStorage.getItem("name");
    this.currentTerm = sessionStorage.getItem("currentTerm");
  },
  methods: {
    out() {
      sessionStorage.clear();
      this.$router.push("/");
    },
  },
};
</script>

<style scoped>
</style>