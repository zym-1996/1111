<template>
    <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>权限管理</el-breadcrumb-item>
    <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="addDialogVisible=true">添加角色</el-button>
            </el-col>
        </el-row>
    <!-- 角色列表区 -->
    <el-table :data="rolelist" border stripe>
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
          <!-- 渲染一级权限 -->
          <el-col :span="5"><el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
          <i class="el-icon-caret-right"></i>
          </el-col>
          <!-- 渲染二级和三级权限 -->
          <el-col :span="19">
            <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
              <el-col :span="6">
                <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="18">
                <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
              </el-col>
            </el-row>
          </el-col>
          </el-row>
        </template>
      </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作" width="300px">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
          <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
    </el-card>
    <el-dialog
    title="分配角色"
    :visible.sync="setRightDialogVisible"
    width="50%" @close="setRightDialogClosed">
    <!-- 树形控件 -->
    <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defkeys" ref="treeRef"></el-tree>
    <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
    </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog
    title="添加角色"
    :visible.sync="addDialogVisible"
    width="50%" @close="addDialogClosed">
    <!-- 内容主体 -->
    <el-form :model="addForm" ref="addFormRef" label-width="80px" :rules="addFormRules">
    <el-form-item label="角色名称"  prop="roleName">
    <el-input v-model="addForm.roleName"></el-input>
    </el-form-item>
    <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="addForm.roleDesc"></el-input>
    </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
    </span>
    </el-dialog>
    <!-- 修改用户的对话框 -->
    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
      <el-form-item label="角色名称" prop="roleName">
      <el-input v-model="editForm.roleName"></el-input>
      </el-form-item>
      <el-form-item label="角色描述" prop="roleDesc">
      <el-input v-model="editForm.roleDesc"></el-input>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
    // 所有角色列表
      rolelist: [],
      setRightDialogVisible: false,
      // 获取的所有数据
      rightslist: [],
      // 树形控件的绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //  默认选中的节点ID值数组
      defkeys: [],
      // 当前即将分配权限的ID
      roleId: [],
      // 控制添加角色的显示与隐藏
      addDialogVisible:false,
      // 添加用户的数据
      addForm:{
        roleName:'',
        roleDesc:''
      },
      // 添加用户的验证规则
      addFormRules : {
        roleName:[{ required: true, message: '请输入角色名称', trigger: 'blur' }],
        roleDesc:[{ required: true, message: '请输入角色描述', trigger: 'blur' }]
      },
      // 查询到的角色信息
      editForm: {},
      // 编辑角色对话框的显示与隐藏
      editDialogVisible: false,
      // 修改角色的验证规则
      editFormRules: {
        roleName:[{ required: true, message: '请输入角色名称', trigger: 'blur' }],
        roleDesc:[{ required: true, message: '请输入角色描述', trigger: 'blur' }]
      }
    }
    
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolelist = res.data
    },
    // 根据对应ID删除权限
    async removeRightById (role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除！')
      }

      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      role.children = res.data
    },
    // 控制分配角色的显示与隐藏
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败！')
      }
      // 把获取到的数据赋值给rightslist
      this.rightslist = res.data
      // 递归获取的3级节点ID
      this.getLeafKeys(role, this.defkeys)
      this.setRightDialogVisible = true
    },
    //  通过递归的形式，将获取到的角色下的三级权限ID，保存到数组中
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item =>
        this.getLeafKeys(item, arr)
      )
    },
    // 监听分配权限的对话框事件
    setRightDialogClosed () {
      this.defkeys = []
    },
    // 分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 监听添加角色对话框重置为空
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 添加用户请求
    addUser () {
      this.$refs.addFormRef.validate(async valid=> {
        if (!valid) return
        const { data: res} = await this.$http.post('roles',this.addForm)
        if (res.meta.status !==201 ) {
          this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')
        this.addDialogVisible = false
        this.getRolesList()
      })
    },
    // 监听编辑角色对话框
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 显示与查询角色信息
    async showEditDialog (id) {
      const { data:res } = await this.$http.get('roles/' + id)
      if (res.meta.status !==200 ) {
        return this.$message.error('查询角色信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
      console.log(this.editForm);
    },
    // 修改角色信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid =>{
        if (!valid) return
        // 发起修改请求
        const { data: res } = await this.$http.put(`roles/${this.editForm.roleId}`,{
          roleName:this.editForm.roleName,roleDesc:this.editForm.roleDesc
        } )
        if (res.meta.status !==200 ) {
          return this.$message.error('更新角色信息失败！')
        }
        this.editDialogVisible = false
        this.getRolesList()
        this.$message.success('更新角色信息成功！')
      })
    },
    // 根据ID删除角色
    async removeUserById (id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
       if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getRolesList()
    }
  }
}
</script>

<style lang="less" scoped>
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
