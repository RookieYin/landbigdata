<template>
  <el-container>
    <el-main>
      <el-row>
        <el-card class="my-card">
          <div slot="header" class="clearfix">
            <slot name="c-title"></slot>
          </div>
          <div style="margin-bottom: 10px;">
            <slot name="c-intro">
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
            </slot>
          </div>
        </el-card>
      </el-row>
      <el-row>
        <el-card class="my-card" id="upload-card">
          <ExcelUpload @getInitData="$emit('getInitData',$event)"></ExcelUpload>
        </el-card>
      </el-row>
      <el-row>
        <el-card>
          <slot>

          </slot>
        </el-card>
      </el-row>

    </el-main>
  </el-container>
</template>


<script>
import ExcelUpload from "@/components/ExcelUpload/upload-excel";

export default {
  name: "cluster",
  components: { ExcelUpload },
  data() {
    return {};
  },
  methods: {
      getInitData(data) {
          console.log(data);
      }
  },
  mounted() {

  }
};
</script>

<style scoped>
/*.clearfix*/
.my-card {
  margin-bottom: 10px;
}
#upload-card /deep/ .el-card {
  padding: 0px !important;
}
</style>
