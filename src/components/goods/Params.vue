<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图区域 -->
        <el-card>
            <el-alert show-icon title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false"></el-alert>
            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <!-- 选择商品分类的级联选择框 -->
                    <el-cascader expand-trigger="hover" :options="cateList"
                    :props="cascaderProps" v-model="selectedKeys"
                    @change="parentCateChanged" clearable></el-cascader>
                </el-col>
            </el-row>

            <!-- Tabs -->
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <!-- 动态参数 -->
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTableData" border stripe>
                        <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">
                                    {{item}}
                                </el-tag>

                                <el-input class="input-new-tag" v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue" ref="saveTagInput" size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <!-- 静态属性 -->
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible = true">添加属性</el-button>
                    <!-- 静态属性表格 -->
                    <el-table :data="onlyTableData" border stripe>
                        <!-- 展开行 -->
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">
                                    {{item}}
                                </el-tag>

                                <el-input class="input-new-tag" v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue" ref="saveTagInput" size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)">
                                </el-input>
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <!-- 索引列 -->
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        
        </el-card>

        <!-- 添加参数对话框 -->
        <el-dialog :title="'添加' + titleText" :visible.sync="addDialogVisible" width="50%" @close="addCateClosed">

            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addAttr">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改参数对话框 -->
        <el-dialog :title="'修改' + titleText" :visible.sync="editDialogVisible" width="50%" @close="editCateClosed">

            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editAttr">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            cateList: [],
            cascaderProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedKeys: [],
            activeName: 'many',
            manyTableData: [],
            onlyTableData: [],
            //控制添加对话框
            addDialogVisible:false,
            addForm: {
                attr_name: ''
            },
            addFormRules: {
                attr_name: [
                    {required: true, message: '请输入参数名称', trigger: 'blur'}
                ]
            },
            //控制修改对话框
            editDialogVisible:false,
            editForm: {
                attr_name: ''
            },
            editFormRules: {
                attr_name: [
                    {required: true, message: '请输入参数名称', trigger: 'blur'}
                ]
            },
        }
    },
    created() {
        this.getCateList();
    },
    methods: {
        async getCateList() {
            const {data:res} = await this.$http.get('categories');
            if(res.meta.status !== 200) return this.$message.error('error');
            this.cateList = res.data;
        },
        parentCateChanged() {
            
            this.getParamsData();
        },
        handleClick() {
            this.getParamsData();
        },
        async getParamsData() {
            if(this.selectedKeys.length !== 3){
                this.selectedKeys = [];
                this.manyTableData = [];
                this.onlyTableData = [];
                return 
            }
            // 根据所选id和当前面板获取对应的参数
            const {data:res} = await this.$http.get(`categories/${this.casteId}/attributes`,{params:{sel:this.activeName}})
            if(res.meta.status !== 200) return this.$message.error('error');
        
            //console.log(res.data);
            res.data.forEach(item => {
                item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                //控制文本框的显示与隐藏
                item.inputVisible = false
                item.inputValue = ''
            })
            //console.log(res.data);
            if(this.activeName === 'many'){
                this.manyTableData = res.data;
            }
            else{
                this.onlyTableData = res.data;
            }
        },
        //监听对话框的关闭事件
        addCateClosed() {
            this.$refs.addFormRef.resetFields();
        },
        // 点击按钮，添加参数
        addAttr() {
            this.$refs.addFormRef.validate(async valid => {
                if(!valid) return;
                const {data : res} = await this.$http.post(`categories/${this.casteId}/attributes`,{
                    attr_name:this.addForm.attr_name,attr_sel: this.activeName
                })
                if(res.meta.status !== 201) return this.$message.error('error');

                this.$message.success('success')
                this.addDialogVisible = false
                this.getParamsData();
            })
        },
        //点击按钮，
        async showEditDialog(id) {
            const {data : res} = await this.$http.get(`categories/${this.casteId}/attributes/${id}`,{
                params:{attr_sel: this.activeName}
            })
            if(res.meta.status !== 200) return this.$message.error('error');
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        //监听对话框的关闭事件
        editCateClosed() {
            this.$refs.editFormRef.resetFields();
        },
        // 点击按钮，添加参数
        editAttr() {
            this.$refs.editFormRef.validate(async valid => {
                if(!valid) return;
                const {data : res} = await this.$http.put(`categories/${this.casteId}/attributes/${this.editForm.attr_id}`,{
                    attr_name:this.editForm.attr_name,attr_sel: this.activeName
                })
                if(res.meta.status !== 200) return this.$message.error('error');

                this.$message.success('success')
                this.editDialogVisible = false
                this.getParamsData();
            })
        },
        //删除对应的参数
        async removeById(id) {
            //询问用户是否删除
            const confirmRes = await this.$confirm('此操作将永久删除该参数或属性, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            //如果用户确认删除，则返回的字符串为confirm
            //如果用户取消删除，则返回值为字符串cancel
            if(confirmRes !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete(`categories/${this.casteId}/attributes/${id}`)

            if(res.meta.status !== 200) return this.$message.error('删除失败')
            this.$message.success('删除成功')
            this.getParamsData();
        },
        //事件处理函数
        handleInputConfirm(row) {
            //console.log('ok');
            if(row.inputValue.trim().length === 0){
                row.inputValue = ''
                row.inputVisible = false
                return
            }
            //输入了内容
            row.attr_vals.push(row.inputValue.trim())
            //console.log(row.attr_vals);
            row.inputValue = ''
            row.inputVisible = false
            this.saveAttrVals(row);
        },
        //点击按钮，展示文本输入框
        showInput(row) {
            row.inputVisible = true
            //让文本框自动获得焦点
            //$nextTick方法作用，就是当页面上元素被重新渲染之后，才会执行回调函数
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus()
            })
        },
        async saveAttrVals(row) {
            const {data :res} = await this.$http.put(`categories/${this.casteId}/attributes/${row.attr_id}`,{
                attr_name:row.attr_name,attr_sel:row.attr_sel,attr_vals:row.attr_vals.join(' ')
            })
            if(res.meta.status !== 200) return this.$message.error('保存失败')
            this.$message.success('保存成功')
        },
        handleClose(i,row) {
            //console.log(row.attr_vals);
            row.attr_vals.splice(i,1)
            //console.log(row.attr_vals);
            this.saveAttrVals(row);
        },
    },
    computed: {
        isBtnDisabled() {
            return this.selectedKeys.length !== 3;
        },
        casteId() {
            return this.selectedKeys.length === 3 ? this.selectedKeys[2] : null;
        },
        //动态计算标题属性
        titleText() {
            return this.activeName === 'many' ? '动态参数' : '静态属性' ;
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}

.el-tag {
    margin: 10px;
}

.input-new-tag {
    width: 120px;
}
</style>