<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>重金属污染聚类溯源</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 计算每种金属与管控制（环境标准值）的比值。用于划分污染风险和进行环境质量评估。
        <br />
      </p>
      <div style="float: left;line-height: 1.8em;">
        <strong>输入：</strong>
      </div>
      <div style="float:left;margin-bottom: 20px;line-height: 1.8em;">
        1. 武清区的采样点数据：位置信息、土壤金属含量。
        <br />2. 武清区的土壤背景值。
      </div>
    </template>

    <div class="map">
      <el-tabs class="mytabs" type="border-card" style v-model="activeName">
        <el-tab-pane label="相关性矩阵" name="first">
          <table class="matrix">
            <thead>
              <tr>
                <th colspan="9">相关系数分析结果表</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>相关系数</td>
                <td v-for="metal in metals" v-bind:key="metal" v-text="metal" />
              </tr>
              <tr v-for="(metal, index) in metals" v-bind:key="10*index">
                <td v-text="metal" />
                <td
                  v-for="(text,i) in matrix[index]"
                  :key="i"
                  v-text="text"
                  :class="{ high: high(matrix[index][i]), higher: higher(matrix[index][i])}"
                />
              </tr>
            </tbody>
          </table>
        </el-tab-pane>
        <el-tab-pane label="图表" name="second">2</el-tab-pane>
        <el-tab-pane label="详情" name="third">3</el-tab-pane>
      </el-tabs>
    </div>
  </my-content>
</template>

<script>
import MyContent from "@/layout/content";
export default {
  components: { MyContent },
  data() {
    return {
      activeName: "first",
      metals: ["Cr", "Ni", "Cu", "Zn", "As", "Cd", "Hg", "Pb"],
      matrix: []
    };
  },
  methods: {
    getInitData(data) {
      let vm = this;
      let tableData = data.tableData;
      let tableHeader = data.tableHeader;
      const n = tableData.length;
      const m = vm.metals.length;

      // 每个样本（X）的 金属含量
      let Dnm = [];
      for (let data of tableData) {
        let tmp = [];
        for (let metal of vm.metals) {
          tmp.push(data[metal]);
        }
        Dnm.push(tmp);
      }
      // x，y score 归一二维数组
      let Sij = [];
      const Sum = (n * (n - 1)) / 2;
      for (let i = 0; i < m; ++i) Sij.push([]);
      for (let i = 0; i < m; ++i) {
        for (let j = 0; j < m; ++j) {
          if (i >= j) {
            Sij[i][j] = "-";
            continue;
          }
          let score = 0;
          for (let k = 0; k < n; ++k) {
            for (let l = k + 1; l < n; ++l) {
              if (
                (Dnm[k][i] > Dnm[l][i] && Dnm[k][j] > Dnm[l][j]) ||
                (Dnm[k][i] < Dnm[l][i] && Dnm[k][j] < Dnm[l][j])
              ) {
                score = score + 1;
              }
            }
          }
          Sij[i].push(((score / Sum) * 100).toFixed(2) + "%");
        }
      }
      vm.matrix = Sij;
    },
    higher(a) {
      return Number(a.slice(0, -1)) > 80;
    },
    high(a) {
      const val = Number(a.slice(0, -1));
      return val < 80 && val > 70;
    }
  }
};
</script>

<style scoped>
.mytabs /deep/ .el-tabs__item {
  width: 200px;
  text-align: center;
}
/*.clearfix*/
.my-card {
  margin-bottom: 10px;
}
#upload-card /deep/ .el-card {
  padding: 0px !important;
}
.my-table /deep/ .cell {
  text-align: center;
}
.my-table /deep/ table {
  margin: auto;
}
.my-table /deep/ .el-table__empty-block {
  display: none;
}
.my-table /deep/ tbody tr:first-child {
  font-weight: bold;
}

.matrix {
  color: #303133;
  margin: 0 auto;
  border-spacing: 30px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}

.high {
  font-weight: bold;
  color: orangered;
}

.higher {
  font-weight: 900;
  color: red;
}
</style>