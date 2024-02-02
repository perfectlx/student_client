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
            <el-form-item label="学生名" prop="sname">
              <el-input v-model="ruleForm.sname"></el-input>
            </el-form-item>
            <el-form-item label="模糊查询" prop="sFuzzy">
              <el-switch v-model="ruleForm.sFuzzy"></el-switch>
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
          <teacher-grade-course-list
            :rule-form="ruleForm"
          ></teacher-grade-course-list>
        </el-card>
      </el-main>
    </el-container>
  </div>
</template>
<script>
import GradeCourseList from "@/views/Admin/gradeCourseManage/gradeCourseList";
import TeacherGradeCourseList from "@/views/Teacher/teacherGradeCourseManage/teacherGradeCourseList";
import { Message } from "element-ui";
import * as XLSX from "xlsx";

export default {
  components: { TeacherGradeCourseList, GradeCourseList },
  data() {
    return {
      termList: null,
      ruleForm: {
        sid: null,
        sname: null,
        sFuzzy: true,
        tid: sessionStorage.getItem("tid"),
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
        cid: [{ type: "number", message: "必须是数字类型" }],
        tid: [{ type: "number", message: "必须是数字类型" }],
        sid: [{ type: "number", message: "必须是数字类型" }],
        cname: [],
        lowBound: [{ type: "number", message: "必须是数字类型" }],
        highBound: [{ type: "number", message: "必须是数字类型" }],
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
      this.$refs.fileInput.click();
    },
    validateFileSize(file) {
      if (file.size <= 200 * 1024) {
        return true;
      } else {
        alert("文件大小超过200KB，无法上传。");
        return false;
      }
    },

    handleFileUpload(event) {
      this.selectedFile = this.$refs.fileInput.files[0];
      const selectedFiles = event.target.files;
      if (selectedFiles.length > 0) {
        if (
          this.validateFileSize(this.selectedFile) &&
          this.validateFileContent(this.selectedFile)
        ) {
          const formData = new FormData();
          formData.append("file", this.selectedFile);

          axios
            .post("http://localhost:10086/SCT/upload", formData, {
              headers: { "Content-Type": "multipart/form-data" },
            })
            .then((response) => {
              console.log(response.data);
              this.showUploadSuccessMessage(); // 显示上传成功消息
            })
            .catch((error) => {
              console.error(error);
            });
        } else {
          console.log("文件不符合要求，请检查文件大小和内容。");
        }
      }
    },
    validateFileContent(file) {
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const data = e.target.result;
          console.log("Data from Excel file:", data); // 输出data变量
          const workbook = XLSX.read(data, { type: "binary" });
          const sheetName = workbook.SheetNames[0];
          const sheet = workbook.Sheets[sheetName];
          const range = XLSX.utils.decode_range(sheet["!ref"]);
          console.log("Range:", range);

          let valid = false; // 用于跟踪文件内容是否符合要求的标志变量

          for (let R = range.s.r; R <= range.e.r; ++R) {
            for (let C = range.s.c; C <= range.e.c; ++C) {
              const cellAddress = { r: R, c: C };
              const cellRef = XLSX.utils.encode_cell(cellAddress);
              const cell = sheet[cellRef];

              if (
                cell.v === "序号" ||
                cell.v === "学号" ||
                cell.v === "课程编号" ||
                cell.v === "教师编号" ||
                cell.v === "分数" ||
                cell.v === "学期"
              ) {
                valid = true; // 找到一个必需的字段
              }
            }
          }

          // 返回 valid，如果文件内容符合要求则为 true，否则为 false
          return valid;
        };
        reader.readAsBinaryString(file);
      }
      return false;
    },
    showUploadSuccessMessage() {
      Message({
        message: "文件上传成功",
        type: "success",
      });
    },
  },
};
</script>
