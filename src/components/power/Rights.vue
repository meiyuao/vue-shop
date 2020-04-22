<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-table :data="rightsList" stripe>
      <el-table-column type="index" label="#"></el-table-column>
      <el-table-column label="权限名称" prop="authName"></el-table-column>
      <el-table-column label="路径" prop="path"></el-table-column>
      <el-table-column label="权限等级" prop="level">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.level === '0'">一级权限</el-tag>
          <el-tag v-if="scope.row.level === '1'" type="success">二级权限</el-tag>
          <el-tag v-if="scope.row.level === '2'" type="warning">三级权限</el-tag>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rightsList: []
    }
  },
  created() {
    this.getRightsList()
  },
  methods: {
    async getRightsList() {
      const { data } = await this.$http.get('rights/list')
      if (data.meta.status !== 200) return this.$message.error('获取权限列表失败')
      // 如果返回状态正常，将请求的数据保存在data中
      this.rightsList = data.data
      console.log(this.rightsList)
    }
  }
}
</script>

<style lang="less" scoped>
</style>
