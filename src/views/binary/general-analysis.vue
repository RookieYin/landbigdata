<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>土壤综合质量影响指数分析</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 土壤和农产品综合指数法可以解决在土壤-农产品复合污染影响评价中无法兼顾土壤和农产品的问题，将土壤和农产品品质较好的联系起来，推进了农田土壤和作物系统重金属复合影响评价的量化进程。农田(或耕地)土壤重金属复合影响中的综合质量影响指数由土壤综合质量影响指数和农作物综合质量影响指数组成。
      </p>
      <div style="float: left;line-height: 1.8em;">
        <p>
          <strong>输入：</strong>1. 采样点作物数据：包括pH、各种金属含量、经纬度等地理信息。2. 采样点土壤数据：包括pH、各种金属含量、经纬度等地理信息。
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
              <el-table-column prop="样点总数" label="样点总数" width="100"></el-table-column>
              <el-table-column label="I级">
                <el-table-column prop="样点数_0" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_0" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="II级">
                <el-table-column prop="样点数_1" label="样点数" width="120"></el-table-column>
                <el-table-column prop="比例_1" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="III级">
                <el-table-column prop="样点数_2" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_2" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="IV级">
                <el-table-column prop="样点数_3" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_3" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="V级">
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
//土壤重金属标准值
const soilStandard = {
  Cd: [0.3, 0.3, 0.3, 0.6],
  Ni: [60, 70, 100, 190],
  Cu: [50, 50, 100, 100],
  Zn: [200, 200, 250, 300],
  As: [40, 40, 30, 25],
  Cr: [150, 150, 200, 250],
  Hg: [1.3, 1.8, 2.4, 3.4],
  Pb: [70, 90, 120, 170]
};
//土壤重金属背景值
const backgroundData = {
  Cd: 0.09,
  Pb: 20.6,
  Cu: 19.88,
  Zn: 66.87,
  Cr: 63.69,
  Ni: 26.69,
  As: 8.39,
  Hg: 0.04
};
//作物国家标准值
const cropStandards = {
  Cd: 0.15,
  Ni: 1.0,
  As: 0.5,
  Cr: 1.0,
  Hg: 0.02,
  Pb: 0.2
};
//金属氧化数
const oxidationNum = { Cd: 2, Ni: 2, As: 5, Cr: 3, Hg: 2, Pb: 2 };
import MyContent from "@/layout/content.vue";
import GisMap from "@/components/GisMap/index.vue";
import SearchTable from "@/components/SearchTable/index.vue";

export default {
  name: "general-analysis",
  data: function() {
    return {
      activeName: "first",
      initData: {},
      computedData: [],
      outputData: [],
      mapData: [],
      metals: ["As", "Cd", "Cr", "Ni", "Pb"],
      levelIndex: { I: 0, II: 1, III: 2, IV: 3, V: 4 },
      isMultiple: false,
      classes: [
        ["green", "I"],
        ["blue", "II"],
        ["yellow", "III"],
        ["orange", "IV"],
        ["red", "V"]
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
      let computedData = [];
      let metals = this.metals;
      this.initData = data;
      for (let item of data.tableData) {
        let [DDSB, Sum_Pss, Sum_Psb, Sum_Pap, X, Y, Z] = [0, 0, 0, 0, 0, 0, 0];
        for (let metal of metals) {
          let metalStandard = this.getStandardData(item.ph, metal);
          let metalBackground = backgroundData[metal];
          let cropStandard = cropStandards[metal];
          let Pss = item[metal + "土"] / metalStandard;
          if (Pss > 1) X += 1;
          let Psb = item[metal + "土"] / metalBackground;
          if (Psb > 1) Y += 1;
          let Pap = item[metal + "植"] / cropStandard;
          if (Pap > 1) Z += 1;
          Sum_Pss += Math.pow(Pss, 1 / oxidationNum[metal]);
          Sum_Psb += Math.pow(Psb, 1 / oxidationNum[metal]);
          Sum_Pap += Math.pow(Pap, 1 / oxidationNum[metal]);
          DDSB += metalStandard / metalBackground;
        }
        let temp = {};
        for (let i of ["LON", "LAT", "乡镇", "ph"]) {
          temp[i] = item[i];
        }
        temp.RIE = Sum_Pss / metals.length;
        temp.DDDB = Sum_Psb / metals.length;
        temp.DDSB = DDSB;
        temp.IICQs = X * (1 + temp.RIE) + Y * (temp.DDDB / temp.DDSB);
        temp.QIAP = Sum_Pap / metals.length;
        temp.IICQap = Z * (1 + temp.QIAP / 5) + temp.QIAP / (5 * temp.DDSB);
        temp.IICQ = temp.IICQs + temp.IICQap;
        temp.X = X;
        temp.Y = Y;
        temp.Z = Z;
        computedData.push(temp);
        if (temp.IICQ < 5) console.log(temp.IICQ);
      }
      this.computedData = computedData;
      this.getOutputData(computedData);
      this.getMapData(computedData);
    },
    getStandardData(ph, metal) {
      if (ph <= 5.5) return soilStandard[metal][0];
      else if (ph <= 6.5) return soilStandard[metal][1];
      else if (ph <= 7.5) return soilStandard[metal][2];
      else return soilStandard[metal][3];
    },
    getOutputData(computedData) {
      let data = [];
      let temp = {};
      temp["样点总数"] = computedData.length;
      let count = [0, 0, 0, 0, 0];
      for (let item of computedData) {
        if (item["IICQ"] <= 1) {
          count[0] += 1;
        } else if (item["IICQ"] <= 2) {
          count[1] += 1;
        } else if (item["IICQ"] <= 3) {
          count[2] += 1;
        } else if (item["IICQ"] <= 5) {
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
      this.outputData = data;
    },
    getMapData(computedData) {
      if (Object.keys(computedData).length === 0) return;

      let headerData = [
        { prop: "town", label: "乡镇", sortable: true }, //
        { prop: "longitude", label: "经度" }, //
        { prop: "latitude", label: "纬度" }, //
        { prop: "p", label: "综合污染指数", sortable: true }, //
        { prop: "level", label: "等级" } //
      ];
      let tableData = [];

      let temp = [];
      for (let i = 0; i < computedData.length; ++i) {
        let tableRow = {};
        let level = "";
        let computedDataMetal = computedData[i];
        if (computedDataMetal["IICQ"] <= 1) {
          level = "I";
        } else if (computedDataMetal["IICQ"] <= 2) {
          level = "II";
        } else if (computedDataMetal["IICQ"] <= 3) {
          level = "III";
        } else if (computedDataMetal["IICQ"] <= 5) {
          level = "IV";
        } else level = "V";

        tableRow["town"] = computedDataMetal["乡镇"];
        tableRow["longitude"] = computedDataMetal["LON"].toFixed(2);
        tableRow["latitude"] = computedDataMetal["LAT"].toFixed(2);
        tableRow["p"] = computedDataMetal["ph"].toFixed(2);
        tableRow["level"] = level;

        let htmlStr =
          "<table><tbody><tr><td>综合污染指数：</td><td>" +
          tableRow["p"] +
          "</td></tr><tr><td>经度：</td><td>" +
          tableRow["longitude"] +
          "</td></tr><tr><td>纬度：</td><td>" +
          tableRow["latitude"] +
          "</td></tr><tr><td>乡镇</td><td>" +
          tableRow["town"] +
          "</td></tr></tbody></table>";
        temp.push([
          computedDataMetal["LON"],
          computedDataMetal["LAT"],
          this.levelIndex[level],
          htmlStr
        ]);
        tableData.push(tableRow);
      }
      // console.log(temp)
      this.mapData = temp;
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
.my-table /deep/ tbody tr:first-child {
  font-weight: bold;
}
</style>
