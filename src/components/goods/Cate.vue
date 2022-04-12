<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">
            添加分类
          </el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        border
        class="treeTable"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="sucess"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
          >
            二级
          </el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>

        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="showEditCateDialog(scope.row)"
          >
            编辑
          </el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteCatById(scope.row)"
            
          >
            删除
          </el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->

    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类的表单 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>

        <el-form-item label="父级分类：">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            v-model="selectKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
            ref="cascader"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 编辑分类的表单 -->
      <el-form
        :model="editCateForm"
        :rules="editCateFormRules"
        ref="editCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询所需数据
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 总数据
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
        },
        {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isok',
        },
        {
          label: '排序',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'order',
        },
        {
          label: '操作',
          // 表示将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'opt',
        },
      ],
      //   控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      //   添加分类的表单数据对象
      addCateForm: {
        //   将要添加分类的名称
        cat_name: '',
        cat_pid: 0,
        // 当前添加的分类等级默认为一级分类
        cat_level: 0,
      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: {
          required: true,
          message: '请输入分类名称',
          trigger: 'blur',
        },
      },
      // 父级分类的列表
      parentCateList: [],
      //   指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true,
      },
      //   选中的父级分类Id数组
      selectKeys: [],

      //   编辑分类的表单数据
      editCateForm: {},
      // 编辑分类表单的验证规则对象
      editCateFormRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            trigger: 'blur',
          },
        ],
      },
      //   编辑框是否打开
      editCateDialogVisible: false,
      // 创建一个对象来保存编辑框中的数据
      editCatForm: {},
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      //   把数据列表赋值给catelist
      this.catelist = res.data.result
      //   为总数据条数赋值
      this.total = res.data.total
    },
    // 监听 pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮， 展示添加分类对话框
    showAddCateDialog() {
      // 获取父级分类数据
      this.getParentCateList()
      // 展示出对话框
      this.addCateDialogVisible = true
    },

    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 },
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败')
      }
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChanged() {
      // 如果selectedKeys数组中的length大于0，证明选择父级分类
      // 反之，就说明没有选中任何父级分类
      if (this.selectKeys.length > 0) {
        //   父级分类的id
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
        //   为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectKeys.length
        this.$refs.cascader.toggleDropDownVisible()
        return
      } else {
        //   父级分类的id
        this.addCateForm.cat_pid = 0
        //   为当前分类的等级赋值
        this.addCateForm.cat_level = 0
        // 选中后关闭下拉级联
        this.$refs.cascader.toggleDropDownVisible()
      }
    },

    // 点击按钮，添加新的分类
    addCate() {
      this.$refs.addCateFormRef.validate(async (valid) => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败了！')
        }
        this.$message.success('添加消息成功了！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    async showEditCateDialog(value) {
      const { data: res } = await this.$http.get('categories/' + value.cat_id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询失败！')
      }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    editCate() {
      this.$refs.editCateFormRef.validate(async (valid) => {
        if (!valid) return
        // 发起修改分类的信息的数据请求
        const { data: res } = await this.$http.put(
          `categories/${this.editCateForm.cat_id}`,
          {
            cat_name: this.editCateForm.cat_name,
          }
        )
        if (res.meta.status !== 200)
          return this.$message.error('更新分类信息失败')

        //   关闭对话框
        this.editCateDialogVisible = false
        // 刷新数据
        this.getCateList()
        // 提示更新成功
        this.$message.success('更新分类信息成功')
      })
    },
    // 监听修改分类对话框
    editDialogClosed() {
      this.$refs.editCateFormRef.resetFields()
    },
    // 删除分类
   async deleteCatById(value) {
        // 弹窗询问是否删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }
      ).catch((err) => {
        return err
      })
      // 如果用户确认删除，则返回值为字符串confrim 反之cancel
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete('categories/' + value.cat_id)
      if (res.meta.status !== 200) {
        return this.message.error('删除用户失败')
      }
      this.$message.success('删除成功')
      this.getCateList()
   },
  },
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
