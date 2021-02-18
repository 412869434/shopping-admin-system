<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片 -->
        <el-card>
            <!-- 搜索 -->
            <el-row :gutter="20">
                <el-col :span="7">
                    <el-input placeholder="请输入内容" v-model="querryInfo.query" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <!-- 用户列表 -->
            <el-table :data="userList" border stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="username" label="姓名"></el-table-column>
                <el-table-column prop="email" label="邮箱"></el-table-column>
                <el-table-column prop="mobile" label="电话"></el-table-column>
                <el-table-column prop="role_name" label="角色"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false" >
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <!-- 分页区域 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="querryInfo.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="querryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>

            <!-- add user -->
            <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
                
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" lable-width="70px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="addForm.username"></el-input>
                    </el-form-item>
                
                    <el-form-item label="密码" prop="pawwsord">
                        <el-input v-model="addForm.password"></el-input>
                    </el-form-item>
                
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="addForm.email"></el-input>
                    </el-form-item>
                
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="addForm.mobile"></el-input>
                    </el-form-item>
                </el-form>

                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addUser()">确 定</el-button>
                </span>
            </el-dialog>

            <!-- show edit dialog -->
            <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
                <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" lable-width="70px">
                    <el-form-item label="用户名">
                        <el-input v-model="editForm.username" disabled></el-input>
                    </el-form-item>
                
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="editForm.email"></el-input>
                    </el-form-item>
                
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="editForm.mobile"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editUserInfo">确 定</el-button>
                </span>
            </el-dialog>

            <!-- show set role dialog -->
            <el-dialog title="分配权限" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClosed">
                <div>
                    <p>当前的用户：{{userInfo.username}}</p>
                    <p>当前的角色：{{userInfo.role_name}}</p>
                    <p>分配新角色：
                        <el-select v-model="selectedRoleId" placeholder="请选择">
                            <el-option
                            v-for="item in rolesList"
                            :key="item.id"
                            :label="item.roleName"
                            :value="item.id">
                            </el-option>
                        </el-select>
                    </p>
                </div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="saveRoleInfo()">确 定</el-button>
                </span>
            </el-dialog>

        </el-card>
    </div>
</template>

<script>
export default {
    data () {
        var checkEmail = (rule,value,cb) => {
            //验证邮箱正则表达式
            const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
            if(regEmail.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法邮箱'))
        }

        var checkMobile = (rule,value,cb) => {
            //验证手机号的正则
            const regMobile = /^(0[86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
            if(regMobile.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法手机号'))
        }

        return {
            //获取用户列表的参数
            querryInfo: {
                querry: '',
                //当前页数
                pagenum: 1,
                //当前页显示多少数据
                pagesize: 2
            },
            userList: [],
            total: 0,
            //控制对话框
            addDialogVisible:false,
            //添加用户的表单数据
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            //添加表单的验证规则对象
            addFormRules: {
                username: [
                    { required: true, message: '请输入用户名', trigger: 'blur' },
                    { min: 3, max: 10, message: '用户名长度在3到10个字符之间', trigger: 'blur' }
                ],
                password: [
                    { required: true, message: '请输入密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '密码长度在6到15个字符之间', trigger: 'blur' }
                ],
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur'}
                ]
            },
            editDialogVisible:false,
            //查询到的用户对象
            editForm: {},
            editFormRules: {
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur'}
                ],
                mobile: [
                    { required: true, message: '请输入手机', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur'}
                ]
            },
            setRoleDialogVisible:false,
            //需要被分配角色的用户信息
            userInfo: {},
            //所有角色的数据列表
            rolesList: {},
            selectedRoleId: ''
        }
    },
    created () {
        this.getUserList();
    },
    methods: {
        async getUserList() {
            const {data:res} = await this.$http.get('users',{params:this.querryInfo})
            if(res.meta.status !== 200) return this.$message.error('用户获取失败');
            this.userList = res.data.users;
            this.total = res.data.total;
        },
        //监听pagesize改变事件
        handleSizeChange(newSize) {
            this.querryInfo.pagesize = newSize;
            this.getUserList();
        },
        //监听页码值改变的事件
        handleCurrentChange(newPage){
            this.querryInfo.pagenum = newPage;
            this.getUserList();
        },
        //监听switch开关状态
        async userStateChange(userinfo) {
           const {data: res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`);
            if(res.meta.status !== 200) {
                userinfo.mg_state = !userinfo.mg_state;
                return this.$message.error('更新用户状态失败');
            }
            this.$message.success('更新成功！');
        },
        //监听对话框关闭
        addDialogClosed() {
            this.$refs.addFormRef.resetFields();
        },
        addUser() {
            //console.log(this.$refs.addFormRef);
            this.$refs.addFormRef.validate(async valid => {
                if(!valid) return ;
                const {data:res} = await this.$http.post('users',this.addForm);
                if(res.meta.status !== 201) {
                    return this.$message.error('add user error');
                }
                this.$message.success('add user success');
                this.addDialogVisible = false;
                this.getUserList();
            })
        },
        //展示给用户的对话框
        async showEditDialog(id) {
            const {data:res} = await this.$http.get('users/'+id);
            if(res.meta.status !== 200) {
                return this.$message.error('find user error');
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
                const {data:res} = await this.$http.put('users/'+this.editForm.id,{
                    email:this.editForm.email,mobile:this.editForm.mobile
                });
                if(res.meta.status !== 200) {
                    return this.$message.error('edit user error');
                }
                this.$message.success('edit user success');
                this.editDialogVisible = false;
                this.getUserList();
            })
        },
        async removeUserById(id) {
            //询问用户是否删除
            const confirmRes = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            //如果用户确认删除，则返回的字符串为confirm
            //如果用户取消删除，则返回值为字符串cancel
            if(confirmRes !== 'confirm') {
                return this.$message.info('已取消删除')
            }
            const {data:res} = await this.$http.delete('users/'+id)

            if(res.meta.status !== 200) return this.$message.error('删除失败')
            this.$message.success('删除成功')
            this.getUserList()
        },
        //展示分配角色对话框
        async setRole(role) {
            this.userInfo = role;
            const {data:res} = await this.$http.get('roles');
            if(res.meta.status !== 200) return this.$message.error('error');
            this.rolesList = res.data;

            this.setRoleDialogVisible = true;
        },
        async saveRoleInfo() {
            if(!this.selectedRoleId) return this.$message.error('at least choose one role');
            const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectedRoleId})
            if(res.meta.status !== 200) return this.$message.error('error');
            this.$message.success('success');
            this.getUserList();
            this.setRoleDialogVisible = false;
        },
        //分配角色对话框的关闭事件
        setRoleDialogClosed() {
            this.selectedRoleId = '';
            this.userInfo = {}
        }
    }
}
</script>

<style lang="less" scoped>

</style>