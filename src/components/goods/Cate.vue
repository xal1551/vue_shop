<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加分类 按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        class="treeTable"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text=""
        border
      >
        <!-- 是否有效 -->
        <template v-slot:isok="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: #0c0;"
          ></i>
          <i class="el-icon-error" v-else style="color: #c00;"></i>
        </template>
        <!-- 排序 -->
        <template v-slot:order="scope">
          <el-tag size="mini" type="primary" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template v-slot:opt="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="showEditCateDialog(scope.row.cat_id)"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">删除</el-button>
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
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类按钮对应的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类 表单 -->
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
          <!-- options 用来指定数据源 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            :clearable="true"
            @change="parentCateChange"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改分类的对话框 -->
    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%">
      <!-- 内容主体区 -->
      <el-form
        :model="editCateForm"
        :rules="editCateFormRules"
        ref="editCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区 -->
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
      // 商品分类数据列表查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类 的数据列表，默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      // 为 table 指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示将当前列定义为 模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          // 表示将当前列定义为 模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示将当前列定义为 模板列
          type: 'template',
          // 当前列使用的模板名称
          template: 'opt'
        }
      ],
      // 控制 添加分类对话框 的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类 的表单数据对象
      addCateForm: {
        // 将要添加的分类 的名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类的等级，默认要添加的是一级分类
        cat_level: 0
      },
      // 添加分类表单 的验证规则对象
      addCateFormRules: {
        cat_name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }]
      },
      // 父级分类的列表
      parentCateList: [],
      // 选中的父级分类的 id 数组
      selectedKeys: [],
      // 指定 级联选择器 的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 修改分类对话框的显示与隐藏
      editCateDialogVisible: false,
      // 修改分类 的表单数据
      editCateForm: {},
      // 修改分类 的验证规则对象
      editCateFormRules: {
        cat_name: [{ required: true, message: '请输入邮箱地址', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)

      // console.log(res.data)
      // 接受响应的 数据列表
      this.cateList = res.data.result
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 监听 pagesize 改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听 pagenum 改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 添加分类按钮 展示对话框
    showAddCateDialog() {
      // 先获取 父级分类 的数据列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.parentCateList = res.data
    },
    // 级联选择器 选择项发生变化时触发
    parentCateChange() {
      // console.log(this.selectedKeys)
      // 如果 selectedKeys.length 大于0，证明选中了父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的 id 改变
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 父级分类的 level 改变
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击添加分类对话框的 确定按钮
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)

        this.$message.success(res.meta.msg)
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听 添加分类对话框 的关闭，重置表单
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 监听 编辑按钮的点击
    async showEditCateDialog(id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)

      this.editCateForm = res.data
      console.log(this.editCateForm)
      this.editCateDialogVisible = true
    },
    // 点击确定，修改分类
    editCate() {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, {
          cat_name: this.editCateForm.cat_name
        })

        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)

        // 关闭对话框
        this.editCateDialogVisible = false
        // 刷新用户列表
        this.getCateList()
        // 提示修改成功
        this.$message.success(res.meta.msg)
      })
    },
    // 删除分类
    async removeCateById(id) {
      // 弹框询问用户是否删除
      const confirmRes = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户点击确定，返回值为字符串 confirm;
      // 如果用户点击取消，返回值为字符串 cancel
      // console.log(confirmRes)
      if (confirmRes !== 'confirm') return this.$message.info('已取消删除')

      // 发起删除用户的请求
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      // 删除完成后，刷新用户列表
      this.getCateList()
    }
  }
}
</script>

<style lang="scss" scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
