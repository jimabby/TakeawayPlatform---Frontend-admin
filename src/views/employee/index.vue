<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar">
        <label style="margin-right: 5px;">员工姓名：</label>
        <el-input v-model="name" placeholder="请输入员工姓名" style="width: 15%;" />
        <el-button type="primary" style="margin-left: 25px;" @click="pageQuery">查询</el-button>
        <el-button type="primary" style="float:right" @click="handleAddEmployee">+添加员工</el-button>
      </div>

      <el-table :data="records" stripe style="width: 100%;">
        <el-table-column prop="name" label="员工姓名"></el-table-column>
        <el-table-column prop="username" label="账号"></el-table-column>
        <el-table-column prop="phone" label="手机号"></el-table-column>
        <el-table-column prop="status" label="帐号状态">
          <template slot-scope="scope">
            {{ scope.row.status === 0 ? '禁用' : '启用' }}
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="最后操作时间"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="text" @click="handleUpdateEmployee(scope.row)">修改</el-button>
            <el-button type="text" @click="handleStartOrStop(scope.row)">{{scope.row.status === 1 ? '禁用':'启用'}}</el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        class = "pageList"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="page"
        :page-sizes="[10, 20, 30, 40, 50]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </div>
  </div>
</template>
<script lang="ts">

import { getEmployeeList, enableOrDisableEmployee } from '@/api/employee';
export default  {
  //模型数据
  data(){
    return {
      name: '', //员工姓名，对应上面的输入框
      page: 1, //页码
      pageSize: 10, //每页记录数
      total: 0, //总记录数
      records: [] //当前页展示的数据集合
    }
  },
  created() {
    this.pageQuery()
  },
  methods: {
    //分页查询
    pageQuery(){
      //准备请求参数
      const params = {name:this.name, page:this.page, pageSize:this.pageSize}
      //发送ajax请求，访问后端服务，获取分页数据
      getEmployeeList(params).then(res => {
        if(res.data.code === 1){
          this.total = res.data.data.total
          this.records = res.data.data.records
        }
      }).catch(err => {
        this.$message.error('请求出错:' + err.message)
      })
    },
    //pageSize发生变化时触发
    handleSizeChange(pageSize){
      this.pageSize = pageSize
      this.pageQuery()
    },
    //page发生变化时触发
    handleCurrentChange(page){
      this.page = page
      this.pageQuery()
    },
    //启用禁用员工账号
    handleStartOrStop(row){

      if(row.username === 'admin'){
        this.$message.error('无法更改管理员账号状态！')
        return
      }

      //弹出确认提示框
      this.$confirm('确认要修改当前员工账号状态吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        }).then(() => {
          const p = {
            id: row.id,
            status: !row.status ? 1 : 0
          }

          enableOrDisableEmployee(p).then(res =>{
            if (res.data.code === 1){
              this.$message.success('员工账号状态修改成功！')
              this.pageQuery()
            }
          })
        })
    },
    //跳转到新增员工页面（组件）
    handleAddEmployee(){
      //路由跳转
      this.$router.push('/employee/add')
    },
    //跳转到修改员工页面（组件）
    handleUpdateEmployee(row){
      if(row.username === 'admin'){
        this.$message.error('系统管理员账号不能修改！')
        return
      }
      this.$router.push({
        path:'/employee/add',
        query: {id: row.id}
      })
    }
  }

}
</script>

<style lang="scss" scoped>
.disabled-text {
  color: #bac0cd !important;
}
</style>
