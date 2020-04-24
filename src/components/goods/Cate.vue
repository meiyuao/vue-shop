<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片区域 -->
    <el-card>
      <!-- 添加分类按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 分类表格  -->
      <tree-table
        class="treeTable"
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <template slot="is_ok" slot-scope="scope">
          <i
            :class="!scope.row.cat_deleted?'el-icon-success':'el-icon-error'"
            :style="{color: !scope.row.cat_deleted?'lightgreen':'red'}"
          ></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag v-else type="warning">三级</el-tag>
        </template>
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <div class="PaginationBlock">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[1, 2, 5, 10]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        ></el-pagination>
      </div>
    </el-card>

    <!-- 添加商品分类对话框 -->
    <el-dialog
      title="添加商品分类"
      :visible.sync="addCateDialogVisible"
      width="40%"
      @close="addCateDialogClose"
    >
      <el-form :model="addCateForm" :rules="addCateRules" ref="addCateFormRef">
        <el-form-item label="分类名称" prop="cat_name" label-width="80px">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类" prop label-width="80px">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChange()"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      addCateDialogVisible: false,
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name',
          width: '400px'
        },
        {
          label: '是否有效',
          prop: 'cat_deleted',
          type: 'template',
          template: 'is_ok'
        },
        {
          label: '排序',
          prop: 'cat_level',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addCateForm: {
        cat_name: '',
        // 父级分类id
        cat_pid: 0,
        // 分类的等级，默认要添加的是1级分类
        cat_level: 0
      },
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      value: [],
      parentCateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: 'true'
      },
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data } = await this.$http.get('/categories', {
        params: this.queryInfo
      })
      if (data.meta.status !== 200) {
        return this.$message.error('获取商品列表数据失败')
      }
      // 将数据列表赋值给cateList
      this.cateList = data.data.result
      // // 保存总数据条数
      this.total = data.data.total
      // console.log(this.total)
    },
    async getParentCateList() {
      const { data } = await this.$http.get('/categories', {
        params: {
          type: 2
        }
      })
      if (data.meta.status !== 200) {
        return this.$message.error('获取商品父级分类失败')
      }
      // 将数据列表赋值给cateList
      this.parentCateList = data.data
      console.log(this.parentCateList)
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getCateList()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getCateList()
    },
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) this.$message.error('确保字段正确')
        const { data } = await this.$http.post('/categories', this.addCateForm)
        if (data.meta.status === 201) this.$message.error('添加商品分类失败')
        this.getCateList()
        this.addCateDialogVisible = false
        this.$message.success('添加商品分类成功')
      })
    },
    parentCateChange() {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_level = this.selectedKeys.length
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
      } else {
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      }
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-input {
  width: 90%;
}
.el-cascader {
  width: 90%;
}
</style>
