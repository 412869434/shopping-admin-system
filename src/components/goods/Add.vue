<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 提示区 -->
            <el-alert type="info" title="添加商品信息" center show-icon :closable="false"></el-alert>
        
            <!-- 步骤条 -->
            <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
                <el-step title="基本信息"></el-step>
                <el-step title="商品参数"></el-step>
                <el-step title="商品属性"></el-step>
                <el-step title="商品图片"></el-step>
                <el-step title="商品内容"></el-step>
                <el-step title="完成"></el-step>
            </el-steps>

            <!-- tab -->
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
                <el-tabs :tab-position="'left'" v-model="activeIndex" :before-leave="beforeTabLeave"
                    @tab-click="tabClicked">

                    <el-tab-pane label="基本信息" name="0">
                        <el-form-item label="商品名称" prop="goods_name">
                            <el-input v-model="addForm.goods_name"></el-input>
                        </el-form-item>
                        <el-form-item label="商品价格" prop="goods_price">
                            <el-input v-model="addForm.goods_price" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品重量" prop="goods_weight">
                            <el-input v-model="addForm.goods_weight" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品数量" prop="goods_number">
                            <el-input v-model="addForm.goods_number" type="number"></el-input>
                        </el-form-item>
                        <el-form-item label="商品分类" prop="goods_cat">
                            <el-cascader expand-trigger="hover" :props="cateProps"
                            :options="cateList" v-model="addForm.goods_cat"
                            @change="handleChange" >
                            </el-cascader>
                        </el-form-item>
                    </el-tab-pane>

                    <el-tab-pane label="商品参数" name="1">
                        <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
                            <!-- 复选框 -->
                            <el-checkbox-group v-model="item.attr_vals">
                                <el-checkbox border :label="cb" v-for="(cb,i) in item.attr_vals" :key="i"></el-checkbox>
                            </el-checkbox-group>
                        </el-form-item>
                    </el-tab-pane>

                    <el-tab-pane label="商品属性" name="2">
                        <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                            <el-input v-model="item.attr_vals"></el-input>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane label="商品图片" name="3">
                        <!-- action 表示图片上传的api地址 -->
                        <el-upload action="http://127.0.0.1:8888/api/private/v1/upload"
                        :on-preview="handlePreview" :on-remove="handleRemove"
                        list-type="picture" :headers="headerObj" :on-success="handleSuccess">
                        <el-button size="small" type="primary">点击上传</el-button>
                        </el-upload>
                    </el-tab-pane>

                    <el-tab-pane label="商品内容" name="4">
                        <!-- 富文本编辑器 -->
                        <quill-editor v-model="addForm.goods_introduce">

                        </quill-editor>

                        <!-- button -->
                        <el-button type="primary" class="btn" @click="add">添加商品</el-button>
                    </el-tab-pane>
                
                </el-tabs>
            </el-form>
        
        </el-card>

        <!-- 图片预览对话框 -->
        <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
            <img :src="previewPath" alt="" class="previewImg">

        </el-dialog>

    </div>
</template>

<script>
import _ from 'lodash'

export default {
    data() {
        return {
            activeIndex: '0',
            addForm: {
                goods_name: '',
                goods_price: 0,
                goods_weight: 0,
                goods_number: 0,
                //商品所属的分类数组
                goods_cat:[],
                //图片数组
                pics: [],
                goods_introduce: '',
                attrs: []
            },
            addFormRules: {
                goods_name: [
                    { required:true,message:'请输入商品名称',trigger:'blur'}
                ],
                goods_price: [
                    { required:true,message:'请输入商品价格',trigger:'blur'}
                ],
                goods_weight: [
                    { required:true,message:'请输入商品重量',trigger:'blur'}
                ],
                goods_number: [
                    { required:true,message:'请输入商品数量',trigger:'blur'}
                ],
                goods_cat: [
                    { required:true,message:'请输入商品分类',trigger:'blur'}
                ]
            },
            //商品分类列表
            cateList: [],
            cateProps:{
                label: 'cat_name',
                value: 'cat_id',
                children: 'children'
            },
            manyTableData: [],
            onlyTableData: [],
            //图片上传组件的请求头
            headerObj: {
                Authorization: window.sessionStorage.getItem('token')
            },
            previewPath: '',
            previewVisible:false,

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
            //this.total = res.data.total;
            //console.log(this.cateList);
        },
        handleChange() {
            if(this.addForm.goods_cat.length !== 3){
                this.addForm.goods_cat = [];
            }
        },
        beforeTabLeave(activeName,oldActiveName) {
            //console.log(activeName);
            if(oldActiveName === '0' && this.addForm.goods_cat.length !== 3){
                this.$message.error('error')
                return false
            }
        },
        async tabClicked() {
            //证明访问的是动态参数面板
            if(this.activeIndex === '1'){
                const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,
                {params: {sel:'many'}})
                if(res.meta.status !== 200) return this.$message.error('error');
                
                res.data.forEach(item => {
                    item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
                })
                this.manyTableData = res.data
            }
            else if (this.activeIndex === '2') {
                const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,
                {params: {sel:'only'}})
                if(res.meta.status !== 200) return this.$message.error('error');
            
                this.onlyTableData = res.data
            }
        },
        //处理图片预览效果
        handlePreview(file) {
            this.previewPath = file.response.data.url 
            this.previewVisible = true
        },
        //处理移除图片的操作
        handleRemove(file) {
            const filePath = file.response.data.tmp_path
            const i = this.addForm.pics.findIndex(x => x.pic === filePath)
            this.addForm.pics.splice(i,1)
        },
        //监听图片上传成功的函数
        handleSuccess(response) {
            console.log(111);
            console.log(response);
            //拼接得到一个图片信息对象
            //将图片信息对象push到数组中
            const picInfo = {pic: response.data.tmp_path}
            this.addForm.pics.push(picInfo)
        },
        add() {
            this.$refs.addFormRef.validate(async valid => {
                if(!valid) return this.$message.error('error')
                
                //lodash cloneDeep(obj) 深拷贝
                const form = _.cloneDeep(this.addForm)
                form.goods_cat = form.goods_cat.join(',')
                //处理动态参数
                this.manyTableData.forEach(item => {
                    const newInfo = {
                        attr_id: item.attr_id,
                        attr_value: item.attr_vals.join(' ')
                    }
                    this.addForm.attrs.push(newInfo)
                })
                //处理静态属性
                this.onlyTableData.forEach(item => {
                    const newInfo = {
                        attr_id: item.attr_id,
                        attr_value: item.attr_vals
                    }
                    this.addForm.attrs.push(newInfo)
                })
                form.attrs = this.addForm.attrs
                console.log(this.addForm);
                console.log(form);

                //发起请求
                const {data:res} = await this.$http.post('goods',form)
                if(res.meta.status !== 201) return this.$message.error('add goods error')
                this.$message.success('success')
                this.$router.push('/goods')
            })
        }
    },
    computed: {
        cateId() {
            return this.addForm.goods_cat.length === 3 ? this.addForm.goods_cat[2] : []
        }
    }
}
</script>

<style lang="less" scoped>
.el-steps {
    margin: 15px 0;
}

.el-step__title {
    font-size: 13px !important;
}

.el-checkbox {
    margin: 0 10px 0 0  !important;
}

.previewImg {
    width: 100%;
}

.btn {
    margin-top: 15px;
}

</style>