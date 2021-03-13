<template>
  <div class="dashboard-container">
    <div class="directorys-container">
      <el-card shadow="never">
        <!-- 面包屑 -->
        <el-breadcrumb
          separator-class="el-icon-arrow-right"
          v-if="$route.query.id"
        >
          <el-breadcrumb-item>学科目录</el-breadcrumb-item>
          <el-breadcrumb-item>{{ $route.query.name }}</el-breadcrumb-item>
          <el-breadcrumb-item>目录管理</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 搜索的表单 -->
        <el-form :model="directorysForm" ref="directorysForm" class="ruleForm">
          <el-form-item label="目录名称" prop="directoryName">
            <el-input
              @keyup.enter="searchForm"
              v-model="directorysForm.directoryName"
              autocomplete="off"
            ></el-input>
          </el-form-item>
          <el-form-item label="状态" prop="state">
            <el-select v-model="directorysForm.state" placeholder="请选择状态">
              <el-option label="启用" value="1"></el-option>
              <el-option label="禁用" value="0 "></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button class="clearBtn" @click="resetForm('directorysForm')"
              >清除</el-button
            >
            <el-button class="clearBtn" type="primary" @click="searchForm"
              >搜索</el-button
            >
          </el-form-item>
          <div>
          <el-button
            class="clearBtn"
            type="text"
            icon="el-icon-back"
            v-if="$route.query.id"
            @click="returnSubject"
            >返回学科</el-button
          >
          <el-button
            class="addBtn"
            type="success"
            icon="el-icon-edit"
            @click="addDirectory('add')"
            >新增目录</el-button
          >
          </div>
        </el-form>
        <!-- 显示总数据 -->
        <el-alert :title="title" type="info" show-icon :closable="false">
        </el-alert>
        <!-- 展示的数据 -->
        <el-table
          ref="singleTable"
          :data="tableData"
          v-loading="listLoading"
          element-loading-text="给我一点时间"
          highlight-current-row
          style="width: 100%"
        >
          <el-table-column label="序号" type="index" width="50">
          </el-table-column>
          <el-table-column property="subjectName" label="所属学科">
          </el-table-column>
          <el-table-column property="directoryName" label="目录名称">
          </el-table-column>
          <el-table-column property="username" label="创建者">
          </el-table-column>
          <el-table-column property="addDate" label="创建日期" width="180">
            <template slot-scope="scope">
              <span>{{ scope.row.addDate | parseTimeByString }}</span>
            </template>
          </el-table-column>
          <el-table-column property="totals" label="面试题数量">
          </el-table-column>
          <el-table-column align="center" label="状态">
            <template slot-scope="scope">
              <span v-if="scope.row.state == 1">已开启</span>
              <span v-else>已禁用</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template slot-scope="scope">
              <el-link
                type="primary"
                :underline="false"
                v-if="scope.row.state == 0"
                @click="changeStateById(scope.row.id, scope.row.state)"
                >启用</el-link
              >
              <el-link
                type="primary"
                :underline="false"
                v-else
                @click="changeStateById(scope.row.id, scope.row.state)"
                >禁用</el-link
              >
              <el-link
                :type="scope.row.state == 0 ? 'primary' : 'info'"
                :underline="false"
                :disabled="scope.row.state == 1"
                @click="addDirectory(scope.row.id)"
                >修改</el-link
              >
              <el-link
                :type="scope.row.state == 0 ? 'primary' : 'info'"
                :underline="false"
                :disabled="scope.row.state == 1"
                @click="removeDirectory(scope.row.id)"
                >删除</el-link
              >
            </template>
          </el-table-column>
        </el-table>
        <!-- 显示页码 -->
        <div class="pages">
          <el-pagination
            background
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="Number(directorysForm.page)"
            :total="Number(total)"
            :page-size="Number(directorysForm.pagesize)"
            :page-sizes="[10, 20, 30, 50]"
            layout=" prev, pager, next, sizes, jumper"
          ></el-pagination>
        </div>
        <!-- 弹出对话框模块 -->
        <directory-dialog
          ref="editDirectory"
          :dialogData="dialogData"
          :dialogTitle="dialogTitle"
          :subjectNameList="subjectNameList"
          @newData="getDirectorysList"
        />
      </el-card>
    </div>
  </div>
</template>

<script>
import { list, detail, remove, changeState } from '@/api/hmmm/directorys'
import { list as subjectList } from '@/api/hmmm/subjects'
// 引入弹框的组件
import directoryDialog from './../components/directorys-add'
export default {
  name: 'DirectorysContainer',
  components: {
    directoryDialog
  },
  props: {},
  data() {
    return {
      // 存放查询的数据
      directorysForm: {
        page: 1,
        pagesize: 10
      },
      // 表格数据
      tableData: [],
      // 总数据
      total: '',
      // 显示总数据的信息
      title: '',
      // 加载的状态
      listLoading: false,
      // 弹框中的数据
      dialogData: {
        subjectID: 0,
        directoryName: '',
        subjectName: ''
      },
      // 弹框的标题
      dialogTitle: '',
      // 获取所有学科名称
      allSubject: {
        page: 1,
        pagesize: 100
      },
      subjectNameList: []
    }
  },
  computed: {},
  watch: {},
  created() {
    this.getSubjectsName()
    this.searchDirectory()
  },
  mounted() {
    this.getDirectorysList()
    const _this = this
    // 键盘enter操作
    document.onkeydown = function(e) {
      const key = window.event.keyCode
      if (key === 13) {
        _this.searchForm(this.directorysForm)
      }
    }
  },
  methods: {
    // 搜索事件
    searchForm() {
      this.directorysForm.page = 1
      this.getDirectorysList(this.directorysForm)
    },
    // 清除表单
    resetForm(formName) {
      this.$refs[formName].resetFields()
    },
    // 获取目录列表数据
    async getDirectorysList() {
      // 1、加载状态
      this.listLoading = true
      // 2、调用接口，获取数据
      const { data } = await list(this.directorysForm)
      // 3、将数据渲染出来
      this.tableData = data.items
      console.log(this.tableData, 111)
      ///4、总数据
      this.total = data.counts
      // 显示总数据的信息
      this.title = `数据一共 ${this.total} 条`
      this.listLoading = false
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`)
      this.directorysForm.pagesize = val
      if (this.directorysForm.page === 1) {
        this.getDirectorysList(this.directorysForm)
      }
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`)
      this.directorysForm.page = val
      this.getDirectorysList()
    },
    // 新增目录和修改目录
    async addDirectory(id) {
      // 在显示弹框前，将上一次dialog里面的数据进行清空
      this.dialogData = {}
      // 显示弹框
      this.$refs.editDirectory.dialogFormV()
      // console.log(id)
      // 如果传入的参数为'add'，则执行添加操作，否则执行修改操作
      if (id === 'add') {
        this.dialogTitle = '添加目录'
        // 默认开关是开启状态
        // this.dialogData.isFrontDisplay = 1
        this.getSubjectsName()
      } else {
        this.dialogTitle = '修改目录'
        // 将id存在dialogData里面
        this.dialogData.id = id
        // 通过id获取目录详情,需要以对象的形式来进行传参
        const { data } = await detail({ id: id })
        // console.log(data)
        this.dialogData = data
        console.log(this.dialogData)
      }
    },
    // 通过当前id进行删除操作
    async removeDirectory(id) {
      // 通过id获取当前目录的数据
      const { data } = await detail({ id: id })
      this.$confirm(
        `此操作将永久删除该目录${data.directoryName}, 是否继续?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(async () => {
          // 执行删除的操作
          await remove({ id: id })
            .then(response => {
              // console.log(response)
              this.tableData.splice(id, 1)
              this.getDirectorysList(this.directorysForm)
              this.$message.success(`成功删除了目录${data.directoryName}!`)
            })
            .catch(response => {
              console.log(response)
              this.$message.error('删除失败!')
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    // 获取所有学科名称
    async getSubjectsName() {
      const { data } = await subjectList(this.allSubject)
      const subjectArr = data.items
      // 给数组传入之前，要将数据进行清空，不然每次调用都会增加
      this.subjectNameList = []
      subjectArr.forEach((item, index) => {
        this.subjectNameList.push({
          subjectId: item.id,
          subjectName: item.subjectName
        })
      })
      console.log(this.subjectNameList)
    },
    // 修改状态
    async changeStateById(id, state) {
      // 首先判断state是0 还是1
      if (state === 0) {
        state = 1
      } else {
        state = 0
      }
      this.dialogData.id = id
      this.dialogData.state = state
      // 调用接口进行修改
      await changeState(this.dialogData)
      this.getDirectorysList()
    },
    // 返回学科
    returnSubject() {
      this.$router.push({ path: '/list', name: 'subjects-list' })
    },
    // 通过学科传过来的参数进行查询
    searchDirectory() {
      if (this.$route.query.id) {
        this.directorysForm.page = 1
        this.directorysForm.subjectID = this.$route.query.id
        this.getDirectorysList(this.directorysForm)
      }
    }
  }
}
</script>

<style scoped>
/* 面包屑样式 */
.el-breadcrumb {
  border-bottom: 1px solid #ccc;
  padding-bottom: 10px;
  margin-bottom: 20px;
}
/* 搜索表单中的样式 */
.el-card {
  margin: 15px;
}
.el-form {
  display: flex;
  justify-content: space-between;
}
.el-form-item {
  display: flex;
  justify-content: start;
}
.el-input {
  width: 200px;
}
.clearBtn {
  height: 36px;
  /* margin-left: 10px; */
}
.addBtn {
  margin-left: 10px;
  width: 120px;
  height: 36px;
}
/* 搜索表单中的样式 */
/* 总数据中的样式 */
.el-alert {
  margin: 0;
  margin-bottom: 15px;
  font-size: 16px;
}
/* 总数据中的样式 */
/* 表格中的样式 */
.el-table tr {
  height: 60px;
}
.el-table .el-link {
  margin-right: 10px;
}
.el-table--scrollable-x .el-table__body-wrapper {
  overflow-x: auto;
}
.el-table--medium td,
.el-table--medium th {
  padding: 20px 0;
}
/* 表格中的样式 */
/* 页码中的样式 */
.pages {
  margin-top: 20px;
  text-align: right;
}
.el-pagination .el-select .el-input .el-input__inner {
  width: 100px;
  padding-right: 0;
}
.el-pagination__sizes {
  position: relative;
}
.el-input__suffix {
  position: absolute;
  right: 0;
}
.pages .el-pagination .el-pagination__jump {
  margin-left: 0;
}
/* 页码中的样式 */
</style>
