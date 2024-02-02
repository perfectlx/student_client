<template>
  <div>
      <el-popconfirm
          confirm-button-text='确定'
          cancel-button-text='取消'
         icon="el-icon-s-opportunity"
          icon-color="red"
          title="确定退出登陆吗？"
          @confirm="out"
      >
        <el-button icon="el-icon-switch-button" slot="reference"  type="danger" style="margin-top: 10px" round @click>
          退出登录
        </el-button>
      </el-popconfirm>
  </div>
</template>

<script>
export default {
  name: "logout",
  methods: {
    out() {
      sessionStorage.removeItem("token")
      this.$router.push('/')
    }
  }
}
</script>

<style scoped>

</style>