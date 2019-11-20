<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>多元回归分析</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 通过相关分析可以了解变量两两之间的相关程度，但无法分析变量之间的影响规律，而回归分析则可以通过确定系数展现因果关系，同时有预测的功能，另外当变量之间确认是弱相关或者不相关时，通过回归分析可以将因样本量少、误差大和可能非线性相关的变量之间的相关关系展示出来。
        <br />
      </p>
      <div style="float: left;line-height: 1.8em;">
        <p>
          <strong>输入：</strong>1. 采样点作物数据：包括pH、各种金属含量、经纬度等地理信息。2. 采样点土壤数据：包括pH、各种金属含量、经纬度等地理信息。
        </p>
        <p>
          <strong>输出：</strong>多元线性回归方程方程组。
        </p>
      </div>
    </template>
    <div style="width:1050px;border-bottom: solid 1px;margin:0 auto;">
      <el-row style="border-bottom: solid 1px;border-top: solid 1px;line-height: 30px;">
        <el-col :span="2" style="text-align: center">
          <strong>元素</strong>
        </el-col>
        <el-col :span="22" style="text-align: center">
          <strong>多元线性回归方程</strong>
        </el-col>
      </el-row>
      <el-row v-for="(metal, index) in metals" style="margin-top:10px;margin-bottom:10px;">
        <el-col :span="2" style="text-align: center">
          <div>{{ metal }}</div>
        </el-col>
        <el-col :span="22">
          <vue-mathjax :formula="formulas[index]"></vue-mathjax>
        </el-col>
      </el-row>
    </div>
  </my-content>
</template>

<script>
import { MultivariateLinearRegression as MLR } from "ml";
import MyContent from "@/layout/content.vue";
import { VueMathjax } from "vue-mathjax";
export default {
  name: "multi-regression",
  data: function() {
    return {
      initData: {},
      algorithmData: [],
      formulas: [],
      metals: ["As", "Cd", "Cr", "Cu", "Ni", "Pb", "Zn"],
      outputData: []
    };
  },
  components: {
    MyContent,
    VueMathjax
  },
  methods: {
    getInitData(data) {
      this.initData = data;
      console.log(data);
      let soilData = [];
      let cropData = {};
      let metal = this.metals;
      let weights = [];
      for (let m of metal) {
        cropData[m] = [];
      }
      for (let i = 0; i < data.tableData.length; i++) {
        let temp = [];
        for (let m of metal) {
          temp.push(data.tableData[i][m + "土"]);
          cropData[m].push([data.tableData[i][m + "植"]]);
        }
        soilData.push(temp);
      }
      console.log(soilData);
      console.log(cropData);
      for (let m of metal) {
        let mlr = new MLR(soilData, cropData[m]);
        console.log(mlr);
        weights.push(mlr.weights.map(weight => weight[0]));
      }
      console.log(weights);
      for (let i = 0; i < metal.length; i++) {
        let temp = "";
        for (let j = 0; j < weights[i].length; ++j) {
          if (weights[i][j] < 0.1 && weights[i][j] > -0.1) continue;
          if (weights[i][j] < 0 || j === 0)
            temp += weights[i][j].toFixed(4) + "X_{" + metal[i] + "}";
          else temp += "+" + weights[i][j].toFixed(4) + "X_{" + metal[i] + "}";
        }
        console.log(temp);
        if (temp[0] === "+") temp = temp.slice(1);
        this.formulas.push("$$Y_{" + metal[i] + "}={" + temp + "}.$$");
      }
    }
  }
};
</script>

<style scoped>
</style>
