<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>防控监测</template>
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
              <el-table-column label="优">
                <el-table-column prop="样点数_0" label="样点数" width="120"></el-table-column>
                <el-table-column prop="比例_0" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="良好">
                <el-table-column prop="样点数_1" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_1" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="欠缺">
                <el-table-column prop="样点数_2" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_2" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="差">
                <el-table-column prop="样点数_3" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_3" label="比例（%）" width="100"></el-table-column>
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
const controlValue = {
  Cd: [2, 3, 6]
};
const ch_en = {
  Cd: "镉",
  Ni: "镍",
  Cu: "铜",
  Zn: "锌",
  As: "砷",
  Cr: "铬",
  Hg: "汞",
  Pb: "铅"
};
const raw_header = [
  "有效态浓度",
  "longitude",
  "latitude",
  "省市",
  "区县",
  "乡镇",
  "村"
];
import MyContent from "@/layout/content.vue";
import GisMap from "@/components/GisMap/index.vue";
import SearchTable from "@/components/SearchTable/index.vue";

export default {
  data: function() {
    return {
      activeName: "first",
      initData: {},
      computedData: {},
      outputData: [],
      mapData: [],
      metals: ["Cd"],
      levelIndex: { 优: 0, 良好: 1, 欠佳: 2, 差: 3 },
      isMultiple: true,
      classes: [
        ["green", "优"],
        ["yellow", "良好"],
        ["orange", "欠佳"],
        ["red", "差"]
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
      this.initData = data;
      let computedData = {
        Cd: []
      };
      for (let item of this.initData.tableData) {
        for (let metal in computedData) {
          let temp = {};
          for (let i of raw_header) {
            temp[i] = item[i];
          }
          computedData[metal].push(temp);
        }
      }
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
        let count = [0, 0, 0, 0];
        for (let item of computedData[metal]) {
          const 有效态浓度 = item["有效态浓度"];
          if (有效态浓度 <= controlValue[metal][0]) {
            count[0] += 1;
          } else if (有效态浓度 <= controlValue[metal][1]) {
            count[1] += 1;
          } else if (有效态浓度 <= controlValue[metal][2]) {
            count[2] += 1;
          } else count[3] += 1;
        }
        temp["样点数_0"] = count[0];
        temp["比例_0"] = ((count[0] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_1"] = count[1];
        temp["比例_1"] = ((count[1] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_2"] = count[2];
        temp["比例_2"] = ((count[2] / temp["样点总数"]) * 100).toFixed(2);
        temp["样点数_3"] = count[3];
        temp["比例_3"] = ((count[3] / temp["样点总数"]) * 100).toFixed(2);
        data.push(temp);
      }
      this.outputData = data;
    },
    getMapData(computedData) {
      if (Object.keys(computedData).length === 0) return {};

      let headerData = [
        { prop: "province", label: "省市", sortable: true }, //
        { prop: "district", label: "区县", sortable: true }, //
        { prop: "longitude", label: "经度" }, //
        { prop: "latitude", label: "纬度" }, //
        { prop: "metal", label: "金属类型", sortable: true }, //
        { prop: "val", label: "有效态浓度", sortable: true },
        { prop: "level", label: "等级" } //
      ];
      let tableData = [];

      let data = [];
      for (let metal of this.metals) {
        let temp = [];
        for (let i = 0; i < computedData[metal].length; ++i) {
          let tableRow = {};
          let level = "";
          let computedDataMetal = computedData[metal][i];
          const 有效态浓度 = computedDataMetal["有效态浓度"];
          if (有效态浓度 <= controlValue[metal][0]) {
            level = this.classes[0][1];
          } else if (有效态浓度 <= controlValue[metal][1]) {
            level = this.classes[1][1];
          } else if (有效态浓度 <= controlValue[metal][2]) {
            level = this.classes[2][1];
          } else level = this.classes[3][1];

          tableRow["metal"] = metal;
          tableRow["level"] = level;
          tableRow["val"] = 有效态浓度.toFixed(2);
          tableRow["longitude"] = computedDataMetal["longitude"].toFixed(2);
          tableRow["latitude"] = computedDataMetal["latitude"].toFixed(2);
          tableRow["province"] =
            computedDataMetal["省市"] + " " + computedDataMetal["区县"];
          tableRow["district"] =
            computedDataMetal["乡镇"] + " " + computedDataMetal["村"];

          let htmlStr =
            "<table><tbody><tr><td>" +
            ch_en[metal] +
            "有效态浓度（ug/L）：</td><td>" +
            tableRow["val"] +
            "</td></tr><tr><td>经度：</td><td>" +
            tableRow["longitude"] +
            "</td></tr><tr><td>纬度：</td><td>" +
            tableRow["latitude"] +
            "</td></tr><tr><td>省市</td><td>" +
            tableRow["province"] +
            "</td></tr><tr><td>区县</td><td>" +
            tableRow["district"] +
            "</td></tr></tbody></table>";

          temp.push([
            computedDataMetal["longitude"],
            computedDataMetal["latitude"],
            this.levelIndex[level],
            htmlStr
          ]);
          tableData.push(tableRow);
        }
        data.push(temp);
      }
      this.mapData = data;
      this.headerData = headerData;
      this.tableData = tableData;
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

td {
  width: 150px;
}
</style>
