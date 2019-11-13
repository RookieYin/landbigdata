<template>
  <div style="height:100%">
    <el-row style="height:100%" :gutter="30">
      <el-col :span="20" style="height:100%">
        <div style="height:100%; position:relative;">
          <div id="map-container" style="height:100%">
            <div id="map" style="width:100%; height: 100%;"></div>
          </div>
          <div class="tuli">
            <el-card>
              <div style="text-align:center;font-weight: bold;">图例</div>
              <br />
              <el-row>
                <template v-for="(item, index) in classes">
                  <el-col :span="12" v-bind="index" :key="item[0]">
                    <img
                      :src="require('./marker/'+classes[index][0]+'.png')"
                      width="15px"
                      height="15px"
                    />
                    <span v-text="item[1]" style="margin-left:5px"></span>
                  </el-col>
                </template>
              </el-row>
            </el-card>
          </div>
        </div>
      </el-col>
      <el-col :span="4">
        <el-collapse v-model="activeName" accordion>
          <el-collapse-item title="图层管理" name="1">
            <el-tree
              ref="layerTree"
              :data="layerData"
              @check-change="LayerSelectedChange"
              show-checkbox
              node-key="id"
              :default-expanded-keys="[100, 200]"
              :default-checked-keys="[0, 2]"
            ></el-tree>
          </el-collapse-item>
          <el-collapse-item title="地图功能" name="2">
            <el-tree :data="mapMethodData" @node-click="mapMethodClick"></el-tree>
          </el-collapse-item>
          <el-collapse-item v-if="isMultiple" title="污染源标点" name="3">
            <div class="item">
              <span>污染金属类型</span>
              <el-select v-model="metalSelected" placeholder="请选择" @change="onchangemetal">
                <el-option
                  v-for="item in metalOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </div>
          </el-collapse-item>
        </el-collapse>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { map, marker, icon } from "leaflet";
import { dynamicMapLayer } from "esri-leaflet";
import "leaflet/dist/leaflet.css";

// import xgreen from "";
// import green from "@/assets/marker/green.png";
// import yellow from "@/assets/marker/yellow.png";
// import orange from "@/assets/marker/orange.png";
// import red from "@/assets/marker/red.png";
// import {HeatmapOverlay} from 'leaflet-heatmap';

export default {
  name: "GisMap",
  data() {
    return {
      currentMapPointData: [],
      ourMap: {},
      ourDM: {},
      icons: [
        "./marker/xgreen.png",
        "./marker/green.png",
        "./marker/yellow.png",
        "./marker/orange.png",
        "./marker/red.png"
      ],
      activeName: "1",
      metalSelected: "As",
      metalOptions: [],
      mapMethodData: [
        { label: "显示全图", onclick: this.showFull },
        { label: "显示污染影像图", onclick: this.showHeatMap }
      ],
      layerData: [
        {
          id: 100,
          label: "武清区（天津市）",
          children: [
            {
              id: 0,
              label: "行政区轮廓"
            },
            {
              id: 1,
              label: "武清区遥感图"
            }
          ]
        },
        {
          id: 200,
          label: "基础底图图",
          children: [
            {
              id: 2,
              label: "街道图"
            },
            {
              id: -1,
              label: "河流图"
            },
            {
              id: -2,
              label: "山脉图"
            }
          ]
        }
      ]
    };
  },
  props: {
    isMultiple: {
      type: Boolean,
      default: false
    }, //是否每种金属单独展示
    metals: {
      type: Array,
      default: []
    }, //如果金属单独展示，需要提供金属序列
    points: {
      type: Array,
      default: []
    }, //如果金属不单独展示，提供各点的数据，否则提供数组的数组，[[经,纬度,class-idx,popup-html]]
    classes: {
      type: Array,
      default: []
    } // [['green', '轻度污染'], ...]
  },
  methods: {
    reloadMap() {
      document.querySelector("#map-container").innerHTML = "";
      let mapDiv = document.createElement("div");
      mapDiv.id = "map";
      mapDiv.style = "width:100%; height: 100%;";
      document.querySelector("#map-container").append(mapDiv);

      const LAYER_URL =
        "http://192.168.56.101/arcgis/rest/services/ABC/MapServer/";
      this.ourMap = map("map").setView([39.4, 117], 10);
      this.ourDM = dynamicMapLayer({
        url: LAYER_URL,
        layers: [0, 2],
        opacity: 0.8
      });
      this.ourDM.addTo(this.ourMap);
    },
    makepoint(metal_id) {
      let arrs;

      if (this.isMultiple) {
        arrs = this.points[metal_id];
      } else {
        arrs = this.points;
      }

      for (let arr of arrs) {
        marker([arr[1], arr[0]], {
          icon: icon({
            iconUrl: require("./marker/" + this.classes[arr[2]][0] + ".png"),
            iconSize: [15, 15],
            popupAnchor: [0, -5]
          })
        })
          .addTo(this.ourMap)
          .bindPopup(arr[3]);
      }
    },
    onchangemetal(val) {
      if (this.points === undefined || this.points.length === 0) {
        alert("似乎没有上传数据");
        return;
      }
      this.reloadMap();
      this.makepoint(Number(val));
      this.ourMap.setView([39.4, 117], 10);
    },
    mapMethodClick(a) {
      a.onclick();
    },
    showFull() {
      this.ourMap.setView([39.4, 117], 10);
    },
    showHeatMap() {
      alert("heatMap!");
    },
    LayerSelectedChange() {
      let selectedLayers = this.$refs.layerTree
        .getCheckedKeys()
        .filter(e => e < 100);
      this.ourDM.setLayers(selectedLayers);
    }
  },
  mounted() {
    // todo: for debug
    window.myvm = this;

    if (this.isMultiple) {
      for (let i = 0; i < this.metals.length; ++i) {
        this.metalOptions.push({
          label: this.metals[i],
          value: i
        });
      }
    }

    this.reloadMap();
  },
  watch: {
    points() {
      this.makepoint(0);
    }
  }
};
</script>

<style scoped>
.tuli {
  position: absolute;
  width: 300px;
  height: 150px;
  bottom: 25px;
  right: 10px;
  z-index: 9999;

  opacity: 0.8;
}
.tuli /deep/ .el-col{
  /*background-color: #b8b8b8;*/
  /*color: #ffffff;*/
  line-height: 30px;
}
</style>

