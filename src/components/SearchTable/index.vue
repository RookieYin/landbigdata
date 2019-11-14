<template>
  <div>
    <div style="margin-bottom:10px;">
      <el-row>
        <el-col :span="6">
          <el-input placeholder="请输入内容" prefix-icon="el-icon-search" v-model="searchText"></el-input>
        </el-col>
      </el-row>
    </div>
    <el-row>
      <el-table :data="filteredData" height="400" border style="width: 100%">
        <el-table-column
          align="center"
          v-for="header in headerData"
          :key="header.prop"
          :prop="header.prop"
          :label="header.label"
          :sortable="header.sortable || false"
        ></el-table-column>
      </el-table>
    </el-row>
  </div>
</template>

<script>
export default {
  name: "SearchTable",
  props: {
    headerData: Array,
    tableData: Array
  },
  computed: {
    filteredData() {
      return this.tableData.filter(item => {
        for (let header of this.headerData) {
          if (header.searchable === undefined || header.searchable == true) {
            if (String(item[header.prop]).indexOf(this.searchText) != -1)
              return true;
          }
        }
        return false;
      });
    }
  },
  data() {
    return {
      searchText: ""
    };
  }
};
</script>

<style scoped>
</style>