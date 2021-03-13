<template>
  <div class="dashboard-container">
    <div class="subjects-container">
      <el-card shadow="never">
        <!-- 搜索的表单 -->
        <el-form :model="subjectsForm" ref="subjectsForm" class="demo-ruleForm">
          <el-form-item label="学科名称" prop="subjectName">
            <el-input
              @keyup.enter="searchForm"
              type="age"
              v-model="subjectsForm.subjectName"
              autocomplete="off"
            ></el-input>
            <el-button class="clearBtn" @click="resetForm('subjectsForm')"
              >清除</el-button
            >
            <el-button type="primary" @click="searchForm">搜索</el-button>
          </el-form-item>
          <el-button
            class="addBtn"
            type="success"
            icon="el-icon-edit"
            @click="addSubject('add')"
            >新增学科</el-button
          >
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
          header-align="center"
        >
          <el-table-column label="序号" type="index" width="50" align="center">
          </el-table-column>
          <el-table-column
            property="subjectName"
            label="学科名称"
            align="center"
          >
          </el-table-column>
          <el-table-column property="username" label="创建者" align="center">
          </el-table-column>
          <el-table-column
            property="addDate"
            label="创建日期"
            width="180"
            align="center"
          >
            <template slot-scope="scope">
              <span>{{ scope.row.addDate | parseTimeByString }}</span>
            </template>
          </el-table-column>
          <el-table-column
            property="isFrontDisplay"
            label="前台是否显示"
            width="120"
            align="center"
          >
            <template slot-scope="scope">
              <span v-if="scope.row.isFrontDisplay == 1">是</span>
              <span v-else>否</span>
            </template>
          </el-table-column>
          <el-table-column
            property="twoLevelDirectory"
            label="二级目录"
            align="center"
          >
          </el-table-column>
          <el-table-column property="tags" label="标签" align="center">
          </el-table-column>
          <el-table-column property="totals" label="题目数量" align="center">
          </el-table-column>
          <el-table-column label="操作" width="280" align="center">
            <template slot-scope="scope">
              <el-link
                type="primary"
                :underline="false"
                @click="subjectDirectory(scope.row)"
                >学科分类</el-link
              >
              <el-link
                type="primary"
                :underline="false"
                @click="subjectTag(scope.row)"
                >学科标签</el-link
              >
              <el-link
                type="primary"
                :underline="false"
                @click="addSubject(scope.row.id)"
                >修改</el-link
              >
              <el-link
                type="primary"
                :underline="false"
                @click="removeSubject(scope.row.id)"
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
            :current-page="Number(subjectsForm.page)"
            :total="Number(total)"
            :page-size="Number(subjectsForm.pagesize)"
            :page-sizes="[10, 20, 30, 50]"
            layout=" prev, pager, next, sizes, jumper"
          ></el-pagination>
        </div>
        <!-- 弹出对话框模块 -->
        <subject-dialog
          ref="editSubject"
          :dialogData="dialogData"
          :dialogTitle="dialogTitle"
          @newData="getSubjectsList"
        />
      </el-card>
    </div>
  </div>
</template>

<script>
import { list, detail, remove } from '@/api/hmmm/subjects'
// 引入弹框的组件
import subjectDialog from './../components/subjects-add'
export default {
  name: 'SubjectsContainer',
  components: {
    subjectDialog
  },
  props: {},
  data() {
    return {
      // 存放查询的数据
      subjectsForm: {
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
        subjectName: '',
        isFrontDisplay: 1
      },
      // 弹框的标题
      dialogTitle: ''
    }
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {
    this.getSubjectsList()
    const _this = this
    // 键盘enter操作
    document.onkeydown = function(e) {
      const key = window.event.keyCode
      if (key === 13) {
        _this.searchForm(this.subjectsForm)
      }
    }
  },
  methods: {
    // 搜索事件
    searchForm() {
      this.subjectsForm.page = 1
      this.getSubjectsList(this.subjectsForm)
    },
    // 清除表单
    resetForm(formName) {
      this.$refs[formName].resetFields()
      this.getSubjectsList()
    },
    // 获取学科列表数据
    async getSubjectsList() {
      this.listLoading = true
      const { data } = await list(this.subjectsForm)
      console.log(data)
      // 将数据渲染出来
      this.tableData = data.items
      console.log(this.tableData)
      // 总数据
      this.total = data.counts
      // 显示总数据的信息
      this.title = `数据一共 ${this.total} 条`
      this.listLoading = false
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`)
      this.subjectsForm.pagesize = val
      if (this.subjectsForm.page === 1) {
        this.getSubjectsList(this.subjectsForm)
      }
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`)
      this.subjectsForm.page = val
      this.getSubjectsList()
    },
    // 新增学科和修改学科
    async addSubject(id) {
      // 在显示弹框前，将上一次dialog里面的数据进行清空
      this.dialogData = {}
      // 显示弹框
      this.$refs.editSubject.dialogFormV()
      // console.log(id)
      // 如果传入的参数为'add'，则执行添加操作，否则执行修改操作
      if (id === 'add') {
        this.dialogTitle = '添加学科'
        // 默认开关是开启状态
        this.dialogData.isFrontDisplay = 1
      } else {
        this.dialogTitle = '修改学科'
        // 将id存在dialogData里面
        this.dialogData.id = id
        // 通过id获取学科详情,需要以对象的形式来进行传参
        const { data } = await detail({ id: id })
        // console.log(data)
        this.dialogData = data
      }
    },
    // 通过当前id进行删除操作
    async removeSubject(id) {
      // 通过id获取当前学科的数据
      const { data } = await detail({ id: id })
      this.$confirm(
        `此操作将永久删除该学科${data.subjectName}, 是否继续?`,
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
              this.getSubjectsList(this.subjectsForm)
              this.$message.success(`成功删除了学科${data.subjectName}!`)
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
    // 跳转至目录，通过query携带参数
    subjectDirectory(row) {
      this.$router.push({
        path: '/directorys',
        name: 'subjects-directorys',
        query: {
          id: row.id,
          name: row.subjectName
        }
      })
    },
    // 跳转至学科标签，将当前id和学科名称传过去
    subjectTag(row) {
      this.$router.push({
        path: '/tags',
        name: 'subjects-tags',
        query: {
          id: row.id,
          name: row.subjectName
        }
      })
    }
  }
}
</script>

<style scoped>
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
  margin-left: 10px;
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
/* .el-table {
  min-width: 1200px;
} */
.el-table tr {
  height: 60px;
}
.el-table .el-link {
  margin-right: 10px;
}
.el-table--scrollable-x .el-table__body-wrapper {
  overflow-x: auto;
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
