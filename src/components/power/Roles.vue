<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-button type="primary" @click="addDialogVisible = true"
        >添加角色</el-button
      >
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" style="width: 100%" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', index1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级 -->
              <el-col :span="5"
                ><el-tag
                  type=""
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ index1 + item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i
              ></el-col>
              <!-- 二级 -->
              <el-col :span="19">
                <el-row
                  :class="[index2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ index2 + item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      type="warning"
                      v-for="(item3, index3) in item2.children"
                      :key="item3.id"
                      >{{ index3 + item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
              {{ scope.row }}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRolesById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="addRoles"
        :rules="addRolesRules"
        ref="addRolesRef"
        label-width="70px"
      >
        <el-form-item label="角色名称" prop="roleName" label-width="80px">
          <el-input v-model="addRoles.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc" label-width="80px">
          <el-input v-model="addRoles.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="editRoles"
        :rules="editRolesRules"
        ref="editRolesRef"
        label-width="70px"
      >
        <el-form-item label="角色名称" prop="roleName" label-width="80px">
          <el-input v-model="editRoles.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc" label-width="80px">
          <el-input v-model="editRoles.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 权限分配对话框 -->
    <el-dialog
      title="权限分配"
      :visible.sync="RightdialogVisible"
      width="50%"
      @close="RightDialogClose"
    >
      <el-tree
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all=""
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="RightdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="AllotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      // 控制添加角色弹窗的值
      addDialogVisible: false,
      // 添加用户的表单数据
      addRoles: {
        roleName: '',
        roleDesc: ''
      },
      // 添加表单的验证规则
      addRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      },
      editRoles: {
      },
      // 控制修改角色弹窗的值
      editDialogVisible: false,
      // 修改角色表单的验证规则
      editRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      },
      // 控制权限分配对话框
      RightdialogVisible: false,
      // 所有权限数据
      rightsList: [],
      // 权限分配框-树型绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 权限分配框-选中的权限列表id
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      console.log(this.rolesList)
    },
    // 关闭添加角色弹窗时，初始化表单
    addDialogClosed () {
      this.$refs.addRolesRef.resetFields()
      console.log(this.addRoles.roleName)
    },
    // 添加角色
    addUser () {
      this.$refs.addRolesRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoles)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    // 根据id查询角色
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色失败')
      }
      this.$message.success('获取角色成功')
      this.editRoles = res.data
      this.editDialogVisible = true
    },
    // 编辑角色对话框关闭时，初始化表单
    editDialogClosed () {
      this.editDialogVisible = false
      this.$refs.editRolesRef.resetFields()
    },
    // 编辑角色，并提交
    editUser () {
      this.$refs.editRolesRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        console.log(this.editRoles.roleId)
        console.log(typeof this.editRoles.roleId)
        const { data: res } = await this.$http.put('roles/' + this.editRoles.roleId, { roleName: this.editRoles.roleName, roleDesc: this.editRoles.roleDesc })

        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户失败')
        }
        this.$message.success('修改用户成功')
        this.editDialogVisible = false
        this.getRolesList()
      })
    },
    // 删除角色
    async removeRolesById (id) {
      const confirmRulst = await this.$confirm('此操作将永久删除该角色，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmRulst !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.getRolesList()
    },
    // 删除角色权限
    async removeRightById (rulst, rightId) {
      const confirmRulst = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmRulst !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${rulst.id}/rights/${rightId}`)

      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }

      this.$message.success('删除成功')
      // this.getRolesList()
      rulst.children = res.data
    },
    // 分配角色权限
    async showRightDialog (role) {
      this.roleId = role.id
      // 获取树型结构权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      // 把获取到的权限数据保存到rightsList中
      this.rightsList = res.data
      // 获取三级权限
      this.getLeafKeys(role, this.defKeys)
      this.$message.success('获取权限列表成功')
      this.RightdialogVisible = true
    },
    // 获取对应角色的三级权限id,并保存到defKeys数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 分配权限框关闭时初始化
    RightDialogClose () {
      this.defKeys = []
    },
    // 分配权限框-确认事件
    async AllotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.RightdialogVisible = false
    }
  }
}
</script>

<style>
.el-tag {
  margin: 7px;
}
.bdbottom {
  border-bottom: 1px solid black;
}
.bdtop {
  border-top: 1px solid black;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
