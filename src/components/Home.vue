<template>
<el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
        <div class="left">
            <img src="../assets/img/heima.png" alt="">
            <span>电商后台管理系统</span>
        </div>
        <div class="right">
            <el-button type="info" @click="logout"> 退出 </el-button>
        </div>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
        <!-- 侧边栏 -->
        <el-aside :width="isCollapse?'64px':'200px'">
            <div class="toggle" @click="toggleCollapse">|||</div>
            <el-menu background-color="#333744" text-color="#fff" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
                <el-submenu :index="item.id+''" v-for="item in MenuList" :key="item.id">
                    <template slot="title">
                        <i :class="iconsObj[item.id]"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <el-menu-item :index="'/'+ subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="toggleActivePath('/'+ subItem.path)">
                        <template slot="title">
                            <i class="el-icon-location"></i>
                            <span>{{subItem.authName}}</span>
                        </template>
                    </el-menu-item>
                </el-submenu>
            </el-menu>
        </el-aside>
        <!-- 主体结构 -->
        <el-main>
            <router-view></router-view>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
export default {
  created () {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  data () {
    return {
      activePath: '',
      isCollapse: false,
      MenuList: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      }
    }
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('login')
    },
    async getMenuList () {
      const {
        data: res
      } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // console.log(res)
      this.MenuList = res.data
    },
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    toggleActivePath (path) {
      this.activePath = path
      window.sessionStorage.setItem('activePath', path)
    }
  }
}
</script>

<style lang="less" scoped>
.el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;

    .left {
        img {
            vertical-align: middle;
            margin-right: 20px;
        }

        span {
            vertical-align: middle;
            color: #fff;
            font-size: 20px;
        }
    }

    .right {
        display: flex;
        align-items: center;
    }
}

.el-aside {
    background-color: #333744;

}

.el-main {
    background-color: #eaedf1;
}

.home-container {
    height: 100%;
}

.toggle {
    letter-spacing: 5px;
    color: #fff;
    background-color: #4a5064;
    text-align: center;
    cursor: pointer;
}
</style>
