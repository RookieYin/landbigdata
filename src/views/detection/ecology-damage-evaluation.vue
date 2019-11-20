<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>潜在生态危害评价</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 潜在生态危害评价由瑞典著名地球化学家Lars Hakanson提出的应用河流湖泊沉积学原理评价土壤重金属生态风险的方法，是目前运用最广泛的重金属生态风险评价方法之一。国内外已有众多学者将其应用于不同区域土壤重金属的评价。它综合考虑了土壤重金属含量、生态效应及环境效应之间的联系,是涵盖面很广泛的评价方法。
        <br />
      </p>
      <div style="float: left;line-height: 1.8em;">
        <p>
          <strong>输入：</strong>1. 采样点作物数据：包括pH、各种金属含量、经纬度等地理信息。2. 作物金属含量标准。 3. 全球工业化前沉积物汇总污染物含量和金属毒性系数。
        </p>
        <p>
          <strong>输出：</strong>采样点信息地图，采样点污染情况统计图表和采样点污染详细信息。
        </p>
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
              <el-table-column label="轻微">
                <el-table-column prop="样点数_0" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_0" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="中等">
                <el-table-column prop="样点数_1" label="样点数" width="120"></el-table-column>
                <el-table-column prop="比例_1" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="强">
                <el-table-column prop="样点数_2" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_2" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="很强">
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
import MyContent from "@/layout/content.vue";
import GisMap from "@/components/GisMap/index.vue";
import SearchTable from "@/components/SearchTable/index.vue";
let T = { Cd: 30, As: 10, Cu: 5, Pb: 5, Cr: 2, Zn: 1 };
//为了演示效果，更改金属毒性
for (let item in T) {
  T[item] *= 5;
}
const C_n = { Cd: 10, As: 15, Cu: 50, Pb: 70, Cr: 90, Zn: 175 };
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
export default {
  name: "ecology-damage-evaluation",
  data: function() {
    return {
      activeName: "first",
      initData: {},
      computedData: {},
      outputData: [],
      mapData: [],
      metals: ["As", "Cd", "Cr", "Cu", "Pb", "Zn"],
      levelIndex: { 轻微: 0, 中等: 1, 强: 2, 很强: 3 },
      isMultiple: true,
      classes: [
        ["green", "轻微"],
        ["blue", "中等"],
        ["yellow", "强"],
        ["orange", "很强"]
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
      let computedData = { As: [], Cd: [], Cr: [], Cu: [], Pb: [], Zn: [] };
      for (let item of this.initData.tableData) {
        for (let metal in computedData) {
          let temp = {};
          for (let i of [
            "longitude",
            "latitude",
            "省市",
            "区县",
            "乡镇",
            "村",
            "pH"
          ]) {
            temp[i] = item[i];
          }
          temp["C_i"] = item[metal];
          temp["C_n"] = C_n[metal];
          temp["C_f"] = temp["C_i"] / temp["C_n"];
          temp["E"] = temp["C_f"] * T[metal];
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
        let count = [0, 0, 0, 0];
        for (let item of computedData[metal]) {
          if (item["E"] < 40) {
            count[0] += 1;
          } else if (item["E"] < 80) {
            count[1] += 1;
          } else if (item["E"] < 160) {
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
      let RIs = [];
      for (let i = 0; i < computedData["As"].length; ++i) {
        RIs.push(
          computedData["As"][i]["E"] +
            computedData["Cd"][i]["E"] +
            computedData["Cu"][i]["E"] +
            computedData["Pb"][i]["E"] +
            computedData["Cr"][i]["E"] +
            computedData["Zn"][i]["E"]
        );
      }
      // console.log(RIs)
      let RI = {};
      RI["污染物"] = "RI";
      RI["样点总数"] = computedData["As"].length;
      let count = [0, 0, 0, 0];
      for (let ri of RIs) {
        if (ri < 150) {
          count[0] += 1;
        } else if (ri < 300) {
          count[1] += 1;
        } else if (ri < 600) {
          count[2] += 1;
        } else count[3] += 1;
      }
      RI["样点数_0"] = count[0];
      RI["比例_0"] = ((count[0] / RI["样点总数"]) * 100).toFixed(2);
      RI["样点数_1"] = count[1];
      RI["比例_1"] = ((count[1] / RI["样点总数"]) * 100).toFixed(2);
      RI["样点数_2"] = count[2];
      RI["比例_2"] = ((count[2] / RI["样点总数"]) * 100).toFixed(2);
      RI["样点数_3"] = count[3];
      RI["比例_3"] = ((count[3] / RI["样点总数"]) * 100).toFixed(2);
      data.push(RI);
      this.outputData = data;
      // console.log(data);
    },
    getMapData(computedData) {
      if (Object.keys(computedData).length === 0) return {};
      let data = [];
      let headerData = [
        { prop: "province", label: "省市", sortable: true }, //
        { prop: "district", label: "区县", sortable: true }, //
        { prop: "longitude", label: "经度" }, //
        { prop: "latitude", label: "纬度" }, //
        { prop: "metal", label: "金属类型", sortable: true }, //
        { prop: "val", label: "金属含量", sortable: true },
        { prop: "standard", label: "国家标准" }, //
        { prop: "e", label: "潜在污染风险", sortable: true }, //
        { prop: "level", label: "等级" } //
      ];
      let tableData = [];
      for (let metal of this.metals) {
        let temp = [];
        for (let i = 0; i < computedData[metal].length; ++i) {
          let level = "";
          let computedDataMetal = computedData[metal][i];
          if (computedDataMetal["E"] < 40) {
            level = "轻微";
          } else if (computedDataMetal["E"] < 80) {
            level = "中等";
          } else if (computedDataMetal["E"] < 160) {
            level = "强";
          } else level = "很强";
          let tableRow = {};
          tableRow["metal"] = metal;
          tableRow["level"] = level;
          tableRow["val"] = computedDataMetal["C_i"].toFixed(2);
          tableRow["standard"] = computedDataMetal["C_n"].toFixed(2);
          tableRow["e"] = computedDataMetal["E"].toFixed(2);
          tableRow["longitude"] = computedDataMetal["longitude"].toFixed(2);
          tableRow["latitude"] = computedDataMetal["latitude"].toFixed(2);
          tableRow["province"] =
            computedDataMetal["省市"] + " " + computedDataMetal["区县"];
          tableRow["district"] =
            computedDataMetal["乡镇"] + " " + computedDataMetal["村"];
          tableData.push(tableRow);
          let htmlStr =
            "<table><tbody><tr><td>" +
            ch_en[metal] +
            "含量（mg）：</td><td>" +
            computedDataMetal["C_i"].toFixed(2) +
            "</td></tr><tr><td>比值：</td><td>" +
            computedDataMetal["E"].toFixed(2) +
            "</td></tr><tr><td>经度：</td><td>" +
            computedDataMetal["longitude"].toFixed(2) +
            "</td></tr><tr><td>纬度：</td><td>" +
            computedDataMetal["latitude"].toFixed(2) +
            "</td></tr><tr><td>省市</td><td>" +
            computedDataMetal["省市"] +
            " " +
            computedDataMetal["区县"] +
            "</td></tr><tr><td>区县</td><td>" +
            computedDataMetal["乡镇"] +
            " " +
            computedDataMetal["村"] +
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
.my-table /deep/ tbody tr:last-child {
  font-weight: bold;
}
</style>
