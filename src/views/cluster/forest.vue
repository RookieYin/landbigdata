<template>
  <my-content>
    <template v-slot:c-title>随机森林分析</template>
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
      <el-row>
        <el-card class="my-card">
          <div slot="header" class="clearfix">
            <span>参数选择</span>
          </div>
          <el-form :inline="true" :model="options" class="demo-form-inline">
            <el-row>
              <el-col :span="6">
                <el-form-item label="seed">
                  <el-input v-model="options.seed" placeholder="审批人"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="maxFeatures">
                  <el-input v-model="options.maxFeatures" placeholder="审批人"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="replacement">
                  <el-select v-model="options.replacement" placeholder="活动区域">
                    <el-option label="true" value="true"></el-option>
                    <el-option label="false" value="false"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="nEstimators">
                  <el-input v-model="options.nEstimators" placeholder="审批人"></el-input>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
        </el-card>
      </el-row>
      <el-row :gutter="10">
        <el-col :span="12">
          <el-card class="my-card">
            <div slot="header" class="clearfix">
              <span>上传训练数据</span>
            </div>
            <ExcelUpload @getInitData="getTrainData"></ExcelUpload>
          </el-card>
        </el-col>
        <el-col :span="12">
          <el-card class="my-card">
            <div slot="header" class="clearfix">
              <span>上传预测数据</span>
            </div>
            <ExcelUpload @getInitData="getPredictData"></ExcelUpload>
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
import { RandomForestClassifier as RFClassifier } from 'ml-random-forest';

export default {
  data: function() {
    return {
      activeName: "first",
      isMultiple: false,
      classifier: null,
      metals: ["As", "Cd", "Ca", "Cr", "Cu", "Mn", "Ni", "Pb", "Zn"],
      options: {
        seed: 3,
        maxFeatures: 0.8,
        replacement: true,
        nEstimators: 25,
      },
      mapData: [],
      classes: [
        ["green", "无污染"],
        ["blue", "有污染"],
      ],
      tableData: [],
      headerData: [],
      isUpdateTrain: false,
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
    getTrainData(data) {
      console.log(data);
      let trainX = [];
      let trainY = [];
      for(let item of data.tableData){
        let temp = [];
        for(let m of this.metals){
          temp.push(item["T_" + m]);
        }
        temp.push(item.pH);
        trainX.push(temp);
        trainY.push(item.C_Cd);
      }
      console.log(trainX);
      console.log(trainY);
      let classifier = new RFClassifier(this.options);
      classifier.train(trainX, trainY);
      console.log('训练完成');
      this.classifier = classifier;
      this.isUpdateTrain = true;
    },
    getPredictData(data){
      // if(!this.isUpdateTrain){
      //   alert("请先上传训练数据！");
      //   return;
      // }
      console.log(data);
      let predictX = [];
      let predictData = [];
      for(let item of data.tableData){
        let temp = [];
        for(let m of this.metals){
          temp.push(item["T_" + m]);
        }
        temp.push(item.pH);
        predictX.push(temp.slice(0));
        temp.push(item.latitude);
        temp.push(item.longitude);
        predictData.push(item);
      }
      console.log(predictData);
      console.log(predictX);
      let predictY = this.classifier.predict(predictX);
      console.log(predictY);
      this.getMapData(predictData, predictY);
    },
    getMapData(predictData, predictY) {
      if (Object.keys(predictData).length === 0) return {};

      let headerData = [
        { prop: "longitude", label: "经度" }, //
        { prop: "latitude", label: "纬度" },
        { prop: "level", label: "污染情况" } //
      ];
      let tableData = [];
      let data = [];
      for (let i = 0; i < predictData.length; ++i) {
        let tableRow = {};
        let level = "";
        tableRow["level"] = predictY[i];
        tableRow["longitude"] = predictData[i]["longitude"].toFixed(2);
        tableRow["latitude"] = predictData[i]["latitude"].toFixed(2);
        let htmlStr =
          "<table><tbody><tr><td>" +
          "经度：</td><td>" +
          tableRow["longitude"] +
          "</td></tr><tr><td>纬度：</td><td>" +
          tableRow["latitude"] +
          "</td></tr></tbody></table>";
        data.push([
          predictData[i]["longitude"],
          predictData[i]["latitude"],
          predictY[i]%2,
          htmlStr
        ]);
        tableData.push(tableRow);
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
