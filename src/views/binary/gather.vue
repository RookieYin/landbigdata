<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>作物富集因子计算</template>
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
      <search-table :headerData="headerData" :tableData="tableData" />
    </div>
  </my-content>
</template>

<script>
import MyContent from "@/layout/content.vue";
import SearchTable from "@/components/SearchTable/index";
import table from "../../../mock/table";

export default {
  data() {
    return {
      metals: ["Cd", "Pb", "As", "Cr", "Cu", "Ni", "Zn"],
      headerData: [],
      tableData: []
    };
  },
  components: {
    MyContent,
    SearchTable
  },
  methods: {
    getInitData(data) {
      const initData = data.tableData;
      let headerData = [
        { prop: "village", label: "乡镇", sortable: true }, //
        { prop: "longitude", label: "经度" }, //
        { prop: "latitude", label: "纬度" }, //
        { prop: "metal", label: "金属类型", sortable: true }, //
        { prop: "val_land", label: "土壤含量" }, //
        { prop: "val_crop", label: "植物含量" }, //
        { prop: "p", label: "富集因子", sortable: true } //
      ];
      let tableData = [];
      for (let item of initData) {
        for (let 金属 of this.metals) {
          let 临时 = {};
          临时["village"] = item["乡镇"];
          临时["longitude"] = item["LON"].toFixed(2);
          临时["latitude"] = item["LAT"].toFixed(2);
          临时["metal"] = 金属;
          临时["val_land"] = item[金属 + "土"].toFixed(2);
          临时["val_crop"] = item[金属 + "植"].toFixed(2);
          临时["p"] = (临时["val_crop"] / 临时["val_land"]).toFixed(2);
          tableData.push(临时);
        }
      }
      this.headerData = headerData;
      this.tableData = tableData;
    }
  }
};
</script>

<style scoped>
</style>