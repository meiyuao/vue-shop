<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <!-- 警告区域 :closable="false"(是否展示“X”号) show-icon(显示图标) -->
      <el-alert title="注意：只允许为第三级分类设置相关参数" type="warning" show-icon></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cascaderProps"
            @change="handleCateChange()"
          ></el-cascader>
        </el-col>
        <el-col></el-col>
      </el-row>
      <!-- tab页签区域 -->
      <el-tabs v-model="activeName">
        <!-- 添加动态参数的面板 将标签页改为many -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            :disabled="isButtonDisabled"
            @click="addDialogVisible = true"
          >添加参数</el-button>
          <el-table :data="dynamicParams" style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="tag in scope.row.attr_vals?scope.row.attr_vals.split(','):[]"
                  :key="tag"
                  closable
                >{{tag}}</el-tag>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" @click="showEditDialog(scope.row)">修改</el-button>
                <el-button size="mini" type="danger" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 将标签页改为only -->
        <el-tab-pane label="静态参数" name="only">
          <el-button
            type="primary"
            :disabled="isButtonDisabled"
            @click="addDialogVisible = true"
          >添加参数</el-button>
          <el-table :data="staticParams" style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="tag in scope.row.attr_vals?scope.row.attr_vals.split(','):[]"
                  :key="tag"
                  closable
                >{{tag}}</el-tag>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" @click="showEditDialog(scope.row)">修改</el-button>
                <el-button size="mini" type="danger" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数或属性对话框 -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 添加表单 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数或属性对话框 -->
    <el-dialog
      :title="'修改'+titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 添加表单 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 分类列表
      cateList: [],
      // 用户在级联下拉菜单中选中的分类id
      selectedCateKeys: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeName: 'many',
      dynamicParams: [],
      staticParams: [],
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        attr_name: '',
        attr_sel: 'many',
        attr_vals: ''
      },
      editForm: {
        attr_name: '',
        attrId: ''
      },
      // 添加表单验证规则
      addFormRules: {
        attr_name: [{ required: true, message: '请输入名称', trigger: 'blur' }]
      },
      // 修改表单的验证规则
      editFormRules: {
        attr_name: [{ required: true, message: '请输入名称', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getCateList()
  },
  computed: {
    isButtonDisabled() {
      return this.selectedCateKeys.length !== 3
    },
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    titleText() {
      if (this.activeName === 'many') return '动态参数'
      if (this.activeName === 'only') return '静态参数'
      return ''
    }
  },
  methods: {
    async getCateList() {
      const { data } = await this.$http.get('/categories', {
        params: {
          type: 3
        }
      })
      if (data.meta.status !== 200) {
        return this.$message.error('获取商品列表数据失败')
      }
      // 将数据列表赋值给cateList
      this.cateList = data.data
      // // 保存总数据条数
      this.total = data.data.total
      // console.log(this.total)
    },
    handleCateChange() {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        return
      }
      this.getDynamicParams()
      this.getStaticParams()
    },
    async getDynamicParams() {
      const { data } = await this.$http.get(
        'categories/' + this.cateId + '/attributes',
        { params: { sel: 'many' } }
      )

      if (data.meta.status !== 200) this.$message.error('获取动态参数失败')

      this.dynamicParams = data.data
      console.log(data)
    },
    async getStaticParams() {
      const { data } = await this.$http.get(
        'categories/' + this.cateId + '/attributes',
        { params: { sel: 'only' } }
      )

      if (data.meta.status !== 200) this.$message.error('获取动态参数失败')

      this.staticParams = data.data
      console.log(data)
    },
    addDialogClosed() {
      this.addDialogVisible = false
    },
    addParams() {
      // 当用户点击对话框中的确定时，校验表单
      this.$refs.addFormRef.validate(async valid => {
        // 校验不通过，return
        if (!valid) return
        // 校验通过，发送请求完成添加参数或者属性
        const { data } = await this.$http.post(
          'categories/' + this.cateId + '/attributes',
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        console.log(data)
        if (data.meta.status !== 201) {
          return this.$message.error('添加' + this.titleText + '数据失败')
        }
        this.$message.success('添加' + this.titleText + '数据成功')
        this.addDialogVisible = false
        if (this.activeName === 'many') {
          this.getDynamicParams()
        }
        if (this.activeName === 'only') {
          this.getStaticParams()
        }
      })
    },
    showEditDialog(role) {
      this.editDialogVisible = true
      this.editForm.attr_name = role.attr_name
      this.editForm.attrId = role.attr_id
      console.log(role)
    },
    editParams() {
      // 当用户点击对话框中的确定时，校验表单
      this.$refs.editFormRef.validate(async valid => {
        // 校验不通过，return
        if (!valid) return
        // 校验通过，发送请求完成添加参数或者属性
        const { data } = await this.$http.put(
          'categories/' + this.cateId + '/attributes/' + this.editForm.attrId,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        )
        console.log(data)
        if (data.meta.status !== 200) {
          return this.$message.error('修改' + this.titleText + '数据失败')
        }
        this.$message.success('修改' + this.titleText + '数据成功')
        this.editDialogVisible = false
        if (this.activeName === 'many') {
          this.getDynamicParams()
        }
        if (this.activeName === 'only') {
          this.getStaticParams()
        }
      })
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async removeParams(attrId) {
      // 根据id删除对应的参数或属性
      // 弹窗提示用户是否要删除
      const confirmResult = await this.$confirm(
        '请问是否要删除该' + this.titleText,
        '删除提示',
        {
          confirmButtonText: '确认删除',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 如果用户点击确认，则confirmResult 为'confirm'
      // 如果用户点击取消, 则confirmResult获取的就是catch的错误消息'cancel'
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }

      // 没有取消就是要删除，发送请求完成删除
      const { data } = await this.$http.delete(
        'categories/' + this.cateId + '/attributes/' + attrId
      )
      console.log(data)
      if (data.meta.status !== 200) {
        return this.$message.error('删除参数数据失败')
      }
      if (this.activeName === 'many') {
        this.getDynamicParams()
      }
      if (this.activeName === 'only') {
        this.getStaticParams()
      }
      this.$message.success('删除' + this.titleText + '数据成功')
    }
  }
}
</script>

<style lang="less" scoped>
.el-alert {
  margin-bottom: 15px;
}
.el-tag {
  margin: 5px;
}
</style>
