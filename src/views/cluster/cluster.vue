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
        <el-tab-pane label="地图" name="first">1</el-tab-pane>
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
  methods: {
    getInitData() {}
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
</style>