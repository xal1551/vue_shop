<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 提示区 -->
      <el-alert title="编辑商品信息" type="info" center show-icon :closable="false"> </el-alert>
      <!-- 步骤条区域 -->
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- tab栏区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs :tab-position="'left'" v-model="activeIndex" @tab-click="tabClick">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="editForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="editForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="editForm.goods_number" type="number"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染商品参数表单的 item项 -->
            <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_value">
                <el-checkbox
                  :label="cb"
                  v-for="(cb, i) in item.attr_value"
                  :key="i"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
              <el-input v-model="item.attr_value"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 表示图片需要上传到的后台 API 地址 -->
            <el-upload
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">新增图片</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器 -->
            <quill-editor v-model="editForm.goods_introduce"></quill-editor>
            <!-- 修改完成的按钮 -->
            <el-button type="primary" class="btnedit" @click="editGoods">修改完成</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片预览 的对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewDialogVisible" width="50%">
      <img :src="previewPath" alt="" class="preview_img" />
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 步骤条处于激活状态的索引
      activeIndex: '0',
      // 添加商品的表单数据对象
      editForm: {},
      // editForm: {
      //   goods_name: '',
      //   goods_price: 0,
      //   goods_weight: 0,
      //   goods_number: 0,
      //   // 商品所属的分类 数组
      //   goods_cat: [],
      //   // 图片临时路径的数组
      //   pics: [],
      //   // 商品的详情描述
      //   goods_introduce: '',
      //   attrs: []
      // },
      // 添加商品的表单验证对象
      editFormRules: {
        goods_name: [{ required: true, message: '请输入商品名称', trigger: 'blur' }],
        goods_price: [{ required: true, message: '请输入商品价格', trigger: 'blur' }],
        goods_weight: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_number: [{ required: true, message: '请输入商品重量', trigger: 'blur' }],
        goods_cat: [{ required: true, message: '请选择商品分类', trigger: 'blur' }]
      },
      // 商品参数 列表数组
      manyTableData: [],
      // 商品属性 列表数组
      onlyTableData: [],
      // 图片上传到的url地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传组件的 headers 请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 图片预览路径
      previewPath: '',
      // 控制图片预览对话框的显示与隐藏
      previewDialogVisible: false
      // fileList: []
    }
  },
  created() {
    this.getGoods()
  },
  methods: {
    // 获取商品数据
    async getGoods() {
      // console.log(this.$router.currentRoute.query.id)
      const { data: res } = await this.$http.get('goods/' + this.$router.currentRoute.query.id)

      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)

      console.log(res.data)
      // 接受响应的 数据列表
      this.editForm = res.data
    },
    // 单击切换tab事件
    tabClick() {
      // console.log(this.activeIndex)
      if (this.activeIndex === '1') {
        // 证明访问的是 商品参数 面板
        // console.log(this.editForm.attrs)
        this.manyTableData = this.editForm.attrs.filter(item => item.attr_sel === 'many')
        // console.log(this.manyTableData)
        this.manyTableData.forEach(item => {
          if (typeof item.attr_value === 'string') {
            item.attr_value = item.attr_value.split(' ')
          }
          // console.log(item)
        })
      } else if (this.activeIndex === '2') {
        // 证明访问的是 商品属性 面板

        this.onlyTableData = this.editForm.attrs.filter(item => item.attr_sel === 'only')
        // console.log(this.onlyTableData)
      }
      // else if (this.activeIndex === '3') {
      //   // console.log(this.editForm.pics)
      //   this.editForm.pics.forEach(item => {
      //     const obj = {}
      //     var name = item.pics_id
      //     var url = item.pics_mid
      //     this.$set(obj, 'name', name)
      //     this.$set(obj, 'url', url)
      //     // console.log(obj)
      //     this.fileList.push(obj)
      //   })
      //   // console.log(this.fileList)
      // }
    },
    // 处理 图片预览
    handlePreview(file) {
      // console.log(file)
      this.previewPath = file.response.data.url
      this.previewDialogVisible = true
    },
    // 处理 移出图片 的操作
    handleRemove(file) {
      // console.log(file)
      // 1. 获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2. 从 pics 数组中，找到这个图片对应的索引值
      const i = this.addForm.pics.findIndex(item => item.pic === filePath)
      // 3. 调用数组的 splice 方法，把图片信息对象，从 pics 数组中移除
      this.addForm.pics.splice(i, 1)
      // console.log(this.addForm)
    },
    // 监听图片上传成功的事件
    handleSuccess(response) {
      // console.log(response)
      // 1.拼接得到一个图片信息对象
      const picInfo = { pic: response.data.tmp_path }
      // 2.将图片信息对象， push到 pics 数组中
      this.editForm.pics.push(picInfo)
      // console.log(this.editForm)
    },
    // 编辑完成按钮的点击事件
    editGoods() {
      // console.log(this.editForm)
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的表单项')
        // 执行添加的业务逻辑

        // 处理动态参数
        this.editForm.attrs = []
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_value.join(' ')
          }
          this.editForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_value
          }
          this.editForm.attrs.push(newInfo)
        })
        // 发起请求
        // 注意：商品名称必须是唯一的
        const { data: res } = await this.$http.put('goods/' + this.editForm.goods_id, this.editForm)

        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        // console.log(res.data)
        this.$message.success('编辑商品成功')
        this.$router.push('/goods')
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}

.preview_img {
  width: 100%;
}

.btnedit {
  margin-top: 15px;
}
</style>
