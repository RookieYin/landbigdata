<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>土壤单因子分析法</template>
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
      <el-tabs class="my-tabs" type="border-card" style v-model="activeName">
        <el-tab-pane label="地图" name="first">
          <GisMap
            style="height: 500px;"
            :isMultiple="isMultiple"
            :metals="metals"
            :points="mapData"
            :classes="classes"
          ></GisMap>
        </el-tab-pane>
        <el-tab-pane label="图表" name="second">
          <div style="text-align: center;">
            <h2 style="color: #606266">土壤重金属污染物样点超标情况</h2>
            <el-table class="my-table" :data="outputData" style="margin:auto" stripe>
              <el-table-column prop="污染物" label="污染物" width="100" align="center"></el-table-column>
              <el-table-column prop="样点总数" label="样点总数" width="100"></el-table-column>
              <el-table-column label="安全">
                <el-table-column prop="样点数_0" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_0" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="警戒线">
                <el-table-column prop="样点数_1" label="样点数" width="120"></el-table-column>
                <el-table-column prop="比例_1" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="轻度污染">
                <el-table-column prop="样点数_2" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_2" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="中度污染">
                <el-table-column prop="样点数_3" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_3" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="重度污染">
                <el-table-column prop="样点数_4" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_4" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
            </el-table>
          </div>
        </el-tab-pane>
        <el-tab-pane label="详情" name="third">
          <search-table :headerData="headerData" :tableData="tableData" />
        </el-tab-pane>
      </el-tabs>
    </div>
  </my-content>
</template>

<script>
const standard = { Cd: 0.15, Ni: 1.0, As: 0.5, Cr: 1.0, Hg: 0.02, Pb: 0.2 };
const ch_en = { Cd: "镉", Ni: "镍", As: "砷", Cr: "铬", Hg: "汞", Pb: "铅" };
import MyContent from "@/layout/content.vue";
import GisMap from "@/components/GisMap/index.vue";
import SearchTable from "@/components/SearchTable/index.vue";

export default {
  name: "crop-single-factor",
  data: function() {
    return {
      activeName: "first",
      initData: {},
      computedData: {},
      outputData: [],
      mapData: [],
      metals: ["As", "Cd", "Cr", "Ni", "Pb"],
      levelIndex: { 安全: 0, 警戒线: 1, 轻度污染: 2, 中度污染: 3, 重度污染: 4 },
      isMultiple: true,
      classes: [
        ["green", "安全"],
        ["blue", "警戒线"],
        ["yellow", "轻度污染"],
        ["orange", "中度污染"],
        ["red", "重度污染"]
      ],
      tableData: [],
      headerData: []
    };
  },
  components: {
    MyContent,
    GisMap,
    SearchTable
  },
  methods: {
    getInitData(data) {
      console.log(data);
      this.initData = data;
      let computedData = { As: [], Cd: [], Cr: [], Ni: [], Pb: [] };
      for (let item of this.initData.tableData) {
        for (let metal in computedData) {
          let temp = {};
          for (let i of ["longitude", "latitude"]) {
            temp[i] = item[i];
          }
          temp["C"] = item[metal];
          temp["S"] = standard[metal];
          temp["P"] = temp["C"] / temp["S"];
          computedData[metal].push(temp);
        }
      }
      console.log(computedData);
      this.computedData = computedData;
      this.getOutputData(computedData);
      this.getMapData(computedData);
    },
    getOutputData(computedData) {
      if (Object.keys(computedData).length === 0) return;
      let data = [];
      for (let metal of this.metals) {
        let temp = {};
        temp["污染物"] = metal;
        temp["样点总数"] = computedData[metal].length;
        let count = [0, 0, 0, 0, 0];
        for (let item of computedData[metal]) {
          if (item["P"] <= 0.7) {
            count[0] += 1;
          } else if (item["P"] <= 1) {
            count[1] += 1;
          } else if (item["P"] <= 2) {
            count[2] += 1;
          } else if (item["P"] <= 3) {
            count[3] += 1;
          } else count[4] += 1;
        }
        temp["样点数_0"] = count[0];
        temp["比例_0"] = ((count[0] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_1"] = count[1];
        temp["比例_1"] = ((count[1] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_2"] = count[2];
        temp["比例_2"] = ((count[2] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_3"] = count[3];
        temp["比例_3"] = ((count[3] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_4"] = count[4];
        temp["比例_4"] = ((count[4] / temp["样点总数"]) * 100).toFixed(2);
        data.push(temp);
      }
      this.outputData = data;
    },
    getMapData(computedData) {
      if (Object.keys(computedData).length === 0) return {};

      let headerData = [
        // { prop: "province", label: "省市", sortable: true }, //
        // { prop: "district", label: "区县", sortable: true }, //
        { prop: "longitude", label: "经度" }, //
        { prop: "latitude", label: "纬度" }, //
        { prop: "metal", label: "金属类型", sortable: true }, //
        { prop: "val", label: "金属含量", sortable: true },
        { prop: "standard", label: "国家标准" }, //
        { prop: "p", label: "比值", sortable: true }, //
        { prop: "level", label: "等级" } //
      ];
      let tableData = [];

      let data = [];
      for (let metal of this.metals) {
        let temp = [];
        for (let i = 0; i < computedData[metal].length; ++i) {
          let level = "";
          let tableRow = {};
          let computedDataMetal = computedData[metal][i];
          if (computedDataMetal["P"] <= 0.7) {
            level = "安全";
          } else if (computedDataMetal["P"] <= 1) {
            level = "警戒线";
          } else if (computedDataMetal["P"] <= 2) {
            level = "轻度污染";
          } else if (computedDataMetal["P"] <= 3) {
            level = "中度污染";
          } else level = "重度污染";
          tableRow["metal"] = metal;
          tableRow["level"] = level;
          tableRow["val"] = computedDataMetal["C"].toFixed(2);
          tableRow["standard"] = computedDataMetal["S"].toFixed(2);
          tableRow["p"] = computedDataMetal["P"].toFixed(2);
          tableRow["longitude"] = computedDataMetal["longitude"].toFixed(2);
          tableRow["latitude"] = computedDataMetal["latitude"].toFixed(2);
          // tableRow["province"] =
          //     computedDataMetal["省市"] + " " + computedDataMetal["区县"];
          // tableRow["district"] =
          //     computedDataMetal["乡镇"] + " " + computedDataMetal["村"];
          tableData.push(tableRow);
          let htmlStr =
            "<table><tbody><tr><td>" +
            ch_en[metal] +
            "含量（mg）：</td><td>" +
            computedDataMetal["C"].toFixed(2) +
            "</td></tr><tr><td>国家标准（mg）：</td><td>" +
            computedDataMetal["S"].toFixed(2) +
            "</td></tr><tr><td>比值：</td><td>" +
            computedDataMetal["P"].toFixed(2) +
            "</td></tr><tr><td>经度：</td><td>" +
            computedDataMetal["longitude"].toFixed(2) +
            "</td></tr><tr><td>纬度：</td><td>" +
            computedDataMetal["latitude"].toFixed(2) +
            "</td></tr></tbody></table>";
          temp.push([
            computedDataMetal["longitude"],
            computedDataMetal["latitude"],
            this.levelIndex[level],
            htmlStr
          ]);
        }
        data.push(temp);
      }
      // console.log(data)
      this.mapData = data;
      this.headerData = headerData;
      this.tableData = tableData;
    }
  }
};
</script>

<style scoped>
.my-tabs /deep/ .el-tabs__item {
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

td {
  width: 150px;
}
</style>
