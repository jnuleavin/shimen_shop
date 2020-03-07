<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button @click="showAddCateDialog" type="primary">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#"
      border :show-row-hover="false">
        <template slot="isOK" slot-scope="scope">
          <i v-if="scope.row.cat_deleted === false" class="el-icon-success"  style="color: lightgreen;"></i>
          <i v-else class="el-icon-error"  style="color: red;"></i>
        </template>
        <template slot="level" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level == 0">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level == 1" type="success">二级</el-tag>
          <el-tag v-else type="warning">三级</el-tag>
        </template>
        <template slot="handle" slot-scope="scope">
          <!-- 编辑 -->
          <el-button @click="showEditDialog(scope.row.cat_id)" type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
          <!-- 删除 -->
          <el-button @click="deleteCate(scope.row.cat_id)" type="danger" size="mini" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-size="queryInfo.pagesize"
            layout="total, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      @close="addCateDialogClosed"
      title="添加商品分类"
      :visible.sync="addCateDialogVisible"
      width="50%">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="90px">
        <el-form-item label="分类名称: " prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类: ">
          <!-- 级联选择器 -->
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            expand-trigger="hover"
            :options="parentCateList"
            :props="cascaderProps"
            v-model="selectedOptions"
            @change="parentCateChanged"
            clearable
            change-on-select
            show-all-levels
            size="medium">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类的对话框 -->
    <el-dialog
      @close="editCateDialogClosed"
      title="编辑商品分类"
      :visible.sync="editDialogVisible"
      width="50%">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="90px">
        <el-form-item label="分类名称: " prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        type: 3,
        // 当前的页数
        pagenum: 1,
        // 当前每页显示多少数据
        pagesize: 5
      },
      cateList: [],
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '分类名称',
          // 指定模板列
          type: 'template',
          template: 'isOK'
        },
        {
          label: '排序',
          // 指定模板列
          type: 'template',
          template: 'level'
        },
        {
          label: '操作',
          // 指定模板列
          type: 'template',
          template: 'handle'
        }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectedOptions: [],
      editDialogVisible: false,
      editCateForm: {
        cat_name: ''
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ]
      },
      curCateId: 0
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cateList = res.data.result
      this.total = res.data.total
      // console.log(res.data)
    },
    // 监听pagesize改变的事件
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听页码值改变的事件
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.parentCateList = res.data
      // console.log(res.data)
    },
    parentCateChanged () {
      // console.log(this.selectedOptions)
      if (this.selectedOptions.length > 0) {
        this.addCateForm.cat_pid = this.selectedOptions[this.selectedOptions.length - 1]
      } else {
        this.addCateForm.cat_pid = 0
      }
      this.addCateForm.cat_level = this.selectedOptions.length
    },
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        // console.log(this.addCateForm)
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        // console.log(res)
        this.$message.success(res.meta.msg)
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.selectedOptions = []
    },
    async showEditDialog (id) {
      this.curCateId = id
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.editCateForm.cat_name = res.data.cat_name
      this.editDialogVisible = true
    },
    editCate () {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.curCateId}`, this.editCateForm)
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        // console.log(res)
        this.$message.success(res.meta.msg)
        this.getCateList()
        this.editDialogVisible = false
      })
    },
    editCateDialogClosed () {
      this.$refs.editCateFormRef.resetFields()
    },
    deleteCate (id) {
      this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`categories/${id}`)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success(res.meta.msg)
        this.getCateList()()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable{
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}
</style>
