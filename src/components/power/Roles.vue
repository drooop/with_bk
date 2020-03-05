<template>
  <div>
    <!-- 面包屑视图 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Card -->
    <el-card>
      <el-row>
        <el-col>
          <el-button
            type="primary"
            @click="showAddRoleDialog"
          >
            添加角色
          </el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table
        :data="rolesList"
        border
        stripe
      >
        <!-- 展开列 -->
        <el-table-column
          type="expand"
        >
          <template v-slot="scope">
            <el-row
              v-for="(item1, index1) of scope.row.children"
              :key="item1.id"
              :class="['bdbottom', 'vcenter', index1 === 0 ? 'bdtop' : '']"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  type="primary"
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                >
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right" />
              </el-col>
              <!-- 其他权限 -->
              <el-col :span="19">
                <!-- 通过for循环渲染其他权限 -->
                <el-row
                  v-for="(item2, index2) of item1.children"
                  :key="item2.id"
                  :class="['vcenter', index2 !== 0 ? 'bdtop' : '']"
                >
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right" />
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      v-for="item3 of item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          <!--
            <pre>
          {{ scope.row }}
          </pre> -->
          </template>
        </el-table-column>
        <el-table-column
          type="index"
          label="#"
        />
        <el-table-column
          prop="roleName"
          label="角色名称"
        />
        <el-table-column
          prop="roleDesc"
          label="角色描述"
        />
        <el-table-column
          label="操作"
          width="333px"
        >
          <template v-slot="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showRolesEditingDialog(scope.row)"
            >
              编辑
            </el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
            >
              删除
            </el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >
              分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 编辑角色信息的对话框 -->
    <el-dialog
      title="编辑角色"
      :visible.sync="rolesEditDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        ref="editRolesFormRef"
        :model="editRolesForm"
        label-width="70px"
      >
        <el-form-item
          label="角色名称"
        >
          <el-input
            v-model="editRolesForm.roleName"
            disabled
          />
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editRolesForm.roleDesc" />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="rolesEditDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="editRolesDesc"
        >确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        ref="treeRef"
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        default-expand-all
        :default-checked-keys="defKeys"
        node-key="id"
      />
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="updateRights"
        >确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="addRoleDialogVisible"
      width="50%"
    >
      <el-form
        ref="addRoleDialogRef"
        :model="addRoleDialogForm"
        :rules="addRoleDialogRules"
        label-width="100px"
      >
        <el-form-item
          label="角色名称"
          prop="roleName"
        >
          <el-input v-model="addRoleDialogForm.roleName" />
        </el-form-item>
        <el-form-item
          label="角色描述"
          prop="roleDesc"
        >
          <el-input v-model="addRoleDialogForm.roleDesc" />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="updateRole"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolesList: [],
      // 分配权限对话框的显示与否
      setRightDialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点id值数组
      defKeys: [],
      // 当前即将要分配权限的角色
      roleId: '',
      // 控制角色编辑对话框显示与否
      rolesEditDialogVisible: false,
      // 修改角色信息数据绑定
      editRolesForm: {},
      // 控制添加角色的对话框显示与否
      addRoleDialogVisible: false,
      // 添加用户对话框表单数据
      addRoleDialogForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加用户对话框表单验证规则对象
      addRoleDialogRules: {
        roleName: [
          { required: true, msg: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, msg: '请输入角色说明', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('角色列表获取失败')
      }
      this.rolesList = res.data
      return this.$message.success(res.meta.msg)
    },
    // 根据Id删除对应的权限
    async removeRightById (role, rightId) {
      // 弹窗询问用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      this.$message.success('确认了删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }

      // 更新列表内容的同时，防止全部重新渲染，只改当前显示数据
      role.children = res.data
      this.$message.success(res.meta.msg)
    },

    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      // 将当前操作的用户角色id
      this.roleId = role.id

      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }

      // 获取到的权限保存到data中
      this.rightsList = res.data

      // 获取当前角色的三级权限
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 递归 -> 获取所有三级权限id -> 添加到defKeys中
    getLeafKeys (node, arr) {
      // 如果当前节点不包含children属性
      if (!node.children) {
        return arr.push(node.id)
      }

      // 递归
      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭时间, 复位
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 为角色分配权限
    async updateRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      this.$message.success(res.meta.msg)
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 激活角色修改对话框
    async showRolesEditingDialog (rowData) {
      this.rolesEditDialogVisible = true
      const { data: res } = await this.$http.get('roles/' + rowData.id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.editRolesForm = res.data
    },
    // 修改角色说明
    async editRolesDesc () {
      const { data: res } = await this.$http.put(`roles/${this.editRolesForm.roleId}`,
        {
          roleName: this.editRolesForm.roleName,
          roleDesc: this.editRolesForm.roleDesc
        })

      if (res.meta.status !== 200) {
        return this.$message.error('修改角色信息失败')
      }
      this.getRolesList()
      this.$message.success('修改角色信息成功')
      this.rolesEditDialogVisible = false
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed () {
      // this.$refs.editRolesFormRef.resetFields()
    },
    // 根据角色id删除对应的角色
    async removeRoleById (id) {
      // 弹窗提示用户，询问用户是否确认删除动作
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success(res.meta.msg)
      this.getRolesList()
    },
    // 添加角色后,确认新的角色列表
    updateRole () {
      this.$refs.addRoleDialogRef.validate(async valid => {
        // 如果预校验失败,直接返回
        if (!valid) return
        // 可以发起添加角色请求
        const { data: res } = await this.$http.post('roles', this.addRoleDialogForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.$message.success(res.meta.msg)
        this.getRolesList()
        this.addRoleDialogVisible = false
      })
    },
    // 弹出添加角色对话框
    showAddRoleDialog () {
      this.addRoleDialogVisible = true
    }

  }
}
</script>

<style lang="less">
.el-tag {
  margin:7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}
</style>
