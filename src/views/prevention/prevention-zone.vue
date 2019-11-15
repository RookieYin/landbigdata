<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>Cd污染防控分区</template>
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
              <el-table-column label="I类控制区">
                <el-table-column prop="样点数_0" label="样点数" width="120"></el-table-column>
                <el-table-column prop="比例_0" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="II类控制区">
                <el-table-column prop="样点数_1" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_1" label="比例（%）" width="100"></el-table-column>
              </el-table-column>
              <el-table-column label="III类控制区">
                <el-table-column prop="样点数_2" label="样点数" width="100"></el-table-column>
                <el-table-column prop="比例_2" label="比例（%）" width="100"></el-table-column>
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
        Cd: [0.3, 0.3, 0.3, 0.6]
    };
    const threshold = {
        Cd: [0.2, 0.25, 0.3, 0.45]
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
    import MyContent from "@/layout/content.vue";
    import GisMap from "@/components/GisMap/index.vue";
    import SearchTable from "@/components/SearchTable/index.vue";

    export default {
        name: "prevention-zone",
        data: function() {
            return {
                activeName: "first",
                initData: {},
                computedData: {},
                outputData: [],
                mapData: [],
                metals: ["Cd"],
                levelIndex: { I类控制区: 0, II类控制区: 1, III类控制区: 2},
                isMultiple: true,
                classes: [
                    ["green", "I类控制区"],
                    ["blue", "II类控制区"],
                    ["yellow", "III类控制区"],
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
                    Cd: [],
                };
                for (let item of this.initData.tableData) {
                    for (let metal in computedData) {
                        let temp = {};
                        for (let i of [
                            "longitude",
                            "latitude",
                            "乡镇",
                            "pH"
                        ]) {
                            temp[i] = item[i];
                        }
                        temp["C_soil"] = item[metal+'_soil'];
                        temp["C_crop"] = item[metal+'_crop'];
                        temp["S_threshold"] = this.getThreshold(item['pH'], metal);
                        temp["S_controlValue"] = this.getControlValue(item['pH'], metal);
                        computedData[metal].push(temp);
                    }
                }
                this.computedData = computedData;
                console.log(computedData);
                this.getOutputData(computedData);
                this.getMapData(computedData);
            },
            getControlValue(ph, metal) {
                if (ph <= 5.5) return controlValue[metal][0];
                else if (ph <= 6.5) return controlValue[metal][1];
                else if (ph <= 7.5) return controlValue[metal][2];
                else return controlValue[metal][3];
            },
            getThreshold(ph, metal) {
                if (ph < 5) return threshold[metal][0];
                else if (ph < 6) return threshold[metal][1];
                else if (ph < 7) return threshold[metal][2];
                else return threshold[metal][3];
            },
            getOutputData(computedData) {
                if (Object.keys(computedData).length === 0) return;
                let data = [];
                for (let metal of this.metals) {
                    let temp = {};
                    temp["污染物"] = metal;
                    temp["样点总数"] = computedData[metal].length;
                    let count = [0, 0, 0];
                    for (let item of computedData[metal]) {
                        if (item["C_soil"] <= item['S_threshold'] && item['C_crop'] <= 0.1) {
                            count[0] += 1;
                        } else if (item["C_soil"] <= item['S_controlValue'] && item['C_crop'] <= 0.4) {
                            count[1] += 1;
                        } else if(item["C_soil"] > item['S_controlValue'] || item['C_crop'] > 0.4)
                            count[2] += 1;
                    }
                    temp["样点数_0"] = count[0];
                    temp["比例_0"] = ((count[0] / temp["样点总数"]) * 100).toFixed(2);
                    temp["样点数_1"] = count[1];
                    temp["比例_1"] = ((count[1] / temp["样点总数"]) * 100).toFixed(2);
                    temp["样点数_2"] = count[2];
                    temp["比例_2"] = ((count[2] / temp["样点总数"]) * 100).toFixed(2);
                    data.push(temp);
                }
                this.outputData = data;
            },
            getMapData(computedData) {
                if (Object.keys(computedData).length === 0) return {};

                let headerData = [
                    { prop: "town", label: "乡镇", sortable: true }, //
                    { prop: "longitude", label: "经度" }, //
                    { prop: "latitude", label: "纬度" }, //
                    { prop: "metal", label: "金属类型", sortable: true }, //
                    { prop: "soil_val", label: "土壤金属含量", sortable: true },
                    { prop: "crop_val", label: "植物金属含量", sortable: true },
                    { prop: "soil_threshold", label: "土壤金属国家阈值" }, //
                    { prop: "soil_control_value", label: "土壤金属国家管控值" }, //
                    { prop: "crop_threshold", label: "植物金属国家限值" }, //
                    { prop: "level", label: "防控分级" } //
                ];
                let tableData = [];

                let data = [];
                for (let metal of this.metals) {
                    let temp = [];
                    for (let i = 0; i < computedData[metal].length; ++i) {
                        let tableRow = {};
                        let level = "";
                        let computedDataMetal = computedData[metal][i];
                        if (computedDataMetal["C_soil"] <= computedDataMetal['S_threshold'] && computedDataMetal['C_crop'] <= 0.1) {
                            level = "I类控制区";
                        } else if (computedDataMetal["C_soil"] <= computedDataMetal['S_controlValue'] && computedDataMetal['C_crop'] <= 0.4) {
                            level = "II类控制区";
                        } else if (computedDataMetal["C_soil"] > computedDataMetal['S_controlValue'] || computedDataMetal['C_crop'] > 0.4) {
                            level = "III类控制区";
                        }
                        tableRow["metal"] = metal;
                        tableRow["level"] = level;
                        tableRow["soil_val"] = computedDataMetal["C_soil"].toFixed(2);
                        tableRow["crop_val"] = computedDataMetal["C_crop"].toFixed(2);
                        tableRow["soil_threshold"] = computedDataMetal["S_threshold"].toFixed(2);
                        tableRow["soil_control_value"] = computedDataMetal["S_controlValue"].toFixed(2);
                        tableRow["crop_threshold"] = 0.1;
                        tableRow["longitude"] = computedDataMetal["longitude"].toFixed(2);
                        tableRow["latitude"] = computedDataMetal["latitude"].toFixed(2);
                        tableRow["town"] = computedDataMetal["乡镇"];
                        let htmlStr =
                            "<table><tbody><tr><td>" +
                            ch_en[metal] +
                            "土壤金属含量（mg）：</td><td>" +
                            tableRow["soil_val"] +
                            "</td></tr><tr><td>植物金属含量（mg）：</td><td>" +
                            tableRow["crop_val"] +
                            "</td></tr><tr><td>土壤金属含量国家阈值（mg）：</td><td>" +
                            tableRow["soil_threshold"] +
                            "</td></tr><tr><td>土壤金属含量国家管控值（mg）：</td><td>" +
                            tableRow["soil_control_value"] +
                            "</td></tr><tr><td>经度：</td><td>" +
                            tableRow["longitude"] +
                            "</td></tr><tr><td>纬度：</td><td>" +
                            tableRow["latitude"] +
                            "</td></tr><tr><td>乡镇</td><td>" +
                            tableRow["province"] +
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
</style>
