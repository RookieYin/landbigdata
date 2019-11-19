<template>
  <div class="dashboard-editor-container">
    <el-row>
      <el-carousel trigger="click" height="300px" indicator-position="outside" type="card">
        <el-carousel-item v-for="item in 5" :key="item">
          <img
            style="	width: auto;height: auto;max-width: 100%;max-height: 100%;	"
            :src="require(`./img/${item-1}.png`)"
          />
        </el-carousel-item>
      </el-carousel>
    </el-row>
    <el-row :gutter="40" class="panel-group">
      <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
        <div class="card-panel">
          <div class="card-panel-icon-wrapper icon-people">
            <svg-icon icon-class="peoples" class-name="card-panel-icon" />
          </div>
          <div class="card-panel-description">
            <div class="card-panel-text">研究论文</div>
            <count-to :start-val="0" :end-val="102400" :duration="2600" class="card-panel-num" />篇
          </div>
        </div>
      </el-col>
      <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
        <div class="card-panel">
          <div class="card-panel-icon-wrapper icon-message">
            <svg-icon icon-class="message" class-name="card-panel-icon" />
          </div>
          <div class="card-panel-description">
            <div class="card-panel-text">国家标准</div>
            <count-to :start-val="0" :end-val="81212" :duration="3000" class="card-panel-num" />则
          </div>
        </div>
      </el-col>
      <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
        <div class="card-panel">
          <div class="card-panel-icon-wrapper icon-money">
            <svg-icon icon-class="money" class-name="card-panel-icon" />
          </div>
          <div class="card-panel-description">
            <div class="card-panel-text">实测数据</div>
            <count-to :start-val="0" :end-val="9280" :duration="3200" class="card-panel-num" />组
          </div>
        </div>
      </el-col>
      <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
        <div class="card-panel">
          <div class="card-panel-icon-wrapper icon-shopping">
            <svg-icon icon-class="shopping" class-name="card-panel-icon" />
          </div>
          <div class="card-panel-description">
            <div class="card-panel-text">防控知识</div>
            <count-to :start-val="0" :end-val="13600" :duration="3600" class="card-panel-num" />条
          </div>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-card
        class="box-card"
        v-for="cls in navData"
        v-bind:key="cls.name"
        style="margin-bottom: 10px;"
      >
        <div slot="header" class="clearfix">
          <span style="text-align: center;">{{cls.name}}</span>
        </div>
        <el-row :gutter="40">
          <el-col
            :xs="12"
            :sm="12"
            :lg="6"
            v-for="child in cls.children"
            style="margin-bottom: 10px;"
            v-bind:key="child.url"
          >
            <el-card class="item-card" style @click.native="handleClick(child.url)">{{ child.name }}</el-card>
          </el-col>
        </el-row>
      </el-card>
    </el-row>
  </div>
</template>

<script>
import CountTo from "vue-count-to";
import { isExternal } from "@/utils/validate";
const prefix = window.location.href.slice(0, -window.location.hash.length + 1);

export default {
  components: { CountTo },
  computed: {
    routes() {
      return this.$router.options.routes;
    }
  },
  methods: {
    handleClick(url) {
      window.location.href = url;
    }
  },
  data() {
    return {
      navData: []
    };
  },
  mounted() {
    let vm = this;
    window.vi = this;
    let navData = [];
    vm.routes
      .filter(e => !e.hidden && e.meta)
      .forEach(e => {
        // 对每个大类的route e，进行层次遍历

        const title = e.meta.title;
        const basePath = e.path + "/";
        let queue = [];
        let children = [];
        e.children.forEach(child => {
          queue.push({
            prePath: basePath,
            node: child
          });
        });
        while (queue.length != 0) {
          let tmp = queue.shift();
          if (!tmp.node.children) {
            children.push({
              name: tmp.node.meta.title,
              url: prefix + tmp.prePath + tmp.node.path
            });
          } else {
            tmp.node.children.forEach(child => {
              queue.push({
                prePath: tmp.prePath + tmp.node.path + "/",
                node: child
              });
            });
          }
        }
        navData.push({
          name: title,
          children
        });
      });
    vm.navData = navData;
    console.log(navData);
  }
};
</script>

<style lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .github-corner {
    position: absolute;
    top: 0px;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width: 1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}

.panel-group {
  margin-top: 18px;

  .card-panel-col {
    margin-bottom: 32px;
  }

  .card-panel {
    height: 108px;
    cursor: pointer;
    font-size: 12px;
    position: relative;
    overflow: hidden;
    color: #666;
    background: #fff;
    box-shadow: 4px 4px 40px rgba(0, 0, 0, 0.05);
    border-color: rgba(0, 0, 0, 0.05);

    &:hover {
      .card-panel-icon-wrapper {
        color: #fff;
      }

      .icon-people {
        background: #40c9c6;
      }

      .icon-message {
        background: #36a3f7;
      }

      .icon-money {
        background: #f4516c;
      }

      .icon-shopping {
        background: #34bfa3;
      }
    }

    .icon-people {
      color: #40c9c6;
    }

    .icon-message {
      color: #36a3f7;
    }

    .icon-money {
      color: #f4516c;
    }

    .icon-shopping {
      color: #34bfa3;
    }

    .card-panel-icon-wrapper {
      float: left;
      margin: 14px 0 0 14px;
      padding: 16px;
      transition: all 0.38s ease-out;
      border-radius: 6px;
    }

    .card-panel-icon {
      float: left;
      font-size: 48px;
    }

    .card-panel-description {
      float: right;
      font-weight: bold;
      margin: 26px;
      margin-left: 0px;

      .card-panel-text {
        line-height: 18px;
        color: rgba(0, 0, 0, 0.45);
        font-size: 16px;
        margin-bottom: 12px;
      }

      .card-panel-num {
        font-size: 20px;
      }
    }
  }
}

@media (max-width: 550px) {
  .card-panel-description {
    display: none;
  }

  .card-panel-icon-wrapper {
    float: none !important;
    width: 100%;
    height: 100%;
    margin: 0 !important;

    .svg-icon {
      display: block;
      margin: 14px auto !important;
      float: none !important;
    }
  }
}
.item-card {
  text-align: center;
  /*box-shadow: none;*/
}
.item-card:hover {
  cursor: pointer;
  /*box-shadow: 0 2px 6px dodgerblue, 0 0 6px rgba(0, 0, 0, .04);*/
  background-color: dodgerblue;
  color: #fff;
  font-weight: bold;
}
</style>
