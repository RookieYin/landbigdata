<template>
  <my-content @getInitData="getInitData">
    <template v-slot:c-title>重金属污染聚类溯源</template>
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
        <el-tab-pane label="相关性矩阵" name="first">
          <table class="matrix">
            <thead>
              <tr>
                <th colspan="9">相关系数分析结果表</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>相关系数</td>
                <td v-for="metal in metals" v-bind:key="metal" v-text="metal" />
              </tr>
              <tr v-for="(metal, index) in metals" v-bind:key="10*index">
                <td v-text="metal" />
                <td
                  v-for="(text,i) in matrix[index]"
                  :key="i"
                  v-text="text"
                  :class="{ high: high(matrix[index][i]), higher: higher(matrix[index][i])}"
                />
              </tr>
            </tbody>
          </table>
        </el-tab-pane>
        <el-tab-pane label="污染溯源" name="second">
          <search-table :headerData="headerData" :tableData="tableData" />
        </el-tab-pane>
      </el-tabs>
    </div>
  </my-content>
</template>

<script>
import MyContent from "@/layout/content";
import SearchTable from "@/components/SearchTable/index";

const Rules = [
  {
    id: 1,
    name: "工业三废污染",
    description:
      "电镀生产排出的废水或废液的处理。电镀工厂排出的废水和废液中含有大量金属离子如：铬、镐、镍，含氰，含酸，含碱，一般常含有有机添加剂。电镀生产中产生的废水成分非常复杂，除含氰(CN-)和酸碱外，重金属是电镀业潜在危害性极大的污水类别，这些物质严重危害环境和人类身体健康，除去污水污染，工业废气通过雨水沉降进入土壤也是土壤重金属含量显著提高",
    features: [2, 2, 1, 1, 0, 1, 1, 1]
  },
  {
    id: 2,
    name: "化肥（过磷酸盐）与污泥施用",
    description:
      "重金属元素是肥料中报道最多的污染物质，尤其是磷肥。磷肥的生产原料磷矿石，天然伴生镉，不当施用磷肥会造成土壤镉污染这已经获得国际公认，为Cd为主要的污染来源",
    features: [1, 0, 0, 0, 1, 2, 1, 1]
  },
  {
    id: 3,
    name: "公路污染",
    description:
      "以公路、铁路为中间成条带状散布的重金属污染土壤首要是因为汽车尾气的排放、汽车轮胎磨损发生的很多含重金属的有害气体和粉尘的沉降所导致的，污染元素中首要为Pb、Cu、Zn等元素。这些物质随风飘落，进入土壤中导致重金属污染。",
    features: [0, 2, 1, 1, 0, 1, 0, 2]
  },
  {
    id: 4,
    name: "农药",
    description:
      "杀虫剂、杀菌剂、杀鼠剂和除草剂中Cu Zn和As污染可能性较大。滥用农药在造成残毒污染的同时，某些农药在组成中含有汞、砷、铜、锌等重金属，还带来了土壤的重金属污染，如赛力散、西力升、稻脚青、苏农6401、砷酸钳、亚砷酸铅",
    features: [0, 1, 2, 2, 1, 0, 1, 1]
  },
  {
    id: 5,
    name: "农用地膜",
    description:
      "农用地膜生产过程中加入了含有Cd和Pb热稳定剂，有部分可能造成Cd Pb污染",
    features: [0, 0, 0, 0, 0, 1, 0, 1]
  },
  {
    id: 6,
    name: "畜禽粪便及其堆肥产品",
    description:
      "在畜禽养殖过程中，除了使用含Cu和Zn的饲料添加剂，有时还用含As、Cd、Cr、Pb和Hg的添加剂，畜禽粪便中重金属质量分数与饲料直接相关。",
    features: [1, 0, 1, 1, 1, 1, 1, 1]
  }
];

export default {
  components: { MyContent, SearchTable },
  data() {
    return {
      activeName: "first",
      metals: ["Cr", "Ni", "Cu", "Zn", "As", "Cd", "Hg", "Pb"],
      metalsOver: ["Cr", "Ni", "Cu", "Zn", "As", "Cd", "Hg", "Pb"],
      matrix: [],
      headerData: [],
      tableData: []
    };
  },
  methods: {
    getInitData(data) {
      let vm = this;
      let tableData = data.tableData;
      let tableHeader = data.tableHeader;
      const n = tableData.length;
      const m = vm.metals.length;

      // 每个样本（X）的 金属含量
      let Dnm = [];
      for (let data of tableData) {
        let tmp = [];
        for (let metal of vm.metals) {
          tmp.push(data[metal]);
        }
        Dnm.push(tmp);
      }
      // x，y score 归一二维数组
      let Sij = [];
      const Sum = (n * (n - 1)) / 2;
      for (let i = 0; i < m; ++i) Sij.push([]);
      for (let i = 0; i < m; ++i) {
        for (let j = 0; j < m; ++j) {
          if (i >= j) {
            Sij[i][j] = "-";
            continue;
          }
          let score = 0;
          for (let k = 0; k < n; ++k) {
            for (let l = k + 1; l < n; ++l) {
              if (
                (Dnm[k][i] > Dnm[l][i] && Dnm[k][j] > Dnm[l][j]) ||
                (Dnm[k][i] < Dnm[l][i] && Dnm[k][j] < Dnm[l][j])
              ) {
                score = score + 1;
              }
            }
          }
          Sij[i].push(((score / Sum) * 100).toFixed(2) + "%");
        }
      }
      vm.matrix = Sij;
      vm.getResourcesData();
    },
    higher(a) {
      return Number(a.slice(0, -1)) > 80;
    },
    high(a) {
      const val = Number(a.slice(0, -1));
      return val < 80 && val > 70;
    },
    getResourcesData() {
      let vm = this;
      let headerData = [
        { prop: "type", label: "污染源类型", sortable: true }, //
        { prop: "name", label: "名称" }, //
        { prop: "description", label: "描述" } //
      ];
      let tableData = [
        {
          type: "高可能性污染源",
          name: "工业三废污染",
          description:
            "电镀生产排出的废水或废液的处理。电镀工厂排出的废水和废液中含有大量金属离子如：铬、镐、镍，含氰，含酸，含碱，一般常含有有机添加剂。电镀生产中产生的废水成分非常复杂，除含氰(CN-)和酸碱外，重金属是电镀业潜在危害性极大的污水类别，这些物质严重危害环境和人类身体健康，除去污水污染，工业废气通过雨水沉降进入土壤也是土壤重金属含量显著提高"
        }
      ];

      let highRuleIdxs = [];
      let mediumRuleIdxs = [];
      // high
      for (let rule of Rules) {
        const features = rule.features;
        let highMetal = [];
        features.forEach((val, idx) => {
          if (val === 2) highMetal.push(idx);
        });
        if (highMetal.length === 0) continue;
        if (
          highMetal.length === 1 &&
          vm.metalsOver.includes(vm.metals[highMetal[0]])
        )
          highRuleIdxs.push(rule.id - 1);
        else {
          outer: for (let i = 0; i < highMetal.length; ++i) {
            for (let j = i; j < highMetal.length; ++j) {
              if (
                vm.metalsOver.includes(vm.metals[highMetal[i]]) &&
                vm.metalsOver.includes(vm.metals[highMetal[j]]) &&
                Number(vm.matrix[i][j].slice(0, -1)) > 80
              ) {
                highRuleIdxs.push(rule.id - 1);
                break outer;
              }
            }
          }
        }
      }

      // medium
      for (let rule of Rules) {
        const features = rule.features;
        let highMetal = [];
        features.forEach((val, idx) => {
          if (val === 1) highMetal.push(idx);
        });
        if (highMetal.length === 0) continue;
        if (
          highMetal.length === 1 &&
          vm.metalsOver.includes(vm.metals[highMetal[0]])
        )
          mediumRuleIdxs.push(rule.id - 1);
        else {
          outer: for (let i = 0; i < highMetal.length; ++i) {
            for (let j = i; j < highMetal.length; ++j) {
              if (
                vm.metalsOver.includes(vm.metals[highMetal[i]]) &&
                vm.metalsOver.includes(vm.metals[highMetal[j]]) &&
                Number(vm.matrix[i][j].slice(0, -1)) > 70
              ) {
                mediumRuleIdxs.push(rule.id - 1);
                break outer;
              }
            }
          }
        }
      }

      mediumRuleIdxs = mediumRuleIdxs.filter(
        v => highRuleIdxs.indexOf(v) === -1
      );

      console.log(highRuleIdxs);
      console.log(mediumRuleIdxs);

      for (let hidx of highRuleIdxs) {
        tableData.push({
          type: "高可能性污染源",
          name: Rules[hidx].name,
          description: Rules[hidx].description
        });
      }

      for (let lidx of mediumRuleIdxs) {
        tableData.push({
          type: "可能性污染源",
          name: Rules[lidx].name,
          description: Rules[lidx].description
        });
      }

      vm.headerData = headerData;
      vm.tableData = tableData;
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

.matrix {
  width: 800px;
  text-align: center;
  color: #303133;
  margin: 0 auto;
  border-spacing: 30px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}

.high {
  font-weight: bold;
  color: orangered;
}

.higher {
  font-weight: 900;
  color: red;
}
</style>