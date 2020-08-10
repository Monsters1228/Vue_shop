<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图 -->
        <el-card class="box-card">
            <!-- 搜索框 -->
            <el-row :gutter="20">
                <el-col :span="7">
                    <el-input placeholder="请输入内容" v-model="queryIfo.query" clearable @click="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row> 

            <!-- 添加用户 -->
            <el-dialog title="添加用户" :visible.sync="addDialogVisible"
                width="30%" @close="addClear">
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="addForm.username"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password">
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
                    <el-button type="primary" @click="addUser">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 修改用户 -->
            <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" @close="editDialogClosed"
                width="30%">
                 <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="editForm.username" :disabled="true"></el-input>
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
                    <el-button type="primary" @click="editUserInfo" >确 定</el-button>
                </span>
            </el-dialog>

            <!-- 用户列表区 -->
            <el-table :data="userlist" border stripe>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态">
                    <template slot-scope="scope">
                        <!-- {{scope.row}} -->
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" >
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="min" @click="showEditDialog(scope.row.id)"></el-button>
                        <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="min" @click="removeUserById(scope.row.id)"></el-button>
                        <!-- 分配角色按钮 -->
                        <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="min"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination
                @size-change="handleSizeChange" @current-change="handleCurrentChange" 
                :current-page="queryIfo.pagenum"
                :page-sizes="[1, 2, 5, 10]" :page-size="queryIfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </el-card>
    </div>
</template>

<script>
export default {
    data(){
        //验证邮箱规则
        var checkEamil = (rule, value, cb) =>{
            const regEmail = /^(\w-*\.*)+@(\w-?)+(\.\w{2,})+$/

            if(regEmail.test(value)){
                return cb();
            }
            cb(new Error('请输入正确的邮箱'));
        }
            
        // 验证手机号规则
        const checkMobile = (rule, value, cb) => {
            let reg = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
            if (reg.test(value)) {
                cb()
            } else {
                cb(new Error('手机号码格式不正确'))
            }
        }

        return{
            //获取用户列表的参数对象
            queryIfo:{
                query: '',
                pagenum: 1,
                pagesize: 2
            },
            userlist: [],
            total: 0,
            addDialogVisible: false,
            addForm: {
                username: '',
                password: '',
                email: '',
                mobile: ''
            },
            addFormRules:{
                username: [
                    { required: true, message: "请输入用户名", trigger: "blur" },
                    {
                        min: 3,
                        max: 10,
                        message: "长度在 3 到 11 个字符",
                        trigger: "blur",
                    },
                ],
                password: [
                    { required: true, message: "请输入密码", trigger: "blur" },
                    {
                        min: 6,
                        max: 16,
                        message: "长度在 6 到 16 个字符",
                        trigger: "blur",
                    },
                ],
                email: [
                    { required: true, message: "请输入邮箱", trigger: "blur" },
                    { validator: checkEamil, trigger: "blur"}
                ],
                mobile: [
                    { required: true, message: "请输入电话", trigger: "blur" },
                    { validator: checkMobile, trigger: "blur"}
                ]
            },
            editDialogVisible: false,
            //查询的用户信息
            editForm:{}
        }
    },
    created(){
        this.getUserList();
    },
    methods:{
        async getUserList(){
            const { data:res} = await this.$http.get('users', { params: this.queryIfo } );
            if(res.meta.status !== 200){
                return this.$message.error('获取用户列表失败');
            }
            this.userlist = res.data.users;
            this.total = res.data.total;
            console.log(res);
        },
        handleSizeChange(newSize){
            this.queryIfo.pagesize = newSize;
            this.getUserList();
        },
        handleCurrentChange(newPage){
            this.queryIfo.pagenum = newPage;
            this.getUserList();
        },
        async userStateChange(userinof){
           const {data:res} = await this.$http.put(`users/${userinof.id}/state/${userinof.mg_state}`)
           if(res.meta.status !== 200){
               userinof.mg_state = !userinof.mg_state;
               return this.$message.error('更新用户状态失败');
            }
            this.$message.success('更新用户状态成功');
        },
        addClear(){
            this.$refs.addFormRef.resetFields();
        },
        //点击按钮添加用户
        addUser(){
            this.$refs.addFormRef.validate(async valid =>{
                if(!valid) return;
                //发起添加用户的网络请求
               const {data:res} = await this.$http.post('users', this.addForm)
               if(res.meta.status !== 201){
                   this.$message.error('添加用户失败');
               }else{
                   this.$message.success('添加用户成功');
                   this.addDialogVisible = false;
                   this.getUserList();
               }
            })
        },
        //用户信息编辑
        async showEditDialog(id){
            const {data:res} = await this.$http.get('users/' + id)
            if(res.meta.status !== 200){
                   return this.$message.error('查询用户信息失败');
            }
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        editDialogClosed(){
              this.$refs.editFormRef.resetFields();
        },

        editUserInfo(){
            this.$refs.editFormRef.validate(async valid =>{
                if(!valid) return;
                //发起修改用户的网络请求
               const {data:res} = await this.$http.put('users/' + this.editForm.id, {
                   email: this.editForm.email,
                   mobile: this.editForm.mobile
               })
               if(res.meta.status !== 200){
                   this.$message.error('修改用户失败');
               }else{
                   this.$message.success('修改用户成功');
                   //关闭对话框
                   this.editDialogVisible = false;
                   //更新数据列表
                   this.getUserList();
               }
            })
        },
        //删除用户
        async removeUserById(id){
            const confirmResult = await this.$confirm
            ('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)

            if(confirmResult !=='confirm'){
                return this.$message.info('已取消');
            }
            
            const {data:res} = await this.$http.delete('users/' + id);

            if(res.meta.status !== 200){
                return this.$message.error('删除用户失败');
            }
            this.$message.success('删除用户成功');
            this.getUserList();
        }
    }
}
</script>

<style lang="less" scoped>

</style>