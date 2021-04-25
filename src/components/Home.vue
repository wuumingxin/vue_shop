<template>
  <el-container class="home-container">
    <!-- 头部 -->
    <el-header>
      <div style="height: 100%">
        <img src="../assets/logo.png" alt="" /><span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 主体 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409bff"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ''"
            v-for="(item, index) in menulist"
            :key="item.id"
          >
            <!-- 一级菜单模版区 -->
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[index]" class="iconFont"></i>
              <!-- 文本 -->
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
              @click="savaNavState('/' + subItem.path)"
            >
              <!-- 二级菜单模版区 -->
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{ subItem.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容区 -->
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
      menulist: [],
      iconsObj: {
        0: 'el-icon-attract',
        1: 'el-icon-mobile',
        2: 'el-icon-scissors',
        3: 'el-icon-umbrella',
        4: 'el-icon-headset'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的连接地址
      activePath: ''
    }
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    },
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    savaNavState (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  },
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  }
}
</script>

<style>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 0;
  color: #fff;
  font-size: 20px;
}
.el-header div {
  display: flex;
  align-items: center;
}
.el-header img {
  height: 80%;
  width: auto;
}
.el-header span {
  padding-left: 15px;
}
.el-aside {
  background-color: #333744;
}
.el-main {
  background-color: #eaedf1;
}
.iconFont {
  margin-right: 10px;
}
.el-aside .el-menu {
  border-right: none;
}
/* 不允许文字被选中 */
.el-aside span {
  user-select: none;
}
.toggle-button {
  background-color: #4a5064;
  text-align: center;
  line-height: 24px;
  color: #fff;
  font-size: 10px;
  letter-spacing: 0.2em;
  cursor: pointer;
  user-select: none;
}
</style>
