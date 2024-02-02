<template>
  <div>
    <el-container>
      <el-main>
        <h1>学生主页</h1>
        <info-card></info-card>
        <el-card style="width: 100%; margin-top: 10px">
          <p>
            <i class="el-icon-s-home" style="margin-right: 18px"></i
            >项目名：学生管理系统
          </p>
          <el-button @click="fetchExcellentGrades" type="primary"
            >查看优秀科目</el-button
          >
          <el-button @click="fetchFailedGrades" type="danger"
            >查看挂科科目</el-button
          >
        </el-card>

        <div v-if="excellentCourses.length > 0">
          <h2>优秀科目</h2>
          <ul>
            <li v-for="course in excellentCourses" :key="course.id">
              {{ course.courseName }}: {{ course.grade }}
            </li>
          </ul>
        </div>

        <div v-if="failedCourses.length > 0">
          <h2>挂科科目</h2>
          <ul>
            <li v-for="course in failedCourses" :key="course.id">
              {{ course.courseName }}: {{ course.grade }}
            </li>
          </ul>
        </div>
        <logout></logout>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import Logout from "@/components/logout";
import InfoCard from "@/components/infoCard";
import axios from "axios";

export default {
  name: "home",
  components: { InfoCard, Logout },
  data() {
    return {
      excellentCourses: [],
      failedCourses: [],
    };
  },
  methods: {
    fetchExcellentGrades() {
      const sid = sessionStorage.getItem("sid");
      axios
        .get("http://localhost:10086/SCT/excellent-grades/" + sid)
        .then((response) => {
          this.excellentCourses = response.data;
          console.log(this.excellentCourses);
        })
        .catch((error) => {
          console.error("无法获取优秀成绩: " + error);
        });
    },

    fetchFailedGrades() {
      const sid = sessionStorage.getItem("sid");
      axios
        .get("http://localhost:10086/SCT/failed-grades/" + sid)
        .then((response) => {
          this.failedCourses = response.data;
        })
        .catch((error) => {
          console.error("无法获取挂科成绩: " + error);
        });
    },
  },
};
</script>
