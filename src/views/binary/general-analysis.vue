<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>综合质量影响指数分析</template>
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
  import MyContent from "@/layout/content.vue";
  export default {
    name: "general-analysis",
    data: function () {
      return {
        initData: {},
        metals: ['As', 'Cd', 'Cr', 'Cu', 'Ni', 'Pb', 'Zn'],
        outputData: [],
      }
    },
    components: {
      MyContent,
    },
    methods: {
      getInitData(data) {
        this.initData = data;

      },
    }
  }
</script>

<style scoped>

</style>
