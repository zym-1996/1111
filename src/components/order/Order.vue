<template>
    <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>订单管理</el-breadcrumb-item>
    <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索框 -->
        <el-row>
            <el-col :span="8">
              <el-input placeholder="请输入内容">
              <el-button slot="append" icon="el-icon-search"></el-button>
              </el-input>
            </el-col>
        </el-row>
        <!-- 订单列表 -->
        <el-table :data="OredrList" border stripe>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="订单编号" prop="order_number" width="200px"></el-table-column>
            <el-table-column label="订单价格" prop="order_price"></el-table-column>
            <el-table-column label="是否付款" prop="pay_status">
                <template slot-scope="scope">
                    <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
                    <el-tag type="danger" v-else>未付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="是否发货" prop="is_send"></el-table-column>
            <el-table-column label="下单时间" prop="create_time">
                <template slot-scope="scope">{{scope.row.create_time| dateFormat}}</template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="showBox(scope.row.goods_id)"></el-button>
                    <el-button type="success" size="mini" icon="el-icon-location" @click="showProgressBox(scope.row.goods_id)"></el-button>
                </template>
            </el-table-column>
        </el-table>
        <!-- 分页区 -->
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[10, 30, 50, 100]" 
          :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
        </el-pagination>
        <!-- 修改地址对话框 -->
        <el-dialog title="修改地址" :visible.sync="addressVisible" width="50%" @close="addressDiaLogClosed">
          <el-form :model="addressForm" :rules="addressFromRules" ref="addressFromRef" label-width="100px">
          <el-form-item label="省市区/县" prop="address1">
          <el-cascader :options="cityData" v-model="addressForm.address1" :props="{ expandTrigger: 'hover' }"></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="addressVisible = false">取 消</el-button>
            <el-button type="primary" @click="addressVisible = false">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 物流信息对话框 -->
        <el-dialog
        title="物流进度"
        :visible.sync="progressVisible"
        width="50%">
        <span>没有数据</span>
        </el-dialog>
      </el-card>
    </div>
</template>

<script>
import cityData from './citydata'
    export default {
      data () {
          return {
            queryInfo:{
                query:'',
                pagenum:1,
                pagesize:10
            },
            total:0,
            OredrList:[],
            addressVisible:false,
            addressForm:{
              address1:[],
              address2:''
            },
            addressFromRules:{
              address1:[
                { required: true, message: '请选择省市区/县', trigger: 'blur' }
              ],
              address2:[
                { required: true, message: '请填写详细地址', trigger: 'blur' }
              ]
            },
            cityData,
            progressVisible:false,
            
          }
      },
      created () {
          this.getOredrList()
      },
      methods:{
         async getOredrList() {
            const { data: res } = await  this.$http.get('orders',{params:this.queryInfo})
            if (res.meta.status !== 200 ) {
                return this.$message.error('获取数据失败！')
            }
            console.log(res)
            this.total = res.data.total
            this.OredrList = res.data.goods
          },
        // 监听页面显示数量  
        handleSizeChange(newSize) {
          this.queryInfo.pagesize = newSize
          this.getOredrList()
        },
        // 监听页码条数
        handleCurrentChange(newPage) {
          this.queryInfo.pagenum=newPage
          this.getOredrList()
        },
        showBox () {
          this.addressVisible = true
        },
        addressDiaLogClosed () {
          this.$refs.addressFromRef.resetFields()
        },
        showProgressBox () {
          this.progressVisible = true
        }
      }
    }
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>