<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        <el-form ref="ClearForm" :model="searchForm" label-width="80px">
          <el-form-item label="关键字" prop="keyword">
            <el-input
              v-model="searchForm.keyword"
              placeholder="根据编号搜索"
              size="small"
            ></el-input>
            <el-button style="margin-left:682px" size="small" @click="onClear"
              >清除</el-button
            >
            <el-button type="primary" size="small" @click="onSearch"
              >搜索</el-button
            >
          </el-form-item>
        </el-form>
        <el-alert
          :title="`数据一共${total}条`"
          type="info"
          show-icon
          :closable="false"
        >
        </el-alert>
        <!-- 表格 -->
        <div style="margin-top: 10px">
          <el-table
            :data="PleaseList"
            stripe
            style="width: 100%"
            header-align="center"
          >
            <el-table-column prop="id" label="编号" width="220" align="center">
            </el-table-column>
            <el-table-column prop="questionType" label="题型" align="center">
              <template slot-scope="scope">
                <div>
                  {{
                    scope.row.questionType === '1'
                      ? '简单'
                      : scope.row.questionType === '2'
                      ? '一般'
                      : '困难'
                  }}
                </div>
              </template>
            </el-table-column>
            <el-table-column prop="questionIDs" label="题目编号" width="220" align="center">
              <template slot-scope="scope">
                <!-- vue实例外创建 -->
                <div
                  v-for="(item, index) in scope.row.questionIDs"
                  :key="index"
                  style="color: #0099FF; cursor: pointer"
                  @click="viewQuestions(item.id)"
                >
                  {{ item.number }}
                </div>
              </template>
              >
            </el-table-column>
            <el-table-column prop="addTime" label="录入时间" width="180" align="center">
            </el-table-column>
            <el-table-column prop="totalSeconds" label="答题时间(s)" align="center">
            </el-table-column>
            <el-table-column prop="accuracyRate" label="正确率(%)" align="center">
            </el-table-column>
            <el-table-column prop="userName" label="录入人" align="center"> </el-table-column>
            <el-table-column prop="name" label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  circle
                  plain
                  @click="onDelete(scope.row.id)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
        </div>
        <!-- /表格 -->
        <!-- 分页 -->
        <page-tool
          @pageChange="handel1"
          :total="total"
          :paginationPage="searchForm.page"
          :paginationPagesize="searchForm.pagesize"
          @pageSizeChange="handel2"
        />
      </el-card>
    </div>
    <!-- 预览弹出层 -->
    <el-dialog
      title="题目预览"
      :visible.sync="dialogVisible"
      width="70%"
      v-if="dialogVisible"
    >
      <viewQuestion :viewItemId="numberID" @close="dialogVisible = false" />
    </el-dialog>
    <!--/预览弹出层 -->
  </div>
</template>

<script>
import {
  randoms as randomsQuestions,
  removeRandoms
} from '@/api/hmmm/questions'
import viewQuestion from '../components/questions-preview'
import PageTool from '../../module-dashboard/components/pageTool.vue'
export default {
  name: 'QuestionsRandoms',
  components: { viewQuestion, PageTool },
  data() {
    return {
      PleaseList: [], // 获取的数据列表
      total: null, // 数据总条数
      questionsList: null, // 试题编号列表
      searchForm: {
        keyword: null,
        page: 1,
        pagesize: 10
      },
      dialogVisible: false,
      numberID: null
    }
  },
  created() {
    this.getRandomsQuestions()
  },
  methods: {
    // 获取试题列表的方法
    async getRandomsQuestions() {
      try {
        const { data } = await randomsQuestions(this.searchForm)
        console.log(data)
        this.PleaseList = data.items
        // console.log();
        this.total = data.counts
      } catch (err) {
        this.$message('获取组题列表失败')
      }
    },
    // 搜索的方法
    async onSearch() {
      try {
        await randomsQuestions(this.searchForm.keyword)
        this.getRandomsQuestions()
      } catch (err) {
        this.$message.fail('获取组题列表失败')
      }
    },
    // 清楚的方法
    onClear() {
      this.$refs.ClearForm.resetFields()
      this.getRandomsQuestions()
    },
    viewQuestions(item) {
      this.numberID = item
      this.dialogVisible = true
    },
    // 删除的功能
    async onDelete(id) {
      this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          try {
            await removeRandoms({ id })
            this.$message({ type: 'success', message: '删除成功!' })
            this.getRandomsQuestions()
          } catch (err) {
            this.$message({
              message: '删除失败'
            })
          }
        })
        .catch(() => {})
    },
    handel1(val) {
      this.searchForm.page = val
      this.getRandomsQuestions()
    },
    handel2(val) {
      this.searchForm.pagesize = val
      this.getRandomsQuestions()
    }
  }
}
</script>

<style scoped lang="scss">
.el-input {
  width: 20%;
}
</style>
