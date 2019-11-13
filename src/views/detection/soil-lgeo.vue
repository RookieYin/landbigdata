<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>
      土壤地累计指数法
    </template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 通过土壤重金属浓度与自然地球化学背景值之间的关系来确定重金属污染程度的一种方法。<br/>
      </p>
      <div style="float: left;line-height: 1.8em;">
        <strong>输入：</strong>
      </div>
      <div style="float:left;margin-bottom: 20px;line-height: 1.8em;">
        自然地球化学背景值 与 采样点土壤重金属浓度。
      </div>
    </template>
    <div class="map">
      <el-tabs class="mytabs" type="border-card" style="" v-model="activeName">
        <el-tab-pane label="地图" name="first">
          <GisMap style="height: 500px;"
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
              <el-table-column
                prop="污染物"
                label="污染物"
                width="100"
                align="center">
              </el-table-column>
              <el-table-column
                prop="样点总数"
                label="样点总数"
                width="100">
              </el-table-column>
              <el-table-column label="无污染">
                <el-table-column
                  prop="样点数_0"
                  label="样点数"
                  width="100">
                </el-table-column>
                <el-table-column
                  prop="比例_0"
                  label="比例（%）"
                  width="100">
                </el-table-column>
              </el-table-column>
              <el-table-column label="轻度污染">
                <el-table-column
                  prop="样点数_1"
                  label="样点数"
                  width="120">
                </el-table-column>
                <el-table-column
                  prop="比例_1"
                  label="比例（%）"
                  width="100">
                </el-table-column>
              </el-table-column>
              <el-table-column label="中度污染">
                <el-table-column
                  prop="样点数_2"
                  label="样点数"
                  width="100">
                </el-table-column>
                <el-table-column
                  prop="比例_2"
                  label="比例（%）"
                  width="100">
                </el-table-column>
              </el-table-column>
              <el-table-column label="重度污染">
                <el-table-column
                  prop="样点数_3"
                  label="样点数"
                  width="100">
                </el-table-column>
                <el-table-column
                  prop="比例_3"
                  label="比例（%）"
                  width="100">
                </el-table-column>
              </el-table-column>
              <el-table-column label="极重度污染">
                <el-table-column
                  prop="样点数_4"
                  label="样点数"
                  width="100">
                </el-table-column>
                <el-table-column
                  prop="比例_4"
                  label="比例（%）"
                  width="100">
                </el-table-column>
              </el-table-column>
            </el-table>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
  </my-content>
</template>

<script>
const backgroundData = {Cd: 0.09, Pb: 20.60, Cu: 19.88, Zn: 66.87, Cr: 63.69, Ni: 26.69, As: 8.39, Hg: 0.04}
const ch_en = {Cd: '镉', Ni: '镍', Cu: '铜', Zn: '锌', As: '砷', Cr: '铬', Hg: '汞', Pb: '铅'}
import MyContent from '@/layout/content.vue'
import GisMap from '@/components/GisMap/index.vue'

export default {
    name: "soil-lgeo",
    data: function () {
        return {
            activeName: "first",
            initData: {},
            computedData: {},
            outputData: [],
            mapData: [],
            classes: [["green", '无污染'], ["blue", '轻度污染'], ["yellow", '中度污染'], ["orange", '重度污染'], ["red", '极重度污染']],
            metals: ['As', 'Cd', 'Cr', 'Cu', 'Hg', 'Ni', 'Pb', 'Zn'],
            levelIndex: {'无污染': 0, '轻度污染': 1, '中度污染': 2, '重度污染': 3, '极重度污染': 4},
            isMultiple: true,
        }
    },
    components: {
        MyContent,
        GisMap,
    },
    methods: {
        getInitData(data) {
            this.initData = data;
            this.getComputedData(data);
            this.getOutputData();
            this.getMapData();
        },
        getComputedData(data) {
            if (Object.keys(data).length === 0) return {};
            let computedData = {As: [], Cd: [], Cr: [], Cu: [], Hg: [], Ni: [], Pb: [], Zn: []}
            for (let item of data.tableData) {
                for (let metal in computedData) {
                    // console.log("metal: " + metal)
                    let temp = {}
                    for (let i of ['longitude', 'latitude', '省市', '区县', '乡镇', '村', 'pH',]) {
                        temp[i] = item[i]
                    }
                    temp['C'] = item[metal]
                    temp['B'] = backgroundData[metal]
                    temp['I'] = Math.log2(temp['C'] / (1.5 * temp['B']))
                    computedData[metal].push(temp)
                }
            }
            // console.log(computedData)
            this.computedData = computedData
        },
        getOutputData() {
            if (Object.keys(this.computedData).length === 0) return [];
            let data = [];
            let metals = ['As', 'Cd', 'Cr', 'Cu', 'Hg', 'Ni', 'Pb', 'Zn'];
            for (let metal of metals) {
                console.log(metal)
                let temp = {};
                temp['污染物'] = metal;
                temp['样点总数'] = this.computedData[metal].length;
                let count = [0, 0, 0, 0, 0];
                for (let item of this.computedData[metal]) {
                    if (item['I'] <= 0) {
                        count[0] += 1
                    } else if (item['I'] <= 1) {
                        count[1] += 1
                    } else if (item['I'] <= 2) {
                        count[2] += 1
                    } else if (item['I'] <= 5) {
                        count[3] += 1
                    } else
                        count[4] += 1
                }
                temp['样点数_0'] = count[0];
                temp['比例_0'] = (count[0] / temp['样点总数'] * 100).toFixed(2);
                temp['样点数_1'] = count[1];
                temp['比例_1'] = (count[1] / temp['样点总数'] * 100).toFixed(2);
                temp['样点数_2'] = count[2];
                temp['比例_2'] = (count[2] / temp['样点总数'] * 100).toFixed(2);
                temp['样点数_3'] = count[3];
                temp['比例_3'] = (count[3] / temp['样点总数'] * 100).toFixed(2);
                temp['样点数_4'] = count[4];
                temp['比例_4'] = (count[4] / temp['样点总数'] * 100).toFixed(2);
                data.push(temp);
            }
            // console.log(data);
            this.outputData = data
        },
        getMapData() {
            if (Object.keys(this.computedData).length === 0) return;
            let data = []
            for (let metal of this.metals) {
                let temp = []
                for (let i = 0; i < this.computedData[metal].length; ++i) {
                    let level = ""
                    let computedDataMetal = this.computedData[metal][i]
                    if (computedDataMetal['I'] <= 0) {
                        level = '无污染'
                    } else if (computedDataMetal['I'] <= 1) {
                        level = '轻度污染'
                    } else if (computedDataMetal['I'] <= 2) {
                        level = '中度污染'
                    } else if (computedDataMetal['I'] <= 5) {
                        level = '重度污染'
                    } else
                        level = '极重度污染'
                    let htmlStr = "<table><tbody><tr><td>" + ch_en[metal] + "含量（mg）：</td><td>" + computedDataMetal['C'].toFixed(2) +
                        "</td></tr><tr><td>背景值（mg）：</td><td>" + computedDataMetal['B'].toFixed(2) +
                        "</td></tr><tr><td>地累计指数：</td><td>" + computedDataMetal['I'].toFixed(2) +
                        "</td></tr><tr><td>经度：</td><td>" + computedDataMetal['longitude'].toFixed(2) +
                        "</td></tr><tr><td>纬度：</td><td>" + computedDataMetal['latitude'].toFixed(2) +
                        "</td></tr><tr><td>省市</td><td>" + computedDataMetal['省市'] + " " + computedDataMetal['区县'] +
                        "</td></tr><tr><td>区县</td><td>" + computedDataMetal['乡镇'] + " " + computedDataMetal['村'] +
                        "</td></tr></tbody></table>"
                    // let div = document.createElement('div')
                    // div.innerHTML = htmlStr
                    // div.style.float = 'right'
                    // document.body.appendChild(div)
                    temp.push([computedDataMetal['longitude'], computedDataMetal['latitude'], this.levelIndex[level], htmlStr])
                }
                data.push(temp)
            }
            this.mapData = data
            // console.log(data)
        }
    }
}
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
</style>
