<template>
    <div>
      <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query"  clearable @clear="getGoodsList">
          <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格区 -->
    <el-table :data="goodsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name" width="500px"></el-table-column>
        <el-table-column label="商品价格（元）" prop="goods_price" width="90px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="80px"></el-table-column>
        <el-table-column label="商品数量" prop="goods_number" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="145px">
          <template slot-scope="scope">{{scope.row.add_time|dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="150px">
          <template slot-scope="scope">
              <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog(scope.row.goods_id)"></el-button>
              <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeById(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
    </el-table>
    <!-- 分页区 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[10, 30, 50, 100]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total" background>
    </el-pagination>
    </el-card>
    <el-dialog title="修改商品" :visible.sync="editDialogVisible" width="50%">
      <el-form :model="editForm" ref="editFormRef" label-width="80px">
      <el-form-item label="商品名称">
      <el-input v-model="editForm.goods_name"></el-input>
      </el-form-item>
      <el-form-item label="商品价格">
      <el-input v-model="editForm.goods_price"></el-input>
      </el-form-item>
      <el-form-item label="商品重量">
      <el-input v-model="editForm.goods_weight"></el-input>
      </el-form-item>
      <el-form-item label="商品数量">
      <el-input v-model="editForm.goods_number"></el-input>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editInfo">确 定</el-button>
      </span>
    </el-dialog>
    </div>
</template>

<script>
  export default {
    data () {
    return {
      queryInfo: {
        query:'',
        pagenum:1,
        pagesize:10
      },
    // 商品列表
      goodsList:[],
    // 总数据条数
      total:0,
      editDialogVisible:false,
      editForm:{}
      }
    },
    created () {
      // 获取所有商品数据列表
      this.getGoodsList()
    },
    methods: {
    // 查询获取所有商品
      async getGoodsList () {
        const { data: res} = await this.$http.get('goods', { params:this.queryInfo})
        if (res.meta.status !== 200) {
            return this.$message.error('获取商品数据失败！')
        }
        this.$message.success('获取商品列表成功！')
        this.goodsList = res.data.goods
        this.total = res.data.total
      },
    // 监听页面显示数量  
      handleSizeChange(newSize) {
          this.queryInfo.pagesize = newSize
          this.getGoodsList()
      },
    // 监听页码条数
      handleCurrentChange(newPage) {
          this.queryInfo.pagenum=newPage
          this.getGoodsList()
      },
    //   根据ID删除商品
      async removeById (id) {
          const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
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
      const { data: res } = await this.$http.delete('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getGoodsList()
      },
    //   根据ID查询商品
      async showEditDialog(id) {
        const { data: res } = await this.$http.get('goods/'+id) 
        if (res.meta.status !== 200 ) {
            return this.$message.error('查询商品列表失败！')
        }
        this.$message.success('查询商品列表成功！')
        this.editForm = res.data
        console.log(this.editForm)
        this.editDialogVisible = true
      },
    //   添加商品页面路由
      goAddpage () {
          this.$router.push('/goods/add')
      },
    // 编辑商品提交
    async editInfo () {
        const { data: res } = await this.$http.put('goods/'+ this.editForm.goods_id,
        {
        goods_name:this.editForm.goods_name,
        goods_price:this.editForm.goods_price,
        goods_number:this.editForm.goods_number,
        goods_weight:this.editForm.goods_weight,
        goods_cat:this.editForm.goods_cat
        })
        console.log(res)
        if (res.meta.status !== 200) {
            return this.$message.error('修改商品失败！')
        }
        this.$message.success('修改商品成功！')
        this.editDialogVisible = false
        this.getGoodsList()
    }
  }
}
</script>

<style lang="less" scoped>

</style>