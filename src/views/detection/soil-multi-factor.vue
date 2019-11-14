<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>
      土壤多因子分析法
    </template>
    <template v-slot:c-intro>
      <p>
        <strong>简介：</strong> 在单因子评价分析的基础上，利用公式，算出所有金属种类下的综合污染指数。
      </p>
      <div style="float: left;line-height: 1.8em;">
        <strong>输入：</strong>
      </div>
      <div style="float:left;margin-bottom: 20px;line-height: 1.8em;">
        每个采样点的位置信息和每个金属的单项污染指数。
      </div>
    </template>
    <div class="map" >
      <el-tabs class="mytabs" type="border-card" style="" v-model="activeName">
        <el-tab-pane label="地图" name="first">
          <GisMap style="height: 500px;"
                  :isMultiple="isMultiple"
                  :metals="metals"
                  :points="mapData"
                  :classes="classes"
          ></GisMap>            </el-tab-pane>
        <el-tab-pane label="图表" name="second" >
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
              <el-table-column label="安全">
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
              <el-table-column label="警戒线">
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
              <el-table-column label="轻度污染">
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
              <el-table-column label="中度污染">
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
              <el-table-column label="重度污染">
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
        <el-tab-pane label="详情" name="third"></el-tab-pane>
      </el-tabs>
    </div>
  </my-content>
</template>

<script>
    const standard = {Cd:[0.3,0.3,0.3,0.6], Ni:[60,70,100,190], Cu:[50,50,100,100],
        Zn:[200,200,250,300], As:[40,40,30,25], Cr:[150,150,200,250],
        Hg:[1.3,1.8,2.4,3.4],Pb:[70,90,120,170]}
    import MyContent from '@/layout/content.vue'
    import GisMap from '@/components/GisMap/index.vue'
    export default {
        name: "soil-multi-factor",
        data: function () {
            return{
                activeName: "first",
                initData: {},
                computedData: {},
                outputData: [],
                mapData: [],
                metals: ['As', 'Cd', 'Cr', 'Cu', 'Hg', 'Ni', 'Pb', 'Zn'],
                levelIndex: {'安全':0, '警戒线':1, '轻度污染':2, '中度污染':3, '重度污染':4},
                isMultiple: false,
                classes: [["green", '安全'], ["blue", '警戒线'], ["yellow", '轻度污染'], ["orange", '中度污染'], ["red", '重度污染']]
            }
        },
        components: {
            MyContent,
            GisMap
        },
        methods: {
            getInitData(data){
                this.initData = data;
                let computedData = {As: [], Cd: [], Cr: [], Cu: [], Hg: [], Ni: [], Pb: [], Zn: []}
                for(let item of data.tableData){
                    for(let metal in computedData){
                        let temp = {}
                        for(let i of ['longitude', 'latitude', '省市', '区县', '乡镇', '村', 'pH',]){
                            temp[i] = item[i]
                        }
                        temp['C'] = item[metal]
                        temp['S'] = this.getStandardData(item.pH, metal)
                        temp['P'] = temp['C'] / temp['S']
                        computedData[metal].push(temp)
                    }
                }
                let General = []
                let p_max = 0, p_avg = 0, p_general = 0
                for(let i = 0; i < computedData.As.length; ++i){
                    let temp = {}
                    for(let item of ['longitude', 'latitude', '省市', '区县', '乡镇', '村', 'pH',]){
                        temp[item] = computedData.As[i][item]
                    }
                    p_max = Math.max(computedData['As'][i]['P'], computedData['Cd'][i]['P'],computedData['Cr'][i]['P'],computedData['Cu'][i]['P'],
                        computedData['Hg'][i]['P'],computedData['Ni'][i]['P'],computedData['Pb'][i]['P'],computedData['Zn'][i]['P'],)
                    p_avg = (computedData['As'][i]['P']+computedData['Cd'][i]['P']+computedData['Cr'][i]['P']+computedData['Cu'][i]['P']+
                        computedData['Hg'][i]['P']+computedData['Ni'][i]['P']+computedData['Pb'][i]['P']+computedData['Zn'][i]['P']) / 8
                    temp['P'] = Math.sqrt((p_avg**2 + p_max**2) / 2)
                    General.push(temp)
                }
                console.log(General)
                computedData.General = General
                this.computedData = computedData
                this.getOutputData(computedData)
                this.getMapData(computedData)
            },
            getStandardData(ph, metal){
                if(ph <= 5.5)
                    return standard[metal][0];
                else if(ph <= 6.5)
                    return standard[metal][1];
                else if(ph <= 7.5)
                    return standard[metal][2];
                else
                    return standard[metal][3];
            },
            getOutputData (computedData) {
                let data = [];
                let metals = ['General', 'As', 'Cd', 'Cr', 'Cu', 'Hg', 'Ni', 'Pb', 'Zn'];
                for(let metal of metals){
                    let temp = {};
                    temp['污染物'] = metal;
                    temp['样点总数'] = computedData[metal].length;
                    let count = [0,0,0,0,0];
                    for(let item of computedData[metal]){
                        if(item['P'] <= 0.7){
                            count[0] += 1
                        }else if(item['P'] <= 1){
                            count[1] += 1
                        }else if(item['P'] <= 2){
                            count[2] += 1
                        }else if(item['P'] <= 3){
                            count[3] += 1
                        }else
                            count[4] += 1
                    }
                    temp['样点数_0'] = count[0];
                    temp['比例_0'] = (count[0]/temp['样点总数']*100).toFixed(2);
                    temp['样点数_1'] = count[1];
                    temp['比例_1'] = (count[1]/temp['样点总数']*100).toFixed(2);
                    temp['样点数_2'] = count[2];
                    temp['比例_2'] = (count[2]/temp['样点总数']*100).toFixed(2);
                    temp['样点数_3'] = count[3];
                    temp['比例_3'] = (count[3]/temp['样点总数']*100).toFixed(2);
                    temp['样点数_4'] = count[4];
                    temp['比例_4'] = (count[4]/temp['样点总数']*100).toFixed(2);
                    data.push(temp);
                }
                data[0]['污染物'] = '综合'
                this.outputData = data
            },
            getMapData(computedData){
                if(Object.keys(computedData).length === 0) return;
                let temp = []
                for(let i = 0; i < computedData['General'].length; ++i) {
                    let level = ""
                    let computedDataMetal = computedData['General'][i]
                    if(computedDataMetal['P'] <= 0.7){
                        level = '安全'
                    }else if(computedDataMetal['P'] <= 1){
                        level = '警戒线'
                    }else if(computedDataMetal['P'] <= 2){
                        level = '轻度污染'
                    }else if(computedDataMetal['P'] <= 3){
                        level = '中度污染'
                    }else
                        level = '重度污染'
                    let htmlStr = "<table><tbody><tr><td>综合污染指数：</td><td>" + computedDataMetal['P'].toFixed(2) +
                        "</td></tr><tr><td>经度：</td><td>" +computedDataMetal['longitude'].toFixed(2) +
                        "</td></tr><tr><td>纬度：</td><td>" + computedDataMetal['latitude'].toFixed(2) +
                        "</td></tr><tr><td>省市</td><td>" + computedDataMetal['省市'] + " " + computedDataMetal['区县'] +
                        "</td></tr><tr><td>区县</td><td>" + computedDataMetal['乡镇'] + " " + computedDataMetal['村'] +
                        "</td></tr></tbody></table>"
                    temp.push([computedDataMetal['longitude'],computedDataMetal['latitude'],this.levelIndex[level],htmlStr])
                }
                // console.log(temp)
                this.mapData = temp
            }
        }
    }
</script>

<style scoped>
  .mytabs /deep/ .el-tabs__item{
    width: 200px;
    text-align: center;
  }
  /*.clearfix*/
  .my-card{
    margin-bottom: 10px;
  }
  #upload-card /deep/ .el-card{
    padding: 0px !important;
  }
  .my-table /deep/ .cell{
    text-align: center;
  }
  .my-table /deep/ table{
    margin:auto;
  }
  .my-table /deep/ .el-table__empty-block{
    display: none;
  }
  .my-table /deep/ tbody tr:first-child{
    font-weight: bold;
  }
</style>
