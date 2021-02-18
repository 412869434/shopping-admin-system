<template>

    <el-container class="home-container">
        <el-header>
            <div>
                <img src="../assets/logo.png" alt="">
                <span>电商管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
            <el-aside :width="isCollapse ? '64px' :'200px'">
                <div class="toggle-button" @click="toggleCollapse">|||</div>
                <el-menu background-color="#333744" text-color="#fff" active-text-color="#409eff"
                unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
                    <!-- 一级菜单 -->
                    <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
                        <!-- 一级模板 -->
                        <template slot="title">
                            <i :class="iconsobj[item.id]"></i>
                            <span>{{item.authName}}</span>
                        </template>
                        
                        <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id"
                        @click="saveNavState('/'+subItem.path)">
                            <!-- 二级模板 -->
                            <template slot="title">
                                <i class="el-icon-menu"></i>
                                <span>{{subItem.authName}}</span>
                            </template>
                        </el-menu-item>

                    </el-submenu>
                </el-menu>
            </el-aside>
            <el-main>
                <!-- 路由占位符 -->
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
            iconsobj: {
                '125': 'iconfont icon-user',
                '103': 'iconfont icon-tijikongjian',
                '101': 'iconfont icon-shangpin',
                '102': 'iconfont icon-danju',
                '145': 'iconfont icon-baobiao',
            },
            //是否折叠
            isCollapse:false,
            //被激活的地址
            activePath: '',
        }
    },
    created() {
        this.getMenuItem();
        this.activePath = window.sessionStorage.getItem('activePath');
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push('/login');
        },
        //获取所有菜单
        async getMenuItem() {
            const {data:res} = await this.$http.get('menus');
            if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
            //console.log(res);
            this.menulist = res.data;
        },
        //点击按钮切换
        toggleCollapse() {
            this.isCollapse = !this.isCollapse;
        },
        saveNavState(path) {
            window.sessionStorage.setItem('activePath',path);
            this.activePath = path;
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
    align-items: center;
    color: #fff;
    font-size: 20px;
    > div {
        display: flex;
        align-items: center;
        span {
            margin-left: 15px;
        }
    }
    img {
        width: 55px;
        height: 55px;
    }
}

.el-aside {
    background-color: #333744;
    .el-menu {
        border-right: none;
    }
}

.el-main {
    background-color: #eaedf1;
}

.iconfont {
    margin-right: 10px;
}

.toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
}

</style>