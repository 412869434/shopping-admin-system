<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图区域 -->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showCatDialogVisible">添加分类</el-button>
                </el-col>
            </el-row>

            <!-- 表格 -->
            <tree-table :data="cateList" :columns="columns" :selection-type="false"
            :expand-type="false" show-index index-text="#" border :show-row-hover="false">
                <template slot="isok" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color:lightgreen;"></i>
                    <i class="el-icon-error" v-else  style="color:red;"></i>
                </template>

                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>

                <template slot="opt" slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.cat_id)">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCate(scope.row.cat_id)"> 删除</el-button>
                </template>
            </tree-table>

            <!-- 分页 -->
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="querryInfo.pagenum"
            :page-sizes="[3, 5, 10,15]"
            :page-size="querryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>

        </el-card>

        <!-- 添加分类的对话框 -->
        <el-dialog title="添加分类" :visible.sync="addCatDialogVisible" width="50%" @close="addCateDialogClosed">
            <!-- form -->
            <el-form :model="addCatForm" :rules="addCatFormRules" ref="addCatFormRef" label-width="100px">   
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCatForm.cat_name"></el-input>
                </el-form-item>

                <el-form-item label="父级分类：">
                    <el-cascader expand-trigger="hover" :options="parentCateList"
                    :props="cascaderProps" v-model="selectedKeys"
                    @change="parentCateChanged" clearable change-on-select></el-cascader>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="addCatDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 编辑分类的对话框 -->
        <el-dialog title="修改分类" :visible.sync="editDialogVisible" width="30%" @close="editCateClosed">

            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="editForm.cat_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data (){
        return {
            //查询条件
            querryInfo: {
                type:3,
                //当前页数
                pagenum: 1,
                //当前页显示多少数据
                pagesize: 5
            },
            //商品分类的数据列表，默认为空
            cateList: [],
            total: 0,
            //table 的列
            columns: [
                {
                    label: '分类名称',
                    prop: 'cat_name'
                },
                {
                    label: '是否有效',
                    type: 'template',
                    //当前这一列使用的模板名称
                    template: 'isok'
                },
                {
                    label: '排序',
                    type: 'template',
                    //当前这一列使用的模板名称
                    template: 'order'
                },
                {
                    label: '操作',
                    type: 'template',
                    //当前这一列使用的模板名称
                    template: 'opt'
                }
            ],
            //控制添加分类的对话框
            addCatDialogVisible:false,
            addCatForm:{
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            addCatFormRules: {
                cat_name: [
                    { required: true, message: '请输入分类名称', trigger: 'blur' },
                ]
            },
            //父级分类列表
            parentCateList: [],
            cascaderProps:{
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedKeys: [],
            editDialogVisible: false,
            editForm: {},
            editFormRules: {
                cat_name : [
                    { required: true, message: '请输入分类名称', trigger: 'blur' },
                ]
            }
        }
    },
    created(){
        this.getCateList();
    },
    methods: {
        async getCateList() {
            const {data:res} = await this.$http.get('categories',{ params:this.querryInfo});
            if(res.meta.status !== 200) return this.$message.error('error');
            this.cateList = res.data.result;
            this.total = res.data.total;
            //console.log(this.cateList);
        },
        //监听pagesize改变事件
        handleSizeChange(newSize) {
            this.querryInfo.pagesize = newSize;
            this.getCateList();
        },
        //监听页码值改变的事件
        handleCurrentChange(newPage){
            this.querryInfo.pagenum = newPage;
            this.getCateList();
        },
        //点击按钮展示对话框
        showCatDialogVisible() {
            //获取父级数据列表
            this.getParentCateList();
            
            this.addCatDialogVisible = true;
        },
        async getParentCateList() {
            const {data : res } = await this.$http.get('categories',{params:{type:2}})
            if(res.meta.status !== 200) return this.$message.error('error');
            this.parentCateList = res.data;
            //console.log(this.parentCateList);
        },
        parentCateChanged() {
            //console.log(this.selectedKeys);
            //长度大于0，说明选中了父级id
            if(this.selectedKeys.length > 0){
                this.addCatForm.cat_pid = this.selectedKeys[this.selectedKeys.length -1]
                this.addCatForm.cat_level = this.selectedKeys.length
                return 
            }
            else{
                this.addCatForm.cat_pid = 0
                this.addCatForm.cat_level = 0
            }
        },
        addCate() {
            this.$refs.addCatFormRef.validate(async valid => {
                if(!valid) return
                const {data:res} = await this.$http.post('categories',this.addCatForm)
                if(res.meta.status !== 201) return this.$message.error('error');
                this.$message.success('success');
                this.getCateList();
                this.addCatDialogVisible = false;
            })
        },
        addCateDialogClosed() {
            this.$refs.addCatFormRef.resetFields();
            this.selectedKeys = [];
            this.addCatForm.cat_level = 0;
            this.addCatForm.cat_pid = 0;
        },
        async showEditDialog(id) {
            const {data:res} = await this.$http.get('categories/'+id)
            if(res.meta.status !== 200) return this.$message.error('error');
            this.editForm = res.data
            this.editDialogVisible = true;
        },
        editCate() {
            this.$refs.editFormRef.validate(async valid => {
                if(!valid) return ;
                const {data:res} = await this.$http.put('categories/'+this.editForm.cat_id,{cat_name:this.editForm.cat_name})
                if(res.meta.status !== 200) return this.$message.error('error');
                this.$message.success('success');
                this.getCateList();
                this.editDialogVisible = false;
            })
        },
        editCateClosed() {
            this.$refs.editFormRef.resetFields();
        },
        async removeCate(id) {
            //询问用户是否删除
            const confirmRes = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            //如果用户确认删除，则返回的字符串为confirm
            //如果用户取消删除，则返回值为字符串cancel
            if(confirmRes !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete('categories/'+id)

            if(res.meta.status !== 200) return this.$message.error('删除失败')
            this.$message.success('删除成功')
            this.getCateList();
        }
    }
}
</script>

<style lang="less" scoped>
.el-cascader {
    width: 100%;
}
</style>
