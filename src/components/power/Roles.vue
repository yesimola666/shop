<template>
<div>
    <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-button type="primary">添加角色</el-button>
        <el-table :data="rolesList" style="width: 100%">
            <el-table-column type="expand" label="展开">
                <template slot-scope="scope">
                    <el-row style="background-color:#e5e5eb;margin-bottom:20px;padding:10px;" v-for="item1 in scope.row.children" :key="item1.id">
                        <el-col :span="5">
                            <el-tag type="primary" @close="removePower(scope.row,item1.id)" closable>{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <el-col :span="19">
                            <div style="background-color:#c9bbc2;margin-bottom:10px;" v-for="item2 in item1.children" :key="item2.id">
                                <el-row>
                                    <el-col :span="5">
                                        <el-tag type="warning" @close="removePower(scope.row,item2.id)" closable>{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="19">
                                        <el-tag type="danger" closable v-for="item3 in item2.children" :key="item3.id" @close="removePower(scope.row,item3.id)">{{item3.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </div>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit"></el-button>
                    <el-button type="danger" icon="el-icon-delete"></el-button>
                    <el-tooltip effect="dark" content="分配权限" placement="top" :enterable="false">
                        <el-button type="warning" icon="el-icon-setting" @click="getRightTree(scope.row)"></el-button>
                    </el-tooltip>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <!-- 添加分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="dialogRightVisible" width="30%">
        <!-- 树形控件 -->
        <el-tree :default-checked-keys="selectedKey" node-key="id" :data="rightsList" :props="defaultProps" show-checkbox default-expand-all>
        </el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogRightVisible = false">取 消</el-button>
            <el-button type="primary" @click="dialogRightVisible = false">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      dialogRightVisible: false,
      rightsList: [],
      defaultProps: {
        label: 'authName'
      },
      selectedKey: []
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const {
        data: res
      } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.message.error('获取角色列表失败')
      this.rolesList = res.data
      // console.log(this.rolesList)
    },
    removePower (role, id) {
      this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const {
          data: res
        } = await this.$http.delete(`roles/${role.id}/rights/${id}`)
        if (res.meta.status !== 200) return this.$message.error('删除失败！')
        role.children = res.data
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    async getRightTree (role) {
      this.dialogRightVisible = true
      const {
        data: res
      } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.message.error('获取失败')
      this.rightsList = res.data
      // const arr = []
      // if (role.children.length > 0) {
      //   role.children.forEach(item1 => {
      //     if (item1.children.length > 0) {
      //       item1.children.forEach(item2 => {
      //         if (item2.children.length > 0) {
      //           item2.children.forEach(item3 => {
      //             arr.push(item3.id)
      //           })
      //         }
      //       })
      //     }
      //   })
      // }
      // this.selectedKey = arr
      const arr = []
      function fn (obj) {
        if (obj.children) {
          obj.children.forEach(item => {
            fn(item)
          })
        } else {
          arr.push(obj.id)
        }
      }
      fn(role)
      this.selectedKey = arr
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 20px;
}
</style>
