<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="10">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserList">
            </el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <!-- stripe 条纹属性表格 border边框 -->
      <el-table :data="userList" stripe border>
        <el-table-column label="序号" type="index"> </el-table-column>
        <el-table-column label="姓名" prop="username"> </el-table-column>
        <el-table-column label="邮箱" prop="email"> </el-table-column>
        <el-table-column label="电话" prop="mobile"> </el-table-column>
        <el-table-column label="角色" prop="role_name"> </el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <!-- 作用域插槽 -->
            <!-- 监听状态改变同步到数据库中 -->
            <el-switch
              @change="userStateChanged(scope.row)"
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="250px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域
       -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog
      @close="addDialogClose"
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
    >
      <!-- 内容主体区域 -->

      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
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
      <!-- 内容主体区域 -->

      <!-- 底部区域 -->

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定 </el-button>
      </span>
      <!-- 底部区域end
       -->
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主体区域 -->

      <el-form
        ref="editFormRef"
        :rules="editFormRules"
        :model="editForm"
        label-width="70px"
      >
        <el-form-item label="用户名" disabled>
          <el-input v-model="editForm.username"></el-input>
        </el-form-item>

        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 内容主体区域 -->

      <!-- 底部区域 -->

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定 </el-button>
      </span>
      <!-- 底部区域end
       -->
    </el-dialog>
  </div>
</template>


<script>
export default {
  data() {
    //验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      //      验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))

    }
    //验证手机号的规则

    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法手机号'))
    }
    return {




      //获取用户列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      //控制用户对话框的显示与隐藏
      addDialogVisible: false,
      //添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      ///添加用户的验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名在3~10个字符之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '用户名在6~15个字符之间', trigger: 'blur' }
        ],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' },
          , {
          validator: checkEmail, trigger: 'blur'
        }],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' }, {
            validator: checkMobile, trigger: 'blur'
          }
        ]

      }
      ,
      //编辑用户信息
      //定义用户对话框的修改与显示
      editDialogVisible: false,
      editForm: {

      },
      //编辑用户信息验证规则
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' },
          , {
          validator: checkEmail, trigger: 'blur'
        }],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' }, {
            validator: checkMobile, trigger: 'blur'
          }
        ]

      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败!')
      }
      this.userList = res.data.users
      this.total = res.data.total


    },
    handleSizeChange(newSize) {
      // 监控更改页面大小
      this.queryInfo.pagesize = newSize
      // 重新获取数据
      this.getUserList()

    },
    handleCurrentChange(newPage) {
      //监听页码值改变的事件
      this.queryInfo.pagenum = newPage
      // 重新获取数据
      this.getUserList()

    },
    //监听switch开关状态变化
    async userStateChanged(userInfo) {
      //查看后端接口 put请求 ：uid =》${userInfo.id}  ：type =》${userInfo.mg_state}   动态接收参数
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败！')
      }
      this.$message.success('更新用户数据成功')
    }
    ,
    //监听添加用户对话框的关闭事件
    addDialogClose() {
      //重置
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮添加新用户
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        //可以发起真正的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)

        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败!')
        }
        this.$message.success('添加用户成功!')
        this.addDialogVisible = false
        //重新获取用户列表数据
        this.getUserList()
      })


    },
    //展示修改用户的对话框
    async showEditDialog(id) {
      // console.log(id)
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('查询用户失败!')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    }
    ,//关闭修改对话框时,重置操作
    editDialogClosed() {
      //修改表单的重置操作
      this.$refs.editFormRef.resetFields()
    }
    ,
    // 修改用户信息,并提交
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        //判断预校验是否通过
        if (!valid) return
        //通过 则发起修改用户信息的操作
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        // console.log(res)
        if (res.meta.status !== 200) {
          this.$message.error('更新用户信息失败!')
        }
        //1 关闭对话框
        this.editDialogVisible = false
        //2 刷新对话列表
        this.getUserList()
        //3 提示修改成功
        this.$message.success('更新用户信息成功')
      })
    }
    ,//删除用户
    async removeUserById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)


      //用户点击删除确定  confirmResult返回值confirm

      //如果用户取消删除 则返回值为 cancel
      // console.log(confirmResult)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      //发起请求删除用户信息
      const { data: res } = await this.$http.delete('users/' + id)

      if (res.meta.status !== 200)
        return this.$message.error('用户删除失败')
      this.$message.success('确认删除')
      //刷新用户列表
      this.getUserList()

    }


  }

}
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 8px;
  font-size: 12px;
  text-align: center;
}
</style>