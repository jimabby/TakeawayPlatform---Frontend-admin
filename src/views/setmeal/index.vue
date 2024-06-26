<template>
  <div class="dashboard-container">
    <div class="container">
      <div class="tableBar">
        <label style="margin-right: 5px;">套餐名称：</label>
        <el-input v-model="name" placeholder="请输入套餐名称：" style="width: 15%;" />

        <label style="margin-right: 5px;">套餐分类：</label>
        <el-select v-model="categoryId" placeholder="请选择">
          <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>

        <label style="margin-right: 5px;">售卖状态：</label>
        <el-select v-model="status" placeholder="请选择">
          <el-option v-for="item in statusArray" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>

        <el-button type="primary" style="margin-left: 25px;" @click="pageQuery()">搜索</el-button>

        <div style="float: right;">
          <el-button type="danger" @click="handleDelete('B')">批量删除</el-button>
          <el-button type="info" @click="() => this.$router.push('/setmeal/add')">+ 新建套餐</el-button>
        </div>
      </div>

      <el-table :data="records" stripe class="tableBox" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="25" />
        <el-table-column prop="name" label="套餐名称" />
        <el-table-column label="图片">
          <template slot-scope="scope">
            <el-image style="width:80px; height: 40px; border: none;" :src="scope.row.image"></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="categoryName" label="套餐分类" />
        <el-table-column prop="price" label="套餐价格" />
        <el-table-column label="售卖状态">
          <template slot-scope="scope">
            <div class="tableColumn-status" :class="{'stop-use': scope.row.status === 0}">
              {{ scope.row.status === 0 ? '停售' : '启售' }}
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="updateTime" label="最后操作时间" />
        <el-table-column label="操作" align="center" width="250px">
          <template slot-scope="scope">
            <el-button type="text" size="small">修改</el-button>
            <el-button type="text" size="small" @click="handleStartOrStop(scope.row)">
              {{ scope.row.status == '1' ? '停售':'启售' }}
            </el-button>
            <el-button type="text" size="small" @click="handleDelete('S',scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination class="pageList"
                     :page-sizes="[10,20,30,40]"
                     :page-size="pageSize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total"
                     @size-change="handleSizeChange"
                     @current-change="handleCurrentChange" />

    </div>
  </div>
</template>

<script lang="ts">
import { getCategoryByType } from "@/api/category";
import { getSetmealPage, enableOrDisableSetmeal, deleteSetmeal } from '@/api/setMeal'
export default {
  //模型数据
  data(){
    return {
      name: '', //套餐名称，对应上面的输入框
      page: 1, //页码
      pageSize: 10, //每页记录数
      total: 0, //总记录数
      records: [], //当前页展示的数据集合
      options: [],
      categoryId: '',
      statusArray:[
        {
          value:'0',
          label:'停售'
        },
        {
          value:'1',
          label:'启售'
        }
      ],
      status: '',//售卖状态
      multipleSelection: '' //当前表格选中的多个元素
    }
  },
  created(){
    getCategoryByType({type:2}).then(res => {
      if(res.data.code === 1){
        this.options = res.data.data
      }
    })

    //页面加载完成后查询套餐分页数据
    this.pageQuery()
  },
  methods: {
    //分页查询
    pageQuery() {
      //封装分页查询参数
      const params = {
        page: this.page,
        pageSize: this.pageSize,
        name: this.name,
        status: this.status,
        categoryId: this.categoryId
      }
      getSetmealPage(params).then(res => {
        if(res.data.code === 1){
          this.total = res.data.data.total
          this.records = res.data.data.records
        }
      })
    },
    handleSizeChange(pageSize){
      this.pageSize = pageSize
      this.pageQuery()
    },
    handleCurrentChange(page){
      this.page = page
      this.pageQuery()
    },
    //套餐启售停售操作
    handleStartOrStop(row){
      const p = {
        id: row.id,
        status: !row.status ? 1 : 0
      }
      this.$confirm('确认调整当前套餐的售卖状态，是否继续？','提示',{
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        enableOrDisableSetmeal(p).then(res => {
          if(res.data.code === 1){
            this.$message.success("套餐售卖状态修改成功！")
            this.pageQuery()
          }
        })
      })
    },
    //删除套餐
    handleDelete(type: string, id: string){
      this.$confirm('确认删除当前指定的套餐，是否继续？','提示',{
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
          let param = ''
          if(type == 'B'){ //批量删除
            const arr = new Array
            this.multipleSelection.forEach(element => {
              arr.push(element.id)
            });
            param = arr.join(',')
          }else{
            param = id
          }
          deleteSetmeal(param).then(res => {
            if(res.data.code === 1){
              this.$message.success('删除成功！')
              this.pageQuery()
            }else{
              this.$message.error(res.data.msg)
            }
          })
        })
    },
    handleSelectionChange(val){
      this.multipleSelection = val
    }
  }

}
</script>
<style lang="scss">
.el-table-column--selection .cell {
  padding-left: 10px;
}
</style>
<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;

    .container {
      background: #fff;
      position: relative;
      z-index: 1;
      padding: 30px 28px;
      border-radius: 4px;

      .tableBar {
        margin-bottom: 20px;
        .tableLab {
          float: right;
          span {
            cursor: pointer;
            display: inline-block;
            font-size: 14px;
            padding: 0 20px;
            color: $gray-2;
          }
        }
      }

      .tableBox {
        width: 100%;
        border: 1px solid $gray-5;
        border-bottom: 0;
      }

      .pageList {
        text-align: center;
        margin-top: 30px;
      }
      //查询黑色按钮样式
      .normal-btn {
        background: #333333;
        color: white;
        margin-left: 20px;
      }
    }
  }
}
</style>
