<template>
  <div class="teacher-students-grades-chart">
    <div ref="chart" class="chart-container"></div>
    <el-card>
      <el-form
        :model="ruleForm"
        ref="ruleForm"
        label-width="120px"
        class="demo-ruleForm"
      >
        <el-form-item label="选择学期">
          <el-select
            v-model="ruleForm.term"
            placeholder="请选择学期"
            @change="fetchTeacherData"
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
</template>

<script>
import * as echarts from "echarts";
import axios from "axios";

export default {
  data() {
    return {
      termList: null,
      ruleForm: {
        term: sessionStorage.getItem("currentTerm"),
      },
      chart: null,
      teacherData: [],
    };
  },
  created() {
    const that = this;
    axios.get("http://localhost:10086/SCT/findAllTerm").then(function (resp) {
      that.termList = resp.data;
    });
  },
  methods: {
    fetchTeacherData() {
      if (this.chart) {
        this.chart.dispose();
      }

      axios
        .post("http://localhost:10086/SCT/findBySearch", this.ruleForm)
        .then((response) => {
          this.teacherData = response.data;
          // 创建一个数组，用于存储老师名字和课程名字的对应信息
          const teacherCourseInfo = [];
          this.teacherData.forEach((student) => {
            const teacher = student.tname;
            const course = student.cname;
            teacherCourseInfo.push({ teacher, course });
          });

          // 遍历老师数据，根据不同科目计算平均分
          const subjects = {}; // 用于存储不同科目的成绩总和和计数
          this.teacherData.forEach((student) => {
            const subject = student.cname;
            const grade = student.grade;
            if (!subjects[subject]) {
              subjects[subject] = { total: 0, count: 0 };
            }
            subjects[subject].total += grade;
            subjects[subject].count++;
          });

          // 计算平均分
          const subjectNames = Object.keys(subjects);
          const averages = subjectNames.map((subject) => {
            const total = subjects[subject].total;
            const count = subjects[subject].count;
            return (total / count).toFixed(2);
          });

          const options = {
            title: {
              text: "不同科目平均成绩",
            },
            xAxis: {
              type: "category",
              data: subjectNames,
              axisLabel: {
                interval: 0, // 如果希望横坐标全部显示
                rotate: 45, // 逆时针旋转45度
              },
            },
            yAxis: {
              type: "value",
            },
            series: [
              {
                data: averages,
                type: "bar",
                label: {
                  show: true,
                  position: "top",
                  formatter: (params) => {
                    const dataIndex = params.dataIndex;
                    const teacher = teacherCourseInfo[dataIndex].teacher;
                    const course = teacherCourseInfo[dataIndex].course;
                    return `${teacher}-${course}: ${params.value}`;
                  },
                },
                itemStyle: {
                  color: {
                    type: "linear",
                    x: 0, // 渐变色起点的 x 坐标
                    y: 0, // 渐变色起点的 y 坐标
                    x2: 0, // 渐变色终点的 x 坐标
                    y2: 1, // 渐变色终点的 y 坐标
                    colorStops: [
                      { offset: 0, color: "rgba(89, 194, 230, 1)" }, // 起始颜色，这里使用了淡蓝色
                      { offset: 1, color: "rgba(100, 255, 200, 1)" }, // 终止颜色
                    ],
                    global: false, // 缺省为 false
                  },
                },
              },
            ],
            dataZoom: [
              {
                type: "inside", // 内部缩放
                xAxisIndex: [0],
              },
            ],
            toolbox: {
              feature: {
                dataZoom: {
                  yAxisIndex: "none",
                },
                restore: {},
                saveAsImage: {},
              },
            },
          };

          this.$nextTick(() => {
            this.chart = echarts.init(this.$refs.chart);
            this.chart.setOption(options);
          });
        })
        .catch((error) => {
          console.error("Failed to fetch teacher data: " + error);
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
.teacher-students-grades-chart {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.chart-container {
  height: 600px; /* 增大图表容器的高度 */
  border: 1px solid #ccc; /* 添加边框以确保容器位置可见 */
  box-shadow: 0 0 5px #aaa; /* 添加阴影以突出图表容器 */
}
</style>
