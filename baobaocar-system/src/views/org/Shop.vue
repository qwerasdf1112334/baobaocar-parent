<template>
  <section>
    <!--高级搜索栏-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="queryData">
        <el-form-item>
          <el-input v-model="queryData.keyword" placeholder="店铺名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" v-on:click="getShops">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleAdd">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!--
      users : tableData
      @selection-change="selsChange": 表格的多选列
      sortable:  排序





    -->
    <el-table
        :data="pageInfo.data"
        highlight-current-row
        v-loading="listLoading"
        @selection-change="selsChange"
        style="width: 100%;">
      <el-table-column type="selection" width="55">
      </el-table-column>
      <el-table-column type="index" width="60">
      </el-table-column>
      <el-table-column prop="name" label="店铺名" width="120" sortable>
      </el-table-column>

      <el-table-column prop="tel" label="电话" width="180" sortable>
      </el-table-column>
      <el-table-column prop="registerTime" label="注册时间" width="180" sortable>
      </el-table-column>
      <el-table-column prop="address" label="地址" width="180" sortable>
      </el-table-column>

      <el-table-column prop="logo" label="logo" width="180" sortable>
      </el-table-column>

      <el-table-column prop="state" label="状态" width="180" sortable>
        <template slot-scope="scope">
          <span v-if="scope.row.state === 1" class="status-pending">待审核</span>
          <span v-else-if="scope.row.state === 2" class="status-approved">审核通过</span>
          <span v-else-if="scope.row.state === 3" class="status-success">激活成功</span>
          <span v-else class="status-rejected">驳回</span>
        </template>
      </el-table-column>





      <el-table-column label="操作" width="230">
        <template scope="scope">
          <el-button  type="warning" size="small" @click="handleexamine(scope.$index, scope.row)">审核</el-button>
          <el-button type="primary" size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!--分页条和删除全部-->
    <el-col :span="24" class="toolbar">
      <el-button type="danger" @click="batchRemove" :disabled="this.sels.length===0">批量删除</el-button>
      <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="queryData.pageSize" :total="pageInfo.total" style="float:right;">
      </el-pagination>
    </el-col>


    <!--新增/编辑界面-->
    <!--
      对话框的显式  也是一个 布尔值控制的 但是不能使用 v-model 进行绑定
      :visible.sync 这个属性是控制对话框显式的
      id
name
tel
registerTime
state
address
logo
    -->
    <el-dialog title="新增/修改" :visible.sync="saveFormVisible" :close-on-click-modal="false">
      <el-form :model="saveForm" label-width="80px" :rules="saveFormRules" ref="saveForm">
        <el-form-item label="店铺名称" prop="name">
          <el-input v-model="saveForm.name" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="电话" prop="tel">
          <el-input v-model="saveForm.tel" auto-complete="off"></el-input>
        </el-form-item>

        <el-form-item label="注册时间" prop="registerTime">
          <el-input v-model="saveForm.registerTime" auto-complete="off":readonly="true"></el-input>
        </el-form-item>

            <el-form-item label="address" prop="地址">
              <el-input v-model="saveForm.address" auto-complete="off"></el-input>
            </el-form-item>

            <el-form-item label="logo" prop="logo">
              <el-input v-model="saveForm.logo" auto-complete="off"></el-input>
            </el-form-item>
            <!--下拉选项
              :key=""  //唯一标识
              :label 选择之后展示到选择框中的值
              :value 选中之后绑定给模型层的值  如果要绑定对象给模型层 有一个大坑
                 必须要写  value-key="id"
            -->



<!--        <el-form-item label="状态" prop="state">-->
<!--          <el-input v-model="saveForm.state" auto-complete="off"></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="状态" prop="state">-->
<!--          <el-radio-group v-model="saveForm.state">-->
<!--            <el-radio class="radio" :label="2">启用</el-radio>-->
<!--            <el-radio class="radio" :label="1">禁用</el-radio>-->
<!--          </el-radio-group>-->
<!--        </el-form-item>-->
<!--      </el-form>-->

      <el-form-item label="状态" prop="state">
        <el-select v-model="saveForm.state">
          <el-option label="待审核"  value="1"></el-option>
          <el-option label="审核通过待激活" value="2"></el-option>

          <el-option label="激活成功" value="3"></el-option>
          <el-option label="驳回" value="4"></el-option>
        </el-select>
      </el-form-item>
      </el-form>


      <!--
      <el-form-item label="性别">
					<el-radio-group v-model="addForm.sex">
						<el-radio class="radio" :label="1">男</el-radio>
						<el-radio class="radio" :label="0">女</el-radio>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="年龄">
					<el-input-number v-model="addForm.age" :min="0" :max="200"></el-input-number>
				</el-form-item>
				<el-form-item label="生日">
					<el-date-picker type="date" placeholder="选择日期" v-model="addForm.birth"></el-date-picker>
				</el-form-item>
				<el-form-item label="地址">
					<el-input type="textarea" v-model="addForm.addr"></el-input>
				</el-form-item>
      -->
      <div slot="footer" class="dialog-footer">
        <el-button @click.native="saveFormVisible = false">取消</el-button>
        <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
      </div>
    </el-dialog>

<!--审核弹窗-->
    <el-dialog title="审核" :visible.sync="shenFormVisible" :close-on-click-modal="false">
      <el-form :model="saveForm" label-width="80px" :rules="saveFormRules" ref="saveForm">
        <el-form-item label="店铺名称" prop="name">
          <el-input v-model="saveForm.name" auto-complete="off" :readonly="true"></el-input>
        </el-form-item>

        <el-form-item label="电话" prop="tel">
          <el-input v-model="saveForm.tel" auto-complete="off" :readonly="true"></el-input>
        </el-form-item>

        <el-form-item label="注册时间" prop="registerTime">
          <el-input v-model="saveForm.registerTime" auto-complete="off" :readonly="true"></el-input>
        </el-form-item>

        <el-form-item label="address" prop="地址">
          <el-input v-model="saveForm.address" auto-complete="off" :readonly="true"></el-input>
        </el-form-item>

        <el-form-item label="logo" prop="logo">
          <el-input v-model="saveForm.logo" auto-complete="off" :readonly="true"></el-input>
        </el-form-item>

        <el-form-item label="状态" prop="state">
          <el-select v-model="saveForm.state">
            <el-option label="待审核"  value="1"></el-option>
            <el-option label="审核通过待激活" value="2"></el-option>

            <el-option label="激活成功" value="3"></el-option>
            <el-option label="驳回" value="4"></el-option>
          </el-select>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button type="success" @click.native="yesSubmit" :loading="addLoading">通过</el-button>
        <el-button type="danger" @click.native="noSubmit" :loading="addLoading">驳回</el-button>
        <el-button @click.native="shenFormVisible = false">取消</el-button>
      </div>
    </el-dialog>
  </section>
</template>

<script>

export default {
  data() {
    return {
      queryData: {
        keyword: '',
        pageSize: 10,
        currentPage:1
      },
      pageInfo:{
        data: [],
        total: 0,
      },
      page: 1,
      listLoading: false,
      sels: [],//列表选中列

      saveFormVisible:false,

      // id
      // name
      // tel
      // registerTime
      // state
      // address
      // logo
      shenFormVisible:false,

      saveForm:{
        id:null,
        name:"",
        tel:"",
        registerTime:"",
        state:'待审核',
        address:"",
        logo:""
      },
      saveFormRules: {
        name: [
          { required: true, message: '请输入店铺名称', trigger: 'blur' }
        ]
      },
      addFormVisible: false,//新增界面是否显示
      addLoading: false,




    }
  },
  methods: {



    handleCurrentChange(val) {
      this.queryData.currentPage = val;
      this.getShops();
    },
    //获取用户列表
    getShops() {
      // 获取参数 - currentPage pageSize keyword
      // 发起请求
      this.$http.post("/shop",this.queryData)
          .then(res =>{
            res=res.data
            console.log(res+"99999999");
            if (res.success){

              this.pageInfo = res.data
            }else{
              this.$message.error("网络异常请联系管理员");
            }
          })
          .catch(res =>{
          })
    },
    //删除
    handleDel: function (index, row) {
      this.$confirm('确认删除该记录吗?', '提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true; // 加载框
        // 发送请求
        this.deletebyId(row.id)
      }).catch(() => {

      });
    },
    // 删除单条方法
    deletebyId(id){
      // 发起请求
      this.$http.delete("/shop/"+id)
          .then(res =>{
            res = res.data
            if (res.success){
              this.$message({ message: '删除成功', type: 'success' });
              this.listLoading = false; // 加载框
              this.queryData.currentPage = 1
              this.getShops()
            }else{
              this.$message.error("网络异常请联系管理员");
            }
          })
          .catch(res =>{
          })
    },
    //显示编辑界面
    handleEdit: function (index, row) {
      this.saveFormVisible = true;

      var assign = Object.assign({}, row);
      this.saveForm = assign;



    },
    //显示新增界面
    handleAdd: function () {
      this.saveFormVisible = true;

      this.saveForm = {
        id:null,
        name:"",
        intro:"",
        manager:{
          id:null,
          username:""
        },
        parent:{
          id:null,
          name:""
        },
        state:""
      };

    },
    //审核
    handleexamine(index, row){
      this.shenFormVisible = true;

      var assign = Object.assign({}, row);
      this.saveForm = assign;
    },
    //编辑
    editSubmit: function () {
      this.$refs.editForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.editLoading = true;
            //NProgress.start();
            let para = Object.assign({}, this.editForm);
            para.birth = (!para.birth || para.birth == '') ? '' : util.formatDate.format(new Date(para.birth), 'yyyy-MM-dd');
            editUser(para).then((res) => {
              this.editLoading = false;
              //NProgress.done();
              this.$message({
                message: '提交成功',
                type: 'success'
              });
              this.$refs['editForm'].resetFields();
              this.saveFormVisible = false;
              this.getShops();
            });
          });
        }
      });
    },
    //新增
    addSubmit: function () {
      this.$refs.saveForm.validate((valid) => {
        if (valid) {
          this.$confirm('确认提交吗？', '提示', {}).then(() => {
            this.addLoading = true;
            // 发送 修改请求
            this.saveOrUpdate()
          });
        }
      });
    },
    // 新增或者是修改的方法
    saveOrUpdate(){
      var parent = this.saveForm.parent;
      if (parent){
        this.saveForm.parent = {
          id: parent[parent.length-1]
        }
      }else{
        this.saveForm.parent = {
          id: null
        }
      }

      this.$http.put("/shop",this.saveForm)
          .then(res =>{
            res = res.data
            if (res.success){
              this.$message({ message: '保存成功', type: 'success' });
              this.queryData.currentPage = 1
              this.getShops()
              this.saveFormVisible = false;
              this.addLoading = false
            }else{
              this.$message.error("网络异常请联系管理员");
            }
          })
          .catch(res =>{

          })
    },
    selsChange: function (sels) {
      var ids = sels.map(x =>  x.id)
      this.sels = ids;
    },
    //批量删除
    batchRemove: function () {
      this.$confirm('确认删除选中记录吗？', '提示', {
        type: 'warning'
      }).then(() => {
        this.listLoading = true;
        //NProgress.start();

        // 批量删除的方法
        this.batchDelete()
      }).catch(() => {

      });
    },
    batchDelete(){
      // 发起请求
      this.$http.patch("/shop",this.sels)
          .then(res =>{
            res = res.data
            if (res.success){
              this.$message({ message: '删除成功', type: 'success' });
              this.listLoading = false; // 加载框
              this.queryData.currentPage = 1
              this.getShops()
            }else{
              this.$message.error("网络异常请联系管理员");
            }
          })
          .catch(res =>{
          })
    }
  },

  mounted() {
    this.getShops();
  }
}

</script>

<style scoped>
/* 样式可以根据需要自行定义 */
.status-pending {
  color: orange; /* 待审核状态的颜色 */
}

.status-approved {
  color: green; /* 审核通过状态的颜色 */
}

.status-inactive {
  color: blue; /* 待激活状态的颜色 */
}

.status-success {
  color: lime; /* 激活成功状态的颜色 */
}

.status-rejected {
  color: red; /* 驳回状态的颜色 */
}


</style>