<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/img/header.png" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 页面侧边栏 -->
      <el-aside width="isCollapse ? '64px' : '200px' ">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          background-color="#505761"
          text-color="#fff"
          active-text-color="#409eff"
          router
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + subItem.path + ''"
              v-for="subItem in item.children"
              :key="subItem.id"
            >
              <!-- 图标 -->
              <i class="el-icon-menu"></i>
              <!-- 文本 -->
              <span>{{ subItem.authName }}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!--右侧内容主体区域 -->
      <el-main>
        <!-- 路由占位符 -->
        <router-view> </router-view>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {
  data() {
    //左侧菜单数据
    return {
      menulist: [],
      iconsObj: {
        '125': 'iconfont icon-renyuan',
        '103': 'iconfont icon-shezhi ',
        '101': 'iconfont icon-shangcheng',
        '102': 'iconfont  icon-xianzhi',
        '145': 'iconfont icon-shuju'
      },
      //是否折叠侧边栏
      isCollapse: false
    }

  },
  created() {
    this.getMenuList()
  },
  methods: {
    logout() {
      //清空token
      window.sessionStorage.clear()
      //重新路由到登录页
      this.$router.push('/login')
    },
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')

      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    },
    //点击按钮切换菜单的折叠与展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    }
  }

}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #505761;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 2%;
  color: #fff;
  line-height: 60px;
  > div {
    display: flex;
    align-items: center;
    margin-left: 15px;
    > img {
      width: 40px;
      height: 40px;
      border-radius: 40%;
    }
    > span {
      padding-left: 15px;
    }
  }
}

.el-aside {
  background-color: #505761;
  color: #333;
  text-align: left;
  line-height: 200px;
  .el-menu {
    border-right: none;
  }
}

.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
  line-height: 16spx;
}

body > .el-container {
  margin-bottom: 40px;
}
.el-container:nth-child(5) .el-aside,
.el-container:nth-child(6) .el-aside {
  line-height: 260px;
}

.el-container:nth-child(7) .el-aside {
  line-height: 320px;
}

.iconfont {
  margin-right: 10px;
  font-size: 25px;
}

.toggle-button {
  background-color: #858b94;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>