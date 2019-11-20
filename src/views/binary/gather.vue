<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>作物富集因子计算</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 作物重金属污染风险评价方法选用作物污染指数(CPI) ，该方法原理与单因子污染指数方法类似，CPI为农作物样品中元素浓度与国家食品卫生标准中相应元素的国家标准值的比值。
        <br />
      </p>
      <div style="float: left;line-height: 1.8em;">
        <p>
          <strong>输入：</strong>1. 采样点作物数据：包括pH、各种金属含量、经纬度等地理信息。2. 采样点土壤数据：包括pH、各种金属含量、经纬度等地理信息。
        </p>
        <p>
          <strong>输出：</strong>每个采样点作物富集因子和相关信息详情。
        </p>
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