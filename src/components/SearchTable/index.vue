<template>
  <div>
    <div style="margin-bottom:10px;">
      <el-row :gutter="10">
        <el-col :span="6">
          <el-input placeholder="请输入内容" prefix-icon="el-icon-search" v-model="inputText" clearable></el-input>
        </el-col>
        <el-col :span="3">
          <el-button
            icon="el-icon-search"
            @click="()=>{this.searchText = inputText; this.currentPage = 1;}"
          ></el-button>
        </el-col>
        <el-col :span="7" :offset="8" style="padding: 10px;font-size: 15px;color:#909399">
          <strong>当前搜索:</strong>
          <div
            style="border-bottom: solid 1px;display: inline-block;width:180px;text-align: center;"
          >{{searchText!=="" ? searchText:"未输入查询"}}</div>
        </el-col>
      </el-row>
    </div>
    <el-row>
      <el-table
        :data="filteredData.slice(this.currentPage*this.pageSize-this.pageSize, this.currentPage*this.pageSize)"
        height="400"
        border
        style="width: 100%"
      >
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
    <el-row>
      <el-pagination
        background
        layout="total, ->, prev, pager, next, jumper"
        :page-size="pageSize"
        :total="total"
        :current-page="currentPage"
        @current-change="currentPageChange"
        style="margin-top:10px;"
      ></el-pagination>
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
      let data = this.tableData.filter(item => {
        for (let header of this.headerData) {
          if (header.searchable === undefined || header.searchable === true) {
            if (String(item[header.prop]).indexOf(this.searchText) !== -1)
              return true;
          }
        }
        return false;
      });
      this.total = data.length;
      return data;
    }
  },
  data() {
    return {
      searchText: "",
      inputText: "",
      total: 0,
      currentPage: 1,
      pageSize: 7
    };
  },
  methods: {
    currentPageChange(page) {
      this.currentPage = page;
    }
  }
};
</script>

<style scoped>
</style>
