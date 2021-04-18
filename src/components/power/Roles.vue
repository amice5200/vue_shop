<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <template>
              <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                <el-col :span="5">
                  <el-tag closable @close="removeRightById(scope.row, item1.id)">{{ item1.authName }}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="19">
                  <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{ item2.authName }}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">
                        {{ item3.authName }}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
            <!-- <pre>
              {{ scope.row }}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRights(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 添加角色对话框 -->
      <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addDialogClosed">
        <el-form :model="addRolesForm" :rules="rolesFormRules" ref="addRolesFormRef" label-width="100px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRolesForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRolesForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑角色对话框 -->
      <el-dialog title="编辑角色" :visible.sync="editRoleDialogVisible" width="50%" @close="editDialogClosed">
        <el-form :model="editRolesForm" :rules="rolesFormRules" ref="editRolesFormRef" label-width="100px">
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRolesForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRolesForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editRoleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRole">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分配权限对话框 -->
      <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
        <el-tree ref="treeRef" :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeyArr"></el-tree>
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
      roleList: [],
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      setRightDialogVisible: false,
      addRolesForm: {
        roleName: '',
        roleDesc: ''
      },
      editRolesForm: {},
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKeyArr: [],
      // 即将分配权限的角色ID
      roleId: '',
      rolesFormRules: {
        roleName: [{ required: true, message: '请输入角色名称', trigger: 'blur' }],
        roleDesc: [{ required: true, message: '请输入角色描述', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败！')
      this.roleList = res.data
      console.log(this.roleList)
    },
    addRole() {
      this.$refs.addRolesFormRef.validate(async (valid) => {
        if (!valid) return false
        const { data: res } = await this.$http.post('roles', this.addRolesForm)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败！')
        this.$message.success('添加角色成功！')
        this.addRoleDialogVisible = false
        this.getRoleList()
      })
    },
    addDialogClosed() {
      this.$refs.addRolesFormRef.resetFields()
    },
    async showEditDialog(id) {
      const { data: res } = await this.$http.get(`roles/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败！')
      this.editRolesForm = res.data
      this.editRoleDialogVisible = true
    },
    editRole() {
      this.$refs.editRolesFormRef.validate(async (valid) => {
        if (!valid) return false
        const { data: res } = await this.$http.put(`roles/${this.editRolesForm.roleId}`, {
          roleName: this.editRolesForm.roleName,
          roleDesc: this.editRolesForm.roleDesc
        })
        if (res.meta.status !== 200) return this.$message.error('修改角色信息失败！')
        this.editRoleDialogVisible = false
        this.getRoleList()
        this.$message.success('修改角色信息成功！')
      })
    },
    editDialogClosed() {
      this.$refs.editRolesFormRef.resetFields()
    },
    async removeRoleById(id) {
      const result = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)
      // console.log(result)
      if (result !== 'confirm') return this.$message.info('已取消删除！')

      const { data: res } = await this.$http.delete(`roles/${id}`)
      if (res.meta.status !== 200) return this.$message.error('删除角色失败！')
      this.$message.success('删除角色成功！')
      this.getRoleList()
    },
    async removeRightById(role, rightId) {
      console.log(rightId)
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => err)

      if (confirmResult !== 'confirm') return this.$message.info('已取消删除！')

      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败！')

      // this.getRoleList() // 避免整个列表重新渲染合上下拉列表
      role.children = res.data
    },
    async setRights(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败！')
      this.rightsList = res.data

      this.getLeafKeys(role, this.defKeyArr)

      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到 defKeyArr 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含 children,则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach((item) => {
        this.getLeafKeys(item, arr)
      })
    },
    setRightDialogClosed() {
      this.defKeyArr = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]

      const idStr = keys.join(',')
      console.log(idStr)

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })

      if (res.meta.status !== 200) return this.$message.error('分配权限失败！')

      this.$message.success('分配权限成功！')
      this.getRoleList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
