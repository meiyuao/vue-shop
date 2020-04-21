<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <!-- 黑马logo -->
        <img src="../assets/heima.png" alt />
        <!-- 顶部标题 -->
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <!-- 侧边区域 -->
      <el-aside :width="isCollapse? '64px':'200px'">
        <div class="toggle-button" @click="collapse">|||</div>
        <el-menu background-color="#333744" text-color="#fff" :collapse='isCollapse'
        :collapse-transition='false' :router="true" :default-active="activeNavPath">
          <!-- 一级菜单栏 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" v-bind:key='item.id'>
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children"
            :key="subItem.id" @click="saveNavPath('/'+subItem.path)">
              <!-- 二级菜单模板 -->
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧主体区域 -->
      <el-main>
          <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activeNavPath: ''
    }
  },
  created: function() {
    this.getMenuList()
  },
  methods: {
    logout() {
      window.sessionStorage.clear('token')
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('/menus')
        .then(res => {
          return res
        })
        .catch(err => {
          console.error(err)
        })
      if (res.meta.status !== 200) return console.log(res.meta.msg)
      this.menuList = res.data
    },
    collapse () {
      this.isCollapse = !this.isCollapse
    },
    saveNavPath(path) {
      window.sessionStorage.setItem('activeNavPath', path)
      this.activeNavPath = path
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: white;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    > span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;
}
.el-main {
  background-color: #eaedf1;
}
.iconfont{
  margin-right: 10px ;
}
.toggle-button{
  background-color: #4a5064;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  line-height: 30px;
  cursor: pointer;
}
</style>
