<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图 -->
        <el-card>
            <!-- add user button -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>

            <!-- 角色列表区域 -->
            <el-table :data="roleList" border stripe>
                <!-- 展开列 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['vcenter','bdbottom',i1 === 0 ? 'bdtop' : '']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                            <!-- 1 -->
                            <el-col :span="5">
                                <el-tag  closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 2 & 3 -->
                            <el-col :span="19">
                                <!-- 通过for循环，嵌套渲染二级权限 -->
                                <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="['vcenter',i2 !== 0 ? 'bdtop' : '']">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag v-for="(item3,i3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row,item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index" label="#"></el-table-column>
                <!-- 角色信息 -->
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)" >分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <!-- add user -->
            <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
                
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" lable-width="70px">
                    <el-form-item label="角色名称" prop="roleName">
                        <el-input v-model="addForm.roleName"></el-input>
                    </el-form-item>
                
                    <el-form-item label="角色描述" prop="roleDesc">
                        <el-input v-model="addForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>

                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addUser()">确 定</el-button>
                </span>
            </el-dialog>

            <!-- show edit dialog -->
            <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @closed="editDialogClosed">
                <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" lable-width="70px">
                    <el-form-item label="角色名称" prop="roleName">
                        <el-input v-model="editForm.roleName"></el-input>
                    </el-form-item>
                
                    <el-form-item label="角色描述" prop="roleDesc">
                        <el-input v-model="editForm.roleDesc"></el-input>
                    </el-form-item>

                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editUserInfo">确 定</el-button>
                </span>
            </el-dialog>

            <!-- show set right dialog -->
            <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @closed="setRightDialogClosed">
                <!-- tree -->
                <el-tree :data="rightList" :props="treeProps"
                show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys"
                ref="treeRef"></el-tree>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRightDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="allotRights">确 定</el-button>
                </span>
            </el-dialog>

        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        return {
            //角色列表
            roleList: [],
            addDialogVisible:false,
            //添加用户的表单数据
            addForm: {},
            //添加表单的验证规则对象
            addFormRules: {
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' },
                ],
                roleDesc: [
                    { required: true, message: '请输入角色描述', trigger: 'blur' },
                ],
            },
            editDialogVisible:false,
            //查询到的用户对象
            editForm: {},
            editFormRules: {
                roleName: [
                    { required: true, message: '请输入角色名称', trigger: 'blur' },
                ],
                roleDesc: [
                    { required: false, message: '请输入角色描述', trigger: 'blur' },
                ],
            },
            setRightDialogVisible:false,
            // 所有权限的数据
            rightList: [],
            //树形控件的属性绑定对象
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            // 默认选中的id
            defKeys: [],
            roleId: '',


        }
    },
    created() {
        this.getRoleList();
    },
    methods: {
        async getRoleList() {
            const {data: res} = await this.$http.get('roles');
            if(res.meta.status !== 200) return this.$message.error('error');
            this.roleList = res.data;
            //console.log(this.roleList);
        },
        //监听对话框关闭
        addDialogClosed() {
            this.$refs.addFormRef.resetFields();
        },
        addUser() {
            //console.log(this.$refs.addFormRef);
            this.$refs.addFormRef.validate(async valid => {
                if(!valid) return ;
                const {data:res} = await this.$http.post('roles',this.addForm);
                if(res.meta.status !== 201) {
                    return this.$message.error('add role error');
                }
                this.$message.success('add role success');
                this.addDialogVisible = false;
                this.getRoleList();
            })
        },
        //展示给用户的对话框
        async showEditDialog(id) {
            const {data:res} = await this.$http.get('roles/'+id);
            if(res.meta.status !== 200) {
                return this.$message.error('find role error');
            }
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        editDialogClosed() {
            this.$refs.editFormRef.resetFields();
        },
        editUserInfo() {
            this.$refs.editFormRef.validate(async valid => {
                if(!valid) return ;
                const {data:res} = await this.$http.put('roles/'+this.editForm.roleId,{
                    roleName:this.editForm.roleName,roleDesc:this.editForm.roleDesc
                });
                if(res.meta.status !== 200) {
                    return this.$message.error('edit role error');
                }
                this.$message.success('edit role success');
                this.editDialogVisible = false;
                this.getRoleList();
            })
        },
        async removeUserById(id) {
            //询问用户是否删除
            const confirmRes = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            //如果用户确认删除，则返回的字符串为confirm
            //如果用户取消删除，则返回值为字符串cancel
            if(confirmRes !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete('roles/'+id)

            if(res.meta.status !== 200) return this.$message.error('删除失败')
            this.$message.success('删除成功')
            this.getRoleList();
        },
        async removeRightById(role,rightId) {
            //询问用户是否删除
            const confirmRes = await this.$confirm('此操作将永久删除该角色的权限, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            //如果用户确认删除，则返回的字符串为confirm
            //如果用户取消删除，则返回值为字符串cancel
            if(confirmRes !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)

            if(res.meta.status !== 200) return this.$message.error('删除失败')
            
            role.children = res.data;
            this.$message.success('删除成功')
            // this.getRoleList();
        },
        async showSetRightDialog(role) {
            this.roleId = role.id;
            const {data:res} = await this.$http.get('rights/tree');
            if(res.meta.status !== 200) return this.$message.error('error');
            this.rightList = res.data;

            this.getLeafKey(role,this.defKeys)
            this.setRightDialogVisible = true;
        },
        // 通过递归获取三级权限id
        getLeafKey(node,arr) {
            //如果当前node节点不包含children属性，则是三级节点
            if(!node.children) return arr.push(node.id)
            node.children.forEach(item => {
                this.getLeafKey(item,arr)
            });
        },
        setRightDialogClosed() {
            this.defKeys = [];
        },
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ];
            
            const idStr = keys.join(',');

            const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr});
            if(res.meta.status !== 200) return this.$message.error('error');

            this.$message.success('success');
            this.getRoleList();

            this.setRightDialogVisible = false;
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.bdtop {
    border-top:1px solid #eee ;
}

.bdbottom {
    border-bottom: 1px solid #eee;
}

.vcenter {
    display: flex;
    align-items: center;
}
</style>