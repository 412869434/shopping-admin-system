<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图区域 -->
        <el-card>
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
                        <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="goAddPage">添加商品</el-button>
                </el-col>
            </el-row>

            <!-- table -->
            <el-table :data="goodsList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column>
                    <el-table-column label="商品名称" prop="goods_name"></el-table-column>
                    <el-table-column label="商品价格" prop="goods_price" width="95px"></el-table-column>
                    <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
                    <el-table-column label="创建时间" prop="add_time" width="140px">
                        <template slot-scope="scope">
                            {{scope.row.add_time | dateFormat}}
                        </template>    
                    </el-table-column>
                    <el-table-column label="操作" width="130px">
                        <template slot-scope="scope">
                            <el-button type="primary" size="mini" icon="el-icon-edit"></el-button>
                            <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeById(scope.row.goods_id)"></el-button>
                        </template>
                    </el-table-column>
                </el-table-column>
            </el-table>

            <!-- 分页 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15, 20]" :total="total"
            :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper"
            background>
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        return {
            //查询参数对象
            queryInfo: {
                query: '',
                pagenum: 1,
                pagesize:10
            },
            //商品列表
            goodsList: [],
            total: 0
        }
    },
    created() {
        this.getGoodsList();
    },
    methods: {
        //分页查询
        async getGoodsList() {
            const {data:res} = await this.$http.get(`goods`,{params:this.queryInfo});
            if(res.meta.status !== 200) return this.$message.error('error');
            this.goodsList = res.data.goods;
            this.total = res.data.total;
        },
        //监听pagesize改变事件
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize;
            this.getGoodsList()
        },
        //监听页码值改变的事件
        handleCurrentChange(newPage){
            this.queryInfo.pagenum = newPage;
            this.getGoodsList()
        },
        async removeById(id) {
            //询问用户是否删除
            const confirmRes = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            //如果用户确认删除，则返回的字符串为confirm
            //如果用户取消删除，则返回值为字符串cancel
            if(confirmRes !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete('goods/'+id)

            if(res.meta.status !== 200) return this.$message.error('删除失败')
            this.$message.success('删除成功')
            this.getGoodsList()
        },
        goAddPage() {
            this.$router.push('/goods/add')
        }
    }
}
</script>

<style lang="less" scoped>

</style>