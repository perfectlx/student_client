<template>
  <el-descriptions title="用户信息" :column="1" border>
    <el-descriptions-item label="用户 id " content-class-name="my-content">{{
      id
    }}</el-descriptions-item>
    <el-descriptions-item label="用户名" content-class-name="my-label">{{
      name
    }}</el-descriptions-item>
    <el-descriptions-item label="用户类型">
      <el-tag size="medium">{{ type }}</el-tag>
    </el-descriptions-item>
  </el-descriptions>
</template>
<script>
export default {
  name: "infoCard",
  data() {
    return {
      id:
        sessionStorage.getItem("type") === "teacher" ||
        sessionStorage.getItem("type") === "admin"
          ? sessionStorage.getItem("tid")
          : sessionStorage.getItem("sid"),
      name: sessionStorage.getItem("name"),
      type: sessionStorage.getItem("type"),
    };
  },
};
</script>

<style>
.my-label {
  background: #e1f3d8;
}

.my-content {
  background: #fde2e2;
}
</style>