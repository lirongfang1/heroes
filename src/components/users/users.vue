<template>
  <el-card class="box-card">
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item>首页</el-breadcrumb-item>
    <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-row>
        <el-col>
            <div style="margin-top: 15px;">
                <el-input @clear="loadUserList()" clearable  placeholder="请输入内容" v-model="query" class="inputSearch">
                <el-button   @click="searchUser()" slot="append" icon="el-icon-search"></el-button>
                </el-input>
                <el-button type="primary" @click="showAddUserDia">添加用户</el-button>
            </div>
        </el-col>
    </el-row>
     <el-table
      :data="userList"
      style="width: 100%" height='300px'>
      <el-table-column
        type='index'
        label="#"
        width="60">
      </el-table-column>
      <el-table-column
        prop="username"
        label="姓名"
        width="80">
      </el-table-column>
      <el-table-column
        prop="email"
        label="邮箱">
      </el-table-column>
       <el-table-column
        prop="mobile"
        label="电话">
      </el-table-column>
       <el-table-column
        prop="create_time"
        label="创建时间">
        <!-- 两个组件 -->
          <template slot-scope="userList">
             {{userList.row.create_time|format}}
          </template>
      </el-table-column>
       <el-table-column
        label="用户状态">
          <template slot-scope="scope">
            <el-switch
            @change="changeState(scope.row)"
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949">
            </el-switch>
          </template>
      </el-table-column> 
      <el-table-column
        prop="address"
        label="操作">
         <template slot-scope="scope"> 
            <el-button size="mini" plain type="primary" icon="el-icon-edit" @click='showEditdata(scope.row)' circle></el-button>
            <el-button size="mini" plain type="danger" @click='showDeleteUser(scope.row.id)' icon="el-icon-delete" circle></el-button>
            <el-button size="mini" plain type="success" @click="showSetrole(scope.row)" icon="el-icon-check" circle></el-button>
         </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <!-- 添加用户 -->
  <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd" >
  <el-form :model="form">
    <el-form-item label="用户名" label-width="100px">
        <el-input v-model="form.username" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="密码"  label-width="100px">
        <el-input v-model="form.password" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="邮箱" label-width="100px">
        <el-input v-model="form.email" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="电话" label-width="100px">
        <el-input v-model="form.mobile" autocomplete="off"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
     <el-button @click="dialogFormVisibleAdd = false">取消</el-button>
     <el-button type="primary" @click="addFormdata()">确 定</el-button>
  </div>
  </el-dialog>
  <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit" >
  <el-form :model="form">
    <el-form-item label="用户名" label-width="100px">
        <el-input v-model="form.username" disabled autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="邮箱" label-width="100px">
        <el-input v-model="form.email" autocomplete="off"></el-input>
    </el-form-item>
    <el-form-item label="电话" label-width="100px">
        <el-input v-model="form.mobile" autocomplete="off"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
     <el-button @click="dialogFormVisibleEdit = false">取消</el-button>
     <el-button type="primary" @click="editFormdata()">确 定</el-button>
  </div>
  </el-dialog>
   <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRol">
  <el-form :model="form">
    <el-form-item label="用户名" label-width="100px">
      {{currUserName}}
    </el-form-item>
    <el-form-item label="角色" label-width="100px">
      <el-select v-model="vauleId">
        <el-option label="请选择" :value="-1"></el-option>
        <el-option :label="item.roleName" :value="item.id"
           v-for="(item,i) in roles" :key='i'>    
        </el-option>   
      </el-select>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="dialogFormVisibleRol = false">取 消</el-button>
    <el-button type="primary" @click="setRole()">确 定</el-button>
  </div>
  </el-dialog>
  </el-card>
</template>
<script>
export default {
   data(){
     return{
         query:'',
         pagenum:1,
         pagesize:2,
         userList:[],
         total:-1,
         dialogFormVisibleAdd:false,
         dialogFormVisibleEdit:false,
         dialogFormVisibleRol:false,
         form:{
           username:'',
           password:'',
           email:'',
           mobile:'',
         },
         vauleId:-1,
         currUserName:-1,
         currUserId:-1,
         roles:[],   
     }
   },
   created(){
       this.getUserList();
   },
   methods:{
    //  打开对话框
      async showSetrole(user){
         this.currUserName=user.username; 
         this.currUserId = user.id
         const  res1=await this.$http.get(`roles`)
         this.roles=res1.data.data
        //  console.log(this.roles)
         const  res=await this.$http.get(`users/${user.id}`)
         this.vauleId=res.data.data.rid
        //  console.log(res)
         this.dialogFormVisibleRol=true;
     },
    //  分配角色
     async setRole(){
         const res=await this.$http.put(`users/${this.currUserId}/role`,{
           rid:this.vauleId
         }) 
         console.log(res)
         this.dialogFormVisibleRol=false;
     },
     async changeState(user){
       const res=await this.$http.put(`users/${user.id}/state/$(user.mg_state)`)
      //  console.log(res)
      const {meta:{msg,status}}=res.data
      if(status){
           this.$message.success(msg)
      }
     },
     showEditdata(user){
      //  console.log(user)
        this.form=user;
        this.dialogFormVisibleEdit=true;
     },
     async editFormdata(){
         const res=await this.$http.put(`users/${this.form.id}`,this.form)
        console.log(res)
        this.dialogFormVisibleEdit=false;
       if(res.data.meta.status===200){
          this.$message({
              type: 'success',
              message: res.data.meta.msg
            })
       }
       this.getUserList();
     },
    //  删除
     showDeleteUser(userId){
     this.$confirm('删除用户?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
       .then(async () => {
          // 发送删除的请求 :id----> 用户id
          // 1. data中找userId  X
          // 2. 把userId以showDeleUserMsgBox参数形式传进来
          const res = await this.$http.delete(`users/${userId}`)
          console.log(res)
          if (res.data.meta.status === 200) {
            this.pagenum = 1
            // 更新视图
            this.getUserList()
            // 提示
            this.$message({
              type: 'success',
              message: res.data.meta.msg
            })
          }
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
     },
    //  添加
    async addFormdata(){
        const res=await this.$http.post('users',this.form)
        console.log(res)
        this.dialogFormVisibleAdd = false;
        this.userList();
     },
     showAddUserDia(){
       this.dialogFormVisibleAdd=true
     },
     loadUserList(){
        this.getUserList()
     },
     searchUser(){
       this.getUserList()
     },
     handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.pagesize=val;
        this.pagenum=1;
        this.getUserList();
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.pagenum=val;
        this.getUserList();
      },
     async getUserList(){
        const AUTH_TOKEN=localStorage.getItem('token')
        this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN
           const res=await this.$http.get(`users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`)
            console.log(res.data)
            const {
              data:{users,total},meta:{status,msg}
              }=res.data
            if(status===200){
               this.userList=users
               this.total=total
               this.$message.success(msg)
            }else{
               this.$message.warning(msg)
            } 
      }
   }
}
</script>

<style>
 .box-card{
        height:100%;
    }
 .inputSearch{
     width:320px;
 }
</style>
