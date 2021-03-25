<template>
<div>
    <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row :gutter="20">
            <el-col :span="8">
                <el-input v-model="query.query" clearable @clear="getUserList">
                    <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                </el-input>
            </el-col>
            <el-col :span="16">
                <el-button type="primary" @click="addDilogVisible = true">添加用户</el-button>
            </el-col>
        </el-row>
        <el-table :data="userList" stripe border>
            <el-table-column type="index" label="#" />
            <el-table-column prop="username" label="姓名" />
            <el-table-column prop="email" label="邮箱" />
            <el-table-column prop="mobile" label="电话" />
            <el-table-column prop="role_name" label="角色" />
            <el-table-column prop="mg_state" label="状态">
                <template slot-scope="scope">
                    <el-switch v-model="scope.row.mg_state" @change="userStateFn(scope.row)">
                    </el-switch>
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" @click="showEditDilog(scope.row.id)"></el-button>
                    <el-button type="danger" icon="el-icon-delete" @click="removeUserById(scope.row.id)"></el-button>
                    <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                        <el-button type="warning" icon="el-icon-setting"></el-button>
                    </el-tooltip>
                </template>
            </el-table-column>
        </el-table>
        <div class="block">
            <span class="demonstration">完整功能</span>
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="query.pagenum" :page-sizes="[1,2,3]" :page-size="query.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
            </el-pagination>
        </div>
    </el-card>
    <!-- 添加用户 -->
    <el-dialog title="添加用户" :visible.sync="addDilogVisible" width="50%" @close="addDilogClosed">
        <el-form label-width="80px" :model="addForm" ref="addFormRef" :rules="addFormRules">
            <el-form-item label="用户名" prop="username">
                <el-input v-model="addForm.username"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
                <el-input v-model="addForm.password"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
                <el-input v-model="addForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机号" prop="mobile">
                <el-input v-model="addForm.mobile"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDilogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
    </el-dialog>
    <!-- 修改用户 -->
    <el-dialog title="修改用户" :visible.sync="editDilogVisible" width="50%" @close="resetEditForm">
        <el-form label-width="80px" :model="editForm" :rules="editFormRules" ref="editFormRef">
            <el-form-item label="用户名">
                <el-input disabled v-model="editForm.username"></el-input>
            </el-form-item>

            <el-form-item label="邮箱" prop="email">
                <el-input v-model="editForm.email"></el-input>
            </el-form-item>

            <el-form-item label="手机号" prop="mobile">
                <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDilogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
    </el-dialog>
    <template>
</template>
</div>
</template>

<script>
export default {
  data () {
    const checkEmail = (rule, value, callback) => {
      const reg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (!reg.test(value)) {
        return callback(new Error('邮箱格式不对'))
      }
      callback() // 校验通过
    }

    const checkMobile = (rule, value, callback) => {
      const reg = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (!reg.test(value)) {
        return callback(new Error('手机格式不对'))
      }
      callback() // 校验通过
    }

    return {
      query: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userList: [],
      total: 0,
      addDilogVisible: false,
      editDilogVisible: false,
      addForm: {
        usernmae: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [{
          required: true,
          message: '请输入登录名称',
          trigger: 'blur'
        },
        {
          min: 3,
          max: 10,
          message: '长度在3到10个字符',
          trigger: 'blur'
        }
        ],
        password: [{
          required: true,
          message: '请输入登录密码',
          trigger: 'blur'
        },
        {
          min: 6,
          max: 15,
          message: '长度在6到15个字符',
          trigger: 'blur'
        }
        ],
        email: [{
          required: true,
          message: '请输入邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        mobile: [{
          required: true,
          message: '请输入手机号',
          trigger: 'blur'
        },
        {
          validator: checkMobile,
          trigger: 'blur'
        }
        ]
      },
      editForm: {},
      editFormRules: {
        email: [{
          required: true,
          message: '请输入邮箱',
          trigger: 'blur'
        },
        {
          validator: checkEmail,
          trigger: 'blur'
        }
        ],
        mobile: [{
          required: true,
          message: '请输入手机号',
          trigger: 'blur'
        },
        {
          validator: checkMobile,
          trigger: 'blur'
        }
        ]
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {

    async getUserList () {
      const {
        data: res
      } = await this.$http.get('users', {
        params: this.query
      })
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange (curSize) {
      this.query.pagesize = curSize
      this.getUserList()
    },
    handleCurrentChange (cuePageNum) {
      this.query.pagenum = cuePageNum
      this.getUserList()
    },
    async userStateFn (row) {
      const {
        data: res
      } = await this.$http.put(`users/${row.id}/state/${row.mg_state}`)
      if (res.meta.status !== 200) {
        this.$message.error(res.meta.msg)
        row.mg_state = !row.mg_state
      }
      this.$message.success(res.meta.msg)
    },
    addDilogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const {
          data: res
        } = await this.$http.post('users', this.addForm)
        if (res.status !== 200) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        this.addDilogVisible = false
        this.getUserList()
      })
    },
    async showEditDilog (id) {
      this.editDilogVisible = true
      // console.log(id)
      const {
        data: res
      } = await this.$http.get(`users/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('查询用户失败！')
      this.editForm = res.data
    },
    resetEditForm () {
      this.$refs.editFormRef.resetFields()
    },
    editUser () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return console.log('校验失败！')
        const { data: res } = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        console.log(res)
        if (res.meta.status !== 200) return this.$message.error('修改信息失败')
        this.editDilogVisible = false
        this.getUserList()
        this.$message.success('更新成功')
      })
    },
    async removeUserById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`users/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败！')
      this.getUserList()
      this.$message.success('删除成功！')
    }
  }
}
</script>

<style lang="less" scoped>

</style>
