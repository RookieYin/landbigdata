<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>Cd污染防控分区</template>
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
    <div style="width:1050px;border-bottom: solid 1px;margin:0 auto;">
      <el-row style="border-bottom: solid 1px;border-top: solid 1px;line-height: 30px;">
        <el-col :span="2" style="text-align: center"><strong>元素</strong></el-col>
        <el-col :span="22" style="text-align: center"><strong>多元线性回归方程</strong></el-col>
      </el-row>
      <el-row v-for="(metal, index) in metals" style="margin-top:10px;margin-bottom:10px;">
        <el-col :span="2" style="text-align: center"><div>{{ metal }}</div></el-col>
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
  import { VueMathjax } from 'vue-mathjax';
  export default {
    name: "multi-regression",
    data: function () {
      return {
        initData: {},
        algorithmData: [],
        formulas: [],
        metals: ['As', 'Cd', 'Cr', 'Cu', 'Ni', 'Pb', 'Zn'],
        outputData: [],
      }
    },
    components: {
      MyContent,
      VueMathjax,
    },
    methods: {
      getInitData(data) {
        this.initData = data;
        console.log(data);
        let soilData = [];
        let cropData = {};
        let metal = this.metals;
        let weights = [];
        for(let m of metal){
          cropData[m] = [];
        }
        for(let i = 0; i < data.tableData.length; i++){
          let temp = [];
          for(let m of metal){
            temp.push(data.tableData[i][m+'土']);
            cropData[m].push([data.tableData[i][m+'植']]);
          }
          soilData.push(temp);
        }
        console.log(soilData);
        console.log(cropData);
        for(let m of metal) {
          let mlr = new MLR(soilData, cropData[m]);
          console.log(mlr);
          weights.push(mlr.weights.map((weight) => weight[0]));
        }
        console.log(weights);
        for(let i = 0; i < metal.length; i++){
          let temp = '';
          for(let j = 0; j < weights[i].length; ++j){
            if(weights[i][j] < 0.1 && weights[i][j] > -0.1)
              continue;
            if(weights[i][j] < 0 || j === 0)
              temp += weights[i][j].toFixed(4) + 'X_{'+metal[i]+'}';
            else
              temp += '+'+ weights[i][j].toFixed(4) + 'X_{'+metal[i]+'}';
          }
          console.log(temp);
          if(temp[0] === '+')
            temp = temp.slice(1);
          this.formulas.push('$$Y_{'+metal[i]+'}={' + temp+'}.$$');
        }
      },
    }
  }
</script>

<style scoped>

</style>
