<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">
            添加角色
          </el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot:default="scope">
            <el-row
              :class="['bdbottom', , i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>

                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <!-- eslint-disable-next-line -->
          <template v-slot:default="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-search"
              @click="showRoleDialog(scope.row.id)"
            >
              编辑
            </el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRolesById(scope.row.id)"
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

    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="50%"
      @close="addRoleDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="addRoleForm"
        :rules="addRoleFormRules"
        ref="addRoleFormRef"
        label-width="70px"
      >
        <el-form-item label-width="100px" label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label-width="100px" label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加角色对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="editRoleDialogVisible"
      width="50%"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="editRoleForm"
        :rules="editRoleFormRules"
        ref="editRoleFormRef"
        label-width="70px"
      >
        <el-form-item label-width="100px" label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label-width="100px" label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形控件 -->
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      roleList: [],
      //   控制修改用户的对话框显示与隐藏
      addRoleDialogVisible: false,
      // 添加角色表单数据
      addRoleForm: {
        roleName: '',
        roleDesc: '',
      },
      //   添加表单验证规则
      addRoleFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          {
            min: 0,
            max: 20,
            message: '长度在 0 到 20 个字符',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: false, message: '请输入角色描述', trigger: 'blur' },
        ],
      },

      //   控制修改角色的对话框显示和隐藏
      editRoleDialogVisible: false,
      //  查询到的角色信息
      editRoleForm: {},
      editRoleFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '长度在 2 到 10 个字符',
            trigger: 'blur',
          },
        ],
        roleDesc: [
          { required: false, message: '请输入角色描述', trigger: 'blur' },
        ],
      },
      //   控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      //   所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children',
      },
      //   默认选中的节点id值
      defKeys: [],
      //   当前即将分配角色的id
      roleId: '',
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.roleList = res.data
    },
    // // 监听添加用户对话框的关闭事件
    addRoleDialogClosed() {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 添加角色
    async addRoles() {
      this.$refs.addRoleFormRef.validate(async (valid) => {
        if (!valid) return
        // 通过验证发起请求
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 200) {
          this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        // 关闭显示框
        this.addRoleDialogVisible = false
        // 重新获取角色列表数据
        this.getRoleList()
      })
    },
    // 监听角色修改框的关闭事件
    // editRoleDialogClosed() {
    //     // 不使用效果更好
    // //   this.$refs.editRoleFormRef.resetFields()
    // },

    // 展示编辑角色的信息
    async showRoleDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色失败')
      }
      this.editRoleForm = res.data
      this.editRoleDialogVisible = true
    },
    // 修改角色信息并提交
    editRoleInfo() {
      this.$refs.editRoleFormRef.validate(async (valid) => {
        if (!valid) return
        // 发起修改角色信息的数据请求
        const { data: res } = await this.$http.put(
          `roles/${this.editRoleForm.roleId}`,
          {
            roleName: this.editRoleForm.roleName,
            roleDesc: this.editRoleForm.roleDesc,
          }
        )
        if (res.meta.status !== 200)
          return this.$message.error('更新角色信息失败')
        //   关闭对话框
        this.editRoleDialogVisible = false
        // 刷新数据
        this.getRoleList()
        // 提示修改成功
        this.$message.success('更新角色信息成功')
      })
    },

    // 更具roleId删除对应角色信息
    async removeRolesById(id) {
      // 弹窗询问是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => {
        return err
      })

      //取消删除
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')

      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.message.error('删除角色失败')
      }
      this.$message.success('删除成功')
      this.getRoleList()
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm(
        '此操作将删除该权限, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => {
        return err
      })
      //取消删除
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')

      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.message.error('删除权限失败失败')
      }
      //   该操作会重新渲染页面，不建议使用
      //   this.getRoleList()
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }

      // 把获取到的权限数据保存到data中
      this.rightslist = res.data
      //   console.log(this.rightslist)
      // 递归获取三级节点的id
      this.getleafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取所有角色三级权限的id，并保存到数组中
    getleafKeys(node, arr) {
      // 如果当前node不包含children，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach((item) => {
        this.getleafKeys(item, arr)
      })
    },

    // 监听分配对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        this.$refs.treeRef.getCheckedKeys(),
        this.$refs.treeRef.getHalfCheckedKeys(),
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功！')
      this.getRoleList()
      this.setRightDialogVisible = false
    },
  },
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
