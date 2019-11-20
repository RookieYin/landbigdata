<template>
  <my-content>
    <template v-slot:c-title>决策树分析</template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 决策论中，决策树由一个决策图和可能的结果组成，用来创建到达目标的规划。决策树建立并用来辅助决策，是一种特殊的树结构。决策树是一个利用像树一样的图形或决策模型的决策支持工具，包括随机事件结果，资源代价和实用性。它是一个算法显示的方法。决策树经常在运筹学中使用，特别是在决策分析中，它帮助确定一个能最可能达到目标的策略。
        <br />
      </p>
      <div style="float: left;line-height: 1.8em;">
        <p>
          <strong>输入：</strong>1. 用于训练模型的一组数据，包含各种土壤信息和土壤上植物的金属含量。2. 用于预测的一组真实数据。3. 决策树模型参数。
        </p>
        <p>
          <strong>输出：</strong>对于真实数据的预测详情和地图可视化。
        </p>
      </div>
    </template>
    <template v-slot:upload-area>
      <el-row>
        <el-card class="my-card">
          <div slot="header" class="分裂节点评价准则">
            <span>参数选择</span>
          </div>
          <el-form :inline="true" :model="options" class="demo-form-inline">
            <el-row>
              <el-col :span="8">
                <el-form-item label="增益函数">
                  <el-select v-model="options.gainFunction">
                    <el-option label="gini" value="gini"></el-option>
                    <el-option label="regression" value="mean"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="树最大深度">
                  <el-input v-model="options.maxDepth"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="叶子结点最小样本数">
                  <el-input v-model="options.minNumSamples"></el-input>
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
import { DecisionTreeClassifier as DTClassifier } from "ml-cart";

export default {
  data: function() {
    return {
      activeName: "first",
      isMultiple: false,
      classifier: null,
      metals: ["As", "Cd", "Ca", "Cr", "Cu", "Mn", "Ni", "Pb", "Zn"],
      options: {
        gainFunction: "gini",
        maxDepth: 10,
        minNumSamples: 3
      },
      mapData: [],
      classes: [["green", "无污染"], ["blue", "有污染"]],
      tableData: [],
      headerData: [],
      isUpdateTrain: false
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
      for (let item of data.tableData) {
        let temp = [];
        for (let m of this.metals) {
          temp.push(item["T_" + m]);
        }
        temp.push(item.pH);
        trainX.push(temp);
        trainY.push(item.C_Cd);
      }
      console.log(trainX);
      console.log(trainY);
      let classifier = new DTClassifier(this.options);
      classifier.train(trainX, trainY);
      console.log("训练完成");
      this.classifier = classifier;
      this.isUpdateTrain = true;
    },
    getPredictData(data) {
      // if(!this.isUpdateTrain){
      //   alert("请先上传训练数据！");
      //   return;
      // }
      console.log(data);
      let predictX = [];
      let predictData = [];
      for (let item of data.tableData) {
        let temp = [];
        for (let m of this.metals) {
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
          predictY[i] % 2,
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
