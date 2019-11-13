<template>
  <div>
    <upload-excel-component :on-success="handleSuccess" :before-upload="beforeUpload" />
<!--    <el-table :data="tableData" border highlight-current-row style="width: 100%;margin-top:20px;">-->
<!--      <el-table-column v-for="item of tableHeader" :key="item" :prop="item" :label="item" />-->
<!--    </el-table>-->
  </div>
</template>

<script>
import UploadExcelComponent from './upload-excel-component.vue'

export default {
  name: 'UploadExcel',
  components: { UploadExcelComponent },
  data() {
    return {
      table: {tableData: [], tableHeader: []}
    }
  },
  methods: {
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 1

      if (isLt1M) {
        return true
      }

      this.$message({
        message: 'Please do not upload files larger than 1m in size.',
        type: 'warning'
      })
      return false
    },
    handleSuccess({ results, header }) {
      this.table.tableData = results
      this.table.tableHeader = header
      this.$emit('getInitData', this.table)
      alert('上传成功！')
    }
  },
}
</script>
