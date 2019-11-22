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
        @click="dialogVisible = true"
      >添加</el-button>
    </div>

    <el-table :data="filteredData" style="width: 100%" border fit highlight-current-row>
      <el-table-column align="center" :key="1" label="资源名称" prop="name"></el-table-column>
      <el-table-column align="center" :key="2" label="类型" prop="type">
        <template v-slot="{row}">{{TypeName[row.type]}}</template>
      </el-table-column>
      <el-table-column
        :key="3"
        label="操作"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template v-slot="{row}">
          <el-button type="primary" size="mini" @click="downloadByUuid(row.uuid)">下载</el-button>
          <el-button size="mini" type="danger" @click="deleteById(row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog title="添加资源" :visible.sync="dialogVisible" :before-close="handleClose">
      <el-form :inline="true" :model="form" class="demo-form-inline">
        <el-form-item label="上传文件">
          <el-upload
            action
            :multiple="false"
            :limit="1"
            :on-exceed="handleExceed"
            :file-list="fileList"
            :before-upload="beforeUpload"
          >
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>

        <el-form-item label="名称">
          <el-input v-model="form.name"></el-input>
        </el-form-item>

        <el-form-item label="资源类型">
          <el-select v-model="form.type" placeholder="选择资源类型">
            <el-option v-for="item in TypeList" :key="item" :value="item" :label="TypeName[item]" />
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="handleAdd">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";
import user from "../../../mock/user";

const knowledgeTypeOptions = [
  { key: 0, display_name: "所有" },
  { key: 1, display_name: "研究论文" },
  { key: 2, display_name: "国家标准" },
  { key: 3, display_name: "实测数据" },
  { key: 4, display_name: "防控知识" }
];

const TypeList = [
  "REASEARCH_PAPER",
  "NATIONAL_STANDARD",
  "RAW_DATA",
  "PREVENTION_TIPS"
];
const TypeName = {
  REASEARCH_PAPER: "研究论文",
  NATIONAL_STANDARD: "国家标准",
  RAW_DATA: "实测数据",
  PREVENTION_TIPS: "防控知识"
};

export default {
  computed: {
    filteredData() {
      let data = this.tableData.filter(item => {
        if (
          this.userQuery.type != 0 &&
          TypeList[this.userQuery.type - 1] != item.type
        ) {
          return false;
        }
        if (String(item["name"]).indexOf(this.userQuery.keyword) !== -1)
          return true;
        return false;
      });
      // this.total = data.length;
      return data;
    }
  },
  data() {
    return {
      fileList: undefined,
      form: {
        name: "",
        type: "",
        file: undefined
      },
      dialogVisible: false,
      TypeName,
      TypeList,
      userQuery: {
        keyword: "",
        type: 0
      },
      knowledgeTypeOptions,
      headerData: [
        {
          prop: "name",
          label: "资源名称",
          sortable: false
        },
        {
          prop: "type",
          label: "类型",
          sortable: false
        }
      ],
      tableData: []
    };
  },
  methods: {
    beforeUpload(file) {
      this.form.name = file.name;
      this.form.file = file;
      return false;
    },
    fileChange(file, fileList) {
      console.log(arguments);
      console.log(this.fileList);
      this.form.file = fileList[0];
    },
    handleExceed() {
      alert("文件数量超出限制！");
    },
    handleAdd() {
      console.log(this.form);
      let formData = new FormData();
      formData.append("name", this.form.name);
      formData.append("type", this.form.type);
      formData.append("file", this.form.file);
      axios
        .post("http://localhost:8080/api/knowledge", formData, {
          headers: { "Content-Type": "multipart/form-data" }
        })
        .then(function(response) {
          console.log(response);
        });
      this.form = {
        name: "",
        type: "",
        file: undefined
      };
      this.dialogVisible = false;
      location.reload();
    },
    handleClose() {},
    handleFilter() {
      alert("filter");
    },
    handleCreate() {},
    downloadByUuid(uuid) {
      window.open(`http://localhost:8080/api/knowledge/${uuid}`);
    },
    deleteById(id) {
      console.log(id);
      axios.delete(`http://localhost:8080/api/knowledge/${id}`);
      location.reload();
    },
    fetchData() {
      axios.get("http://localhost:8080/api/knowledge").then(response => {
        console.log(response.data);
        this.tableData = response.data;
      });
    }
  },
  created() {
    this.fetchData();
  }
};
</script>

<style scoped>
</style>