<template>
  <div class="teacher-student-grades-chart">
    <div ref="chart" class="chart-container"></div>
    <div>
      <el-card>
        <el-form
          :inline="true"
          :model="ruleForm"
          ref="ruleForm"
          label-width="120px"
          class="demo-ruleForm"
        >
          <el-form-item label="选择学期">
            <el-select
              v-model="ruleForm.term"
              placeholder="请选择学期"
              @change="fetchStudentData"
            >
              <el-option
                v-for="(item, index) in termList"
                :key="index"
                :label="item"
                :value="item"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-card>
    </div>
  </div>
</template>

<script>
import * as echarts from "echarts";
import axios from "axios";

export default {
  data() {
    return {
      termList: null,
      ruleForm: {
        sid: null,
        sname: null,
        tid: sessionStorage.getItem("tid"),
        tname: null,
        cid: null,
        cname: null,
        term: sessionStorage.getItem("currentTerm"),
      },
      chart: null,
      teacherId: 1, // Replace with the actual teacher ID
      studentData: [],
    };
  },
  created() {
    const that = this;
    axios.get("http://localhost:10086/SCT/findAllTerm").then(function (resp) {
      that.termList = resp.data;
    });
  },
  methods: {
    fetchStudentData() {
      if (this.chart) {
        this.chart.dispose();
      }

      axios
        .post("http://localhost:10086/SCT/findBySearch", this.ruleForm)
        .then((response) => {
          this.studentData = response.data;
          console.log(this.studentData);
          const studentNames = this.studentData.map((student) => student.sname);
          const studentScores = this.studentData.map(
            (student) => student.grade
          );

          const options = {
            title: {
              text: "我教的学生成绩分布",
            },
            xAxis: {
              type: "category",
              data: studentNames,
            },
            yAxis: {
              type: "value",
            },
            series: [
              {
                data: studentScores,
                type: "bar",
                label: {
                  show: true,
                  position: "top",
                },
              },
            ],
          };

          this.$nextTick(() => {
            this.chart = echarts.init(this.$refs.chart);
            this.chart.setOption(options);
          });
        })
        .catch((error) => {
          console.error("Failed to fetch student data: " + error);
        });
    },
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  },
};
</script>

<style scoped>
.teacher-student-grades-chart {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

.chart-container {
  height: 400px;
}
</style>