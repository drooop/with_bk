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
          <el-button
            type="primary"
            @click="showAddCateDialog"
          >
            添加类别
          </el-button>
        </el-col>
      </el-row>
      <!-- Vue-Table-Tree-Grid -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        boder
        :show-row-hover="false"
        class="treeTable"
      >
        <!-- 是否有效 -->
        <template
          v-slot:isOK="scope"
        >
          <i
            v-if="scope.row.cat_deleted===false"
            style="color: lightgreen;"
            class="el-icon-success"
          />
          <i
            v-else
            style="color: red;"
            class="el-icon-error"
          />
        </template>
        <!-- 排序 -->
        <template
          v-slot:oeder="scope"
        >
          <el-tag
            v-if="scope.row.cat_level===0"
            size="mini"
          >
            一级
          </el-tag>
          <el-tag
            v-else-if="scope.row.cat_level===1"
            type="success"
            size="mini"
          >
            二级
          </el-tag>
          <el-tag
            v-else
            type="warning"
            size="mini"
          >
            三级
          </el-tag>
        </template>
        <!-- 操作 -->
        <template
          v-slot:opt="scope"
        >
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
          >
            编辑
          </el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
          >
            删除
          </el-button>
        </template>
      </tree-table>
      <!-- Pagenation -->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      class="demo-ruleForm"
    >
      <el-form
        ref="addCateFormRef"
        :model="addCateForm"
        :rules="addCateFormRules"
        label-width="100px"
      >
        <el-form-item
          label="分类名称："
          prop="cat_name"
        >
          <el-input v-model="addCateForm.cat_name" />
        </el-form-item>
        <el-form-item
          label="父级分类："
        >
          <!-- 级联选择器 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            clearable
            @change="parentCateChanged"
          />
        </el-form-item>
      </el-form>

      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="addCateDialogVisible = false"
        >确 定</el-button>
      </span>
    </el-dialog>
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

      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 将当前列指定为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'isOK'
        },
        {
          label: '排序',
          // 将当前列指定为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'oeder'
        },
        {
          label: '操作',
          // 将当前列指定为模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'opt'
        }
      ],
      addCateDialogVisible: false,
      // 添加表单的表单数据对象
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 父级分类的ID
        cat_pid: 0,
        // 分类的等级, 默认要添加的是一级分类
        cat_level: 0
      },
      // 验证规则-添加表单的表单数据
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入活动名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的Id数组
      selectedKeys: []
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
    },
    // 监听 pagesize 改变
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听 pagenum 改变
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮, 展示添加分类的对话框
    showAddCateDialog () {
      // 先获取父级列表
      this.getParentCateList()
      // 在显示对话框
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级列失败')
      }
      this.parentCateList = res.data
      this.$message.success(res.meta.msg)
    },
    // 选择项发生变化
    parentCateChanged () {
      console.log(this.selectedKeys)
    }
  }
}
</script>

<style lang="less">
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
