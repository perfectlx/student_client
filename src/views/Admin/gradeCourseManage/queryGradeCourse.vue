<template>
  <div>
    <el-container>
      <el-main>
        <el-card>
          <el-form
            :inline="true"
            :model="ruleForm"
            :rules="rules"
            ref="ruleForm"
            label-width="140px"
            class="demo-ruleForm"
          >
            <el-form-item label="学号" prop="sid">
              <el-input v-model.number="ruleForm.sid"></el-input>
            </el-form-item>
            <el-form-item label="学生姓名" prop="sname">
              <el-input v-model="ruleForm.sname"></el-input>
            </el-form-item>
            <el-form-item label="模糊查询" prop="sFuzzy">
              <el-switch v-model="ruleForm.sFuzzy"></el-switch>
            </el-form-item>
            <el-form-item label="教师编号" prop="tid">
              <el-input v-model.number="ruleForm.tid"></el-input>
            </el-form-item>
            <el-form-item label="教师名称" prop="tname">
              <el-input v-model="ruleForm.tname"></el-input>
            </el-form-item>
            <el-form-item label="模糊查询" prop="tFuzzy">
              <el-switch v-model="ruleForm.tFuzzy"></el-switch>
            </el-form-item>
            <el-form-item label="课程编号" prop="cid">
              <el-input v-model.number="ruleForm.cid"></el-input>
            </el-form-item>
            <el-form-item label="课程名称" prop="cname">
              <el-input v-model="ruleForm.cname"></el-input>
            </el-form-item>
            <el-form-item label="模糊查询" prop="cFuzzy">
              <el-switch v-model="ruleForm.cFuzzy"></el-switch>
            </el-form-item>
            <el-form-item label="成绩下限" prop="lowBound">
              <el-input v-model.number="ruleForm.lowBound"></el-input>
            </el-form-item>
            <el-form-item label="成绩上限" prop="highBound">
              <el-input v-model.number="ruleForm.highBound"></el-input>
            </el-form-item>

            <el-form-item label="选择学期">
              <el-select v-model="ruleForm.term" placeholder="请选择学期">
                <el-option
                  v-for="(item, index) in termList"
                  :key="index"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>

            <div class="button-container">
              <el-button
                type="primary"
                @click="resetForm('ruleForm')"
                icon="el-icon-refresh"
                >重置</el-button
              >
              <el-button
                type="primary"
                @click="triggerFileInput"
                icon="el-icon-folder-add"
                class="Uploadgrade"
                >批量导入成绩</el-button
              >
              <input
                type="file"
                ref="fileInput"
                accept=".xlsx"
                @change="handleFileUpload"
                style="display: none"
              />
            </div>
          </el-form>
        </el-card>
        <el-card style="margin-top: 10px">
          <grade-course-list :rule-form="ruleForm"></grade-course-list>
        </el-card>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import GradeCourseList from "@/views/Admin/gradeCourseManage/gradeCourseList";
export default {
  components: { GradeCourseList },
  data() {
    return {
      termList: null,
      ruleForm: {
        sid: null,
        sname: null,
        sFuzzy: true,
        tid: null,
        tname: null,
        tFuzzy: true,
        cid: null,
        cname: null,
        cFuzzy: true,
        lowBound: null,
        highBound: null,
        term: sessionStorage.getItem("currentTerm"),
      },
      rules: {
        cid: [{ type: "number", message: "请输入正确数字！" }],
        tid: [{ type: "number", message: "请输入正确数字！" }],
        sid: [{ type: "number", message: "请输入正确数字！" }],
        cname: [],
        lowBound: [{ type: "number", message: "请输入正确数字！" }],
        highBound: [{ type: "number", message: "请输入正确数字！" }],
      },
    };
  },
  created() {
    const that = this;
    axios.get("http://localhost:10086/SCT/findAllTerm").then(function (resp) {
      that.termList = resp.data;
    });
  },
  methods: {
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    triggerFileInput() {
      // 通过JavaScript触发文件上传输入框的点击事件
      this.$refs.fileInput.click();
    },
    handleFileUpload(event) {
      this.selectedFile = this.$refs.fileInput.files[0];
      // 处理文件上传逻辑，可以访问event.target.files获取所选文件
      const selectedFiles = event.target.files;
      if (selectedFiles.length > 0) {
        // 执行文件处理操作
        const formData = new FormData();
        formData.append("file", this.selectedFile);

        axios
          .post("http://localhost:10086/SCT/upload", formData, {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          })
          .then((response) => {
            // 处理上传成功后的逻辑
            console.log(response.data);
          })
          .catch((error) => {
            // 处理上传失败的逻辑
            console.error(error);
          });
        console.log("所选文件:", selectedFiles[0]);
      }
    },
  },
};
</script>

<style scoped>
</style>
