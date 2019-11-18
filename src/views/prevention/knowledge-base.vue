<template>
  <div class="app-container">
    <div class="filter-container">
      <el-select
        v-model="userQuery.type"
        placeholder="类型"
        clearable
        class="filter-item"
        style="width: 130px"
      >
        <el-option
          v-for="item in knowledgeTypeOptions"
          :key="item.key"
          :label="item.display_name+'('+item.key+')'"
          :value="item.key"
        />
      </el-select>
      <el-input
        v-model="userQuery.keyword"
        placeholder="输入关键词搜索"
        style="width: 200px;"
        class="filter-item"
        @keyup.enter.native="handleFilter"
      />
      <el-button class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">搜索</el-button>
      <el-button
        class="filter-item"
        style="margin-left: 10px;"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >添加</el-button>
    </div>
    <el-table style="width: 100%" border fit highlight-current-row></el-table>
  </div>
</template>

<script>
const knowledgeTypeOptions = [
  { key: 0, display_name: "所有" },
  { key: 1, display_name: "研究论文" },
  { key: 2, display_name: "国家标准" },
  { key: 3, display_name: "实测数据" },
  { key: 4, display_name: "防控知识" }
];

export default {
  data() {
    return {
      userQuery: {
        keyword: undefined,
        type: undefined
      },
      knowledgeTypeOptions
    };
  },
  methods: {
    handleFilter() {
      alert("filter");
    },
    handleCreate() {
      alert("create");
    }
  }
};
</script>

<style scoped>
</style>