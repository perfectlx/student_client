<template>
  <div>
    <el-form>
      <el-form-item label="选择学期">
        <el-select v-model="term" placeholder="请选择学期">
          <el-option
            v-for="(item, index) in termList"
            :key="index"
            :label="item"
            :value="item"
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <el-card>
      <el-table :data="tableData" height="250" border style="width: 100%">
        <!-- 表格列配置和数据 -->
        <el-table-column
          fixed
          prop="cid"
          label="课程编号"
          width="150"
        ></el-table-column>
        <el-table-column
          prop="cname"
          label="课程名称"
          width="150"
        ></el-table-column>
        <el-table-column
          prop="tid"
          label="教师编号"
          width="150"
        ></el-table-column>
        <el-table-column
          prop="tname"
          label="教师名称"
          width="150"
        ></el-table-column>
        <el-table-column
          prop="grade"
          label="成绩"
          width="150"
        ></el-table-column>
      </el-table>
      <p>平均成绩：{{ avg.toFixed(2) }}</p>
      <el-pagination
        background
        layout="prev, pager, next"
        :total="total"
        :page-size="pageSize"
        @current-change="changePage"
      >
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  methods: {
    changePage(page) {
      page = page - 1;
      const that = this;
      let start = page * that.pageSize,
        end = that.pageSize * (page + 1);
      let length = that.tmpList.length;
      let ans = end < length ? end : length;
      that.tableData = that.tmpList.slice(start, ans);
    },
  },
  data() {
    return {
      tableData: null,
      pageSize: 10,
      total: null,
      tmpList: null,
      avg: 0,
      term: sessionStorage.getItem("currentTerm"),
      termList: null,
    };
  },
  created() {
    const that = this;
    axios.get("http://localhost:10086/SCT/findAllTerm").then(function (resp) {
      that.termList = resp.data;
    });
  },
  watch: {
    term: {
      handler(newTerm, oldTerm) {
        const sid = sessionStorage.getItem("sid");
        const that = this;
        axios
          .get("http://localhost:10086/SCT/findBySid/" + sid + "/" + newTerm)
          .then(function (resp) {
            that.tmpList = resp.data;
            that.total = resp.data.length;
            let start = 0,
              end = that.pageSize;
            let length = that.tmpList.length;
            let ans = end < length ? end : length;
            that.tableData = that.tmpList.slice(start, end);

            // 计算平均成绩
            let totalGrade = 0;
            for (let i = 0; i < that.tmpList.length; i++) {
              totalGrade += that.tmpList[i].grade;
            }
            if (that.tmpList.length > 0) {
              that.avg = totalGrade / that.tmpList.length;
            } else {
              that.avg = 0;
            }
          });
      },
      immediate: true,
    },
  },
};
</script>
