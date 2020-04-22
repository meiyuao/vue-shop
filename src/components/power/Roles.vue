<template>
  <div>
    <!-- 面包屑导航 -->

    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>

      <el-table :data="rolesList" stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bd_bottom','vcenter']"
              v-for="item1 in scope.row.children"
              :key="item1.id"
            >
              <!--一级菜单栏-->
              <el-col :span="5">
                <el-tag
                  closable
                  :disable-transitions="false"
                  @close="removeRightById(scope.row,item1.id)"
                >{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!--二、三级菜单栏-->
              <el-col :span="19">
                <el-row
                  :class="[i===0?'':'bd_top','vcenter']"
                  v-for="(item2,i) in item1.children"
                  :key="item2.id"
                >
                  <!--二级菜单栏-->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      :disable-transitions="false"
                      @close="removeRightById(scope.row,item2.id)"
                    >{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!--三级菜单栏-->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      :disable-transitions="false"
                      @close="removeRightById(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column fixed="right" label="操作" width="300">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row)"
            >编辑</el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRoleById(scope.row.id)"
            >删除</el-button>
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" @close="addDialogClosed">
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="角色信息修改" :visible.sync="editDialogVisible" @close="editDialogClosed">
      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <!-- 树形组件
    show-checkbox:显示复选框
    node-key:设置选中节点对应的值
    default-expand-all:是否默认展开所有节点
    :default-checked-keys 设置默认选中项的数组
      ref:设置引用-->
      <el-tree
        :data="rightsList"
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
      rolesList: [],
      rightsList: [],
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      editForm: {
        id: '',
        roleName: '',
        roleDesc: ''
      },
      addDialogVisible: false,
      editDialogVisible: false,
      setRightDialogVisible: false,

      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'change' }
        ]
      },
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'change' }
        ]
      },
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
    this.getRightsList()
  },
  methods: {
    async getRolesList() {
      const { data } = await this.$http.get('/roles')
      if (data.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      // 如果返回状态正常，将请求的数据保存在data中
      this.rolesList = data.data
      console.log(this.rolesList)
    },
    async getRightsList() {
      const { data } = await this.$http.get('rights/tree')
      if (data.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      // 如果返回状态正常，将请求的数据保存在data中
      this.rightsList = data.data
      console.log(this.rolesList)
    },
    addDialogClosed() {
      // 对话框关闭之后，重置表单
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      // 对话框关闭之后，重置表单
      this.$refs.editFormRef.resetFields()
    },
    setRightDialogClosed() {
      // 当用户关闭树形权限对话框的时候，清除掉所有选中状态
      this.defKeys = []
    },
    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写完整角色信息')
        const { data } = await this.$http.post('/roles', this.addForm)
        // console.log(data)
        if (data.meta.status !== 201) return this.$message.error('添加角色失败')
        // 关闭对话框
        this.addDialogVisible = false
        // 重新请求最新的数据
        this.getRolesList()
        // 添加成功的提示
        this.$message.success('添加角色成功')
      })
    },
    removeRoleById(id) {
      this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data } = await this.$http.delete('/roles/' + id)
          if (data.meta.status !== 200) {
            return this.$message({ type: 'danger', message: '删除失败!' })
          }
          this.getRolesList()
          this.$message({ type: 'success', message: '删除成功!' })
        })
        .catch(() => this.$message({ type: 'info', message: '已取消删除' }))
    },
    showEditDialog(row) {
      this.editDialogVisible = true
      this.editForm.id = row.id
      this.editForm.roleName = row.roleName
      this.editForm.roleDesc = row.roleDesc
    },
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$message.error('格式不正确')

        const { data } = await this.$http.put('/roles/' + this.editForm.id, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        if (data.meta.status !== 200) return this.$message.error('更新用户失败')

        // 关闭对话框
        this.editDialogVisible = false
        // 重新请求最新的数据
        this.getRolesList()
        // 添加成功的提示
        this.$message.success('更新角色成功')
      })
    },
    async removeRightById(role, rightId) {
      // 弹窗提示用户是否要删除
      const confirmResult = await this.$confirm(
        '请问是否要删除该权限',
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

      // 用户点击了确定表示真的要删除
      // 当发送delete请求之后，返回的数据就是最新的角色权限信息
      const { data } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )
      if (data.meta.status !== 200) {
        return this.$message.error('删除角色权限失败')
      }

      // 无需再重新加载所有权限
      // 只需要对现有的角色权限进行更新即可
      role.children = data.data
      // this.getRoleList();
    },
    showSetRightDialog(role) {
      this.roleId = role.id
      // 调用getLeafKeys进行递归，将三级权限添加到数组中
      this.getLeafKeys(role, this.defKeys)
      //   this.$nextTick官方解释：将回调延迟到下次 DOM 更新循环之后执行。在修改数据之后立即使用它，然后等待 DOM
      this.$nextTick(() => {
        this.$refs.treeRef.setCheckedKeys(this.defKeys)
      })
      this.setRightDialogVisible = true
    },
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      // 递归调用
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    async allotRights(role) {
      // 获取所有选中及半选的内容
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // 将数组转换为 , 拼接的字符串
      const idStr = keys.join(',')

      // 发送请求完成更新
      const { data } = await this.$http.post('roles/' + this.roleId + '/rights', { rids: idStr })
      console.log(data)
      if (data.meta.status !== 200) { return this.$message.error('分配权限失败') }
      this.$message.success('分配权限成功')
      this.getRolesList()

      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bd_top {
  border-top: 1px solid #eee;
}
.bd_bottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
