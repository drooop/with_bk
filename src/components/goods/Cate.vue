<template>
  <div>
    <!-- 面包屑视图 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Card -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">
            添加类别
          </el-button>
        </el-col>
      </el-row>
      <!-- Vue-Table-Tree-Grid -->
      <tree-table
        :data="cateList"
        :columns="columns"
      />
      <!-- Pagenation -->
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 存储商品分类列表
      cateList: [],
      queryInfo: {
        pagenum: 1,
        pagesize: 5
      },
      total: 0,

      columns: [{
        label: '分类名称',
        prop: 'cat_name'
      }]
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      this.cateList = res.data.result
      console.log(res.data)
      this.total = res.data.total
      this.$message.success('获取分类列表成功')
    }
  }
}
</script>

<style lang="less">

</style>
