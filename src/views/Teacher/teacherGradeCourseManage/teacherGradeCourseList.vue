<template>
  <div>
    <el-table :data="tableData" border stripe style="width: 100%">
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
        fixed
        prop="sid"
        label="学号"
        width="100"
      ></el-table-column>
      <el-table-column
        prop="sname"
        label="学生姓名"
        width="100"
      ></el-table-column>
      <el-table-column prop="grade" label="成绩" width="100"></el-table-column>
      <el-table-column label="操作" width="100">
        <template slot-scope="scope">
          <el-button
            @click="editor(scope.row)"
            type="text"
            size="mini"
            icon="el-icon-edit"
            >编辑</el-button
          >
        </template>
        ></el-table-column
      >
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
      let ans = end < length ? end : length; //start和end索引，以确定应该根据pageSize在当前页上显示哪一部分数据。
      that.tableData = that.tmpList.slice(start, ans);
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
    },
    editor(row) {
      this.$router.push({
        path: "/editorGradeCourse",
        query: {
          cid: row.cid,
          tid: row.tid,
          sid: row.sid,
          term: row.term,
        },
      });
    },
  },
  data() {
    return {
      tableData: null,
      pageSize: 10,
      total: null,
      tmpList: null,
      avg: 0,
    };
  },
  props: {
    ruleForm: Object,
  },
  watch: {
    ruleForm: {
      handler(newRuleForm, oldRuleForm) {
        const that = this;
        that.tmpList = null;
        that.total = null;
        that.tableData = null;
        axios
          .post("http://localhost:10086/SCT/findBySearch", newRuleForm)
          .then(function (resp) {
            that.tmpList = resp.data;
            that.total = resp.data.length;
            let start = 0,
              end = that.pageSize;
            let length = that.tmpList.length;
            let ans = end < length ? end : length;
            that.tableData = that.tmpList.slice(start, ans);
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
      deep: true,
      immediate: true,
    },
  },
};
</script>
