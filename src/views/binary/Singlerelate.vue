<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>土壤作物相关分析</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 相关分析是定量描述和分析两个或多个变量观测值之间相关性程度的一种统计方法。相关性分析可以通过相关系数确定两变量的关系程度，若相关系数为正则说明两变量之间呈正相关关系，对应土壤和作物重金属含量之间表现为协同作用，相反，则呈负相关关系，对应土壤和作物重金属含量之间表现为拮抗作用。
        <br />
      </p>
      <div style="float: left;line-height: 1.8em;">
        <p>
          <strong>输入：</strong>1. 采样点作物数据：包括pH、各种金属含量、经纬度等地理信息。2. 采样点土壤数据：包括pH、各种金属含量、经纬度等地理信息。
        </p>
        <p>
          <strong>输出：</strong>相关系数分析结果表
        </p>
      </div>
    </template>
    <div label="相关性矩阵" name="first">
      <table class="matrix">
        <thead>
          <tr>
            <th colspan="9">相关系数分析结果表</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>相关系数</td>
            <td v-for="metal in metals" v-bind:key="metal" v-text="metal+'/植'" />
          </tr>
          <tr v-for="(metal, index) in metals" v-bind:key="10*index">
            <td v-text="metal+'/土'" />
            <td
              v-for="(text,i) in matrix[index]"
              :key="i"
              v-text="text"
              :class="{ high: high(matrix[index][i]), higher: higher(matrix[index][i])}"
            />
          </tr>
        </tbody>
      </table>
    </div>
  </my-content>
</template>

<script>
import MyContent from "@/layout/content";
import SearchTable from "@/components/SearchTable/index";
import standardDeviation from "ml-array-standard-deviation";
import mean from "ml-array-mean";
export default {
  components: { MyContent, SearchTable },
  data() {
    return {
      metals: ["Cr", "Ni", "Cu", "Zn", "As", "Cd", "Pb"],
      matrix: []
    };
  },
  methods: {
    getInitData(data) {
      let vm = this;
      let tableData = data.tableData;
      let tableHeader = data.tableHeader;
      const sampleLength = tableData.length;
      console.log(data.tableData);
      let soilData = [];
      let cropData = [];
      let matrix = [];
      for (let metal of vm.metals) {
        let soilTemp = [];
        let cropTemp = [];
        for (let d of data.tableData) {
          soilTemp.push(d[metal + "土"]);
          cropTemp.push(d[metal + "植"]);
        }
        soilData.push(soilTemp);
        cropData.push(cropTemp);
      }
      console.log(soilData);
      console.log(cropData);
      let cropVars = [];
      let soilVars = [];
      let cropMean = [];
      let soilMean = [];
      cropData.forEach(e => {
        cropVars.push(standardDeviation(e));
        cropMean.push(mean(e));
      });
      soilData.forEach(e => {
        soilVars.push(standardDeviation(e));
        soilMean.push(mean(e));
      });
      console.log(cropMean);
      console.log(cropVars);
      for (let i = 0; i < soilData.length; i++) {
        matrix.push([]);
        for (let j = 0; j < cropData.length; j++) {
          // if(i > j){
          //   matrix[i].push('-');
          //   continue;
          // }
          let sum = 0;
          for (let m = 0; m < sampleLength; m++) {
            sum +=
              (soilData[i][m] - soilMean[i]) * (cropData[j][m] - cropMean[j]);
          }
          matrix[i].push(
            (sum / (sampleLength - 1) / (cropVars[j] * soilVars[i])).toFixed(2)
          );
        }
        console.log(this.matrix);
        vm.matrix = matrix;
      }
    },
    higher(a) {
      return Math.abs(Number(a)) > 0.4;
    },
    high(a) {
      const val = Math.abs(Number(a));
      return val < 0.4 && val > 0.3;
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
  width: 800px;
  text-align: center;
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
