<template>
  <my-content>
    <template v-slot:c-title>决策树分析</template>
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
    <template v-slot:upload-area>
      <el-row :gutter="10">
        <el-col :span="12">
          <el-card class="my-card">
            <div slot="header" class="clearfix">
              <span>上传训练数据</span>
            </div>
            <ExcelUpload @getInitData="test"></ExcelUpload>
          </el-card>
        </el-col>
        <el-col :span="12">
          <el-card class="my-card">
            <div slot="header" class="clearfix">
              <span>上传训练数据</span>
            </div>
            <ExcelUpload @getInitData="test"></ExcelUpload>
          </el-card>
        </el-col>
        <el-col :span="24">
          <el-card class="my-card">
            <div slot="header" class="clearfix">
              <span>参数选择</span>
            </div>模型参数选择 ...
            <el-button>开始计算</el-button>
          </el-card>
        </el-col>
      </el-row>
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
        <el-tab-pane label="详情" name="second">
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
import ExcelUpload from "@/components/ExcelUpload/upload-excel";

export default {
  data: function() {
    return {
      activeName: "first",
      isMultiple: true,
      metals: ["As", "Cd", "Cr", "Cu", "Hg", "Ni", "Pb", "Zn"],
      mapData: [],
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
    SearchTable,
    ExcelUpload
  },
  methods: {
    test() {
      alert("hello");
    },
    getInitData(data) {
      // console.log(data);
      this.initData = data;
      let computedData = {
        As: [],
        Cd: [],
        Cr: [],
        Cu: [],
        Hg: [],
        Ni: [],
        Pb: [],
        Zn: []
      };
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
          temp["C"] = item[metal];
          temp["S"] = this.getStandardData(item.pH, metal);
          temp["P"] = temp["C"] / temp["S"];
          computedData[metal].push(temp);
        }
      }
      this.computedData = computedData;
      // console.log(computedData)
      this.getOutputData(computedData);
      this.getMapData(computedData);
    },
    getStandardData(ph, metal) {
      if (ph <= 5.5) return standard[metal][0];
      else if (ph <= 6.5) return standard[metal][1];
      else if (ph <= 7.5) return standard[metal][2];
      else return standard[metal][3];
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
        { prop: "province", label: "省市", sortable: true }, //
        { prop: "district", label: "区县", sortable: true }, //
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
          let tableRow = {};
          let level = "";
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
          tableRow["province"] =
            computedDataMetal["省市"] + " " + computedDataMetal["区县"];
          tableRow["district"] =
            computedDataMetal["乡镇"] + " " + computedDataMetal["村"];

          let htmlStr =
            "<table><tbody><tr><td>" +
            ch_en[metal] +
            "含量（mg）：</td><td>" +
            tableRow["val"] +
            "</td></tr><tr><td>国家标准（mg）：</td><td>" +
            tableRow["standard"] +
            "</td></tr><tr><td>比值：</td><td>" +
            tableRow["p"] +
            "</td></tr><tr><td>经度：</td><td>" +
            tableRow["longitude"] +
            "</td></tr><tr><td>纬度：</td><td>" +
            tableRow["latitude"] +
            "</td></tr><tr><td>省市</td><td>" +
            tableRow["province"] +
            "</td></tr><tr><td>区县</td><td>" +
            tableRow["district"] +
            "</td></tr></tbody></table>";
          // let div = document.createElement('div')
          // div.innerHTML = htmlStr
          // div.style.float = 'right'
          // document.body.appendChild(div)
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
      console.log(data);
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

.my-card {
  margin-bottom: 10px;
}
.my-card /deep/ .el-card {
  padding: 0px !important;
}
</style>
