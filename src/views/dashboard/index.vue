<template>
  <el-container>
    <el-main>
      <el-row>
        <el-card>
          <search-table :headerData="headerData" :tableData="tableData" />
        </el-card>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
import SearchTable from "@/components/SearchTable/index";

import { MultivariateLinearRegression as MLR } from "ml";

const x = [[0, 0], [1, 2], [2, 3], [3, 4]];
// Y0 = X0 * 2, Y1 = X1 * 2, Y2 = X0 + X1
const y = [[0, 0, 0], [2, 4, 3], [4, 6, 5], [6, 8, 7]];
const mlr = new MLR(x, y);
console.log(mlr.predict([3, 3]));
// [6, 6, 6]

export default {
  name: "Dashboard",
  components: { SearchTable },
  data() {
    return {
      headerData: [
        {
          prop: "date",
          label: "时间",
          sortable: true,
          searchable: false
        },
        {
          prop: "name",
          label: "姓名"
        },
        {
          prop: "address",
          label: "地址"
        }
      ],
      tableData: [
        {
          date: "2016-05-03",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-02",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-04",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-01",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-08",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-06",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-07",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-03",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-02",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-04",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-01",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-08",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-06",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-07",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        }
      ]
    };
  }
};
</script>

<style lang="scss" scoped>
</style>
