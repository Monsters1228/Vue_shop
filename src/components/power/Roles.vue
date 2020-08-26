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
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>

            <el-table :data="rolesList" border stripe>
                <!-- 展开列 -->
                 <el-table-column type="expand">
                     <template slot-scope="scope">
                        <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                            <!-- 一级权限 -->
                            <el-col :span="5">
                               <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                               <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 二级和三级权限 -->
                            <el-col :span="19">
                                <!-- 二级权限 -->
                                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <!-- 三级权限 -->
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="(item3) in item2.children" 
                                        :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                     </template>
                 </el-table-column>
                 <el-table-column type="index" label="#"></el-table-column>
                 <el-table-column label="角色名称" prop="roleName" ></el-table-column>
                 <el-table-column label="角色描述" prop="roleDesc" ></el-table-column>
                 <el-table-column label="操作" width="300px">
                     <template slot-scope="scope">
                       <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                       <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
                       <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">权限分配</el-button>
                     </template>
                 </el-table-column>
            </el-table>
        </el-card>
        <!-- 分配权限对话框 -->
            <el-dialog
                title="分配权限"
                :visible.sync="setRightiDalogVisible" @close="setRightDialogClose"
                width="30%">
                <!-- 树形控件 -->
                <el-tree :data="rightslist" :props="treeProps" show-checkbox 
                node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>

                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRightiDalogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="allotRights">确 定</el-button>
                </span>
            </el-dialog>
        <!-- 添加用户 -->
            <el-dialog title="添加用户" :visible.sync="addDialogVisible"
                width="30%" @close="addClear">
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                    <el-form-item label="角色名称" prop="roleName" label-width="100px">
                        <el-input v-model="addForm.roleName"></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc" label-width="100px">
                        <el-input v-model="addForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>

                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addUser">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 修改用户 -->
            <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" @close="editDialogClosed"
                width="40%">
                 <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">
                    <el-form-item label="角色名称" prop="roleName" label-width="100px">
                        <el-input v-model="editForm.roleName"></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述" prop="roleDesc" label-width="100px">
                        <el-input v-model="editForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editUserInfo" >确 定</el-button>
                </span>
            </el-dialog>

    </div>
</template> 

<script>
export default {
    data(){
        return{
            rolesList: [],
            addDialogVisible: false,
            editDialogVisible: false,
            addForm:{
                roleName:'',
                roleDesc:''
            },
            editForm:{},
            addFormRules:{
                roleName: [
                    { required: true, message: "请输入角色名称", trigger: "blur" }
                ],
                roleDesc: [
                    { required: true, message: "请输入角色描述", trigger: "blur" }
                ]
            },
            setRightiDalogVisible: false,
            rightslist:[],
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            //默认选中
            defKeys: [],
            roleId: ''
        }
    },
    created(){
        this.getRolesList();
    },
    methods:{
        async getRolesList(){
            const {data:res} = await this.$http('roles/');
            if(res.meta.status !==200){
                return this.$message.error('获取权限列表失败');
            }

            this.rolesList = res.data;
            console.log(this.rolesList);   
        },
        addClear(){
            this.$refs.addFormRef.resetFields();
        },
        //点击按钮添加角色
        addUser(){
            this.$refs.addFormRef.validate(async valid =>{
                if(!valid) return;
                //发起添加用户的网络请求
               const {data:res} = await this.$http.post('roles', this.addForm)
               if(res.meta.status !== 201){
                   this.$message.error('添加用户失败');
               }else{
                   this.$message.success('添加用户成功');
                   this.addDialogVisible = false;
                   this.getRolesList();
               }
            })
        },
        //根据id查询角色
        async showEditDialog(id){
            const {data:res} = await this.$http.get('roles/' + id)
            if(res.meta.status !== 200){
                   return this.$message.error('查询用户信息失败');
            }
            this.editForm = res.data;
            console.log(this.editForm.id);
            this.editDialogVisible = true;
        },
        editDialogClosed(){
            this.$refs.editFormRef.resetFields();
        },
        editUserInfo(){
            this.$refs.editFormRef.validate(async valid =>{
                if(!valid) return;
                //发起修改用户的网络请求
               const {data:res} = await this.$http.put('roles/' + this.editForm.roleId, {
                   roleName: this.editForm.roleName,
                   roleDesc: this.editForm.roleDesc
               })
               if(res.meta.status !== 200){
                   this.$message.error('编辑用户失败');
               }else{
                   this.$message.success('编辑用户成功');
                   //关闭对话框
                   this.editDialogVisible = false;
                   //更新数据列表
                   this.getRolesList();
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
            
            const {data:res} = await this.$http.delete('roles/' + id);

            if(res.meta.status !== 200){
                return this.$message.error('删除用户失败');
            }
            this.$message.success('删除用户成功');
            this.getRolesList();
        },
        //删除权限
        async removeRightById(role, id){
            console.log(11);
            const confirmResult = await this.$confirm
            ('此操作将永久删除该权限, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err)
            
            if(confirmResult !=='confirm'){
                return this.$message.info('已取消');
            }
            
            const {data:res} = await 
            this.$http.delete(`roles/${role.id}/rights/${id}`);

            if(res.meta.status !== 200){
                return this.$message.error('删除失败');
            }
            this.$message.success('删除成功');
            //更新
            role.children = res.data;
        },
        //权限分配
        async showSetRightDialog(role){
            this.roleId = role.id;
            //获取所有权限数据
            const {data:res} = await this.$http.get('rights/tree')
            if(res.meta.status !== 200){
                return this.$message.error('获取权限数据失败');
            }
            this.rightslist = res.data;
            console.log(this.rightslist)

            this.getLeafKeys(role, this.defKeys)
            this.setRightiDalogVisible = true;

        },
        getLeafKeys(node, arr){
            if(!node.children){
                return arr.push(node.id);
            }
            node.children.forEach(item => {
                this.getLeafKeys(item, arr);
            })
        },
        setRightDialogClose(){
            this.defKeys = [];
        },
        //分配权限
        async allotRights(){
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ] 
            
            const idStr = keys.join(',');

            const {data:res} = await this.$http.post(
                `roles/${this.roleId}/rights`,
                { rids: idStr}
            )
            console.log(res);
            if(res.meta.status !== 200){
                return this.$message.error('分配权限失败');
            }
            this.$message.success('分配权限成功');
            this.getRolesList();
            this.setRightiDalogVisible = false;
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag{
    margin: 7px;
}
.bdtop{
    border-top: 1px solid #eee;
}

.bdbottom{
    border-bottom: 1px solid #eee;
}
.vcenter{
    display: flex;
    align-items: center;
}
</style>