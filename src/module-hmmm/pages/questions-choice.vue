<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        <el-row
          type="flex"
          justify="space-between"
          style="margin-bottom: 10px;"
        >
          <el-alert
            title="说明：目前支持学科和关键字条件筛选"
            type="error"
            :closable="false"
          >
          </el-alert>
          <el-col :span="6" style="text-align: right;">
            <el-button
              type="success"
              icon="el-icon-edit"
              size="small"
              @click="
                $router.push({
                  path: '/new',
                  name: 'questions-new'
                })
              "
              >新增试题</el-button
            >
          </el-col>
        </el-row>
        <!-- 表单 -->
        <el-form
          ref="choiceForm"
          :model="picklists"
          label-width="80px"
          size="small"
        >
          <el-row>
            <el-col :span="6">
              <el-form-item label="学科" prop="subjectID">
                <!-- @change="changeSubject" -->
                <el-select
                  v-model="picklists.subjectID"
                  placeholder="请选择学科"
                  style="width: 100%"
                  @change="onSimples(picklists.subjectID)"
                >
                  <el-option
                    v-for="item in subjectsList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="二级目录" prop="catalogID">
                <el-select
                  v-model="picklists.catalogID"
                  placeholder="请选择目录"
                  style="width: 100%"
                >
                  <el-option
                    v-for="item in simples"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="标签" prop="tags">
                <el-select
                  v-model="picklists.tags"
                  placeholder="请选择标签"
                  style="width: 100%"
                >
                  <el-option
                    v-for="item in tagrs"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="关键字" prop="keyword">
                <el-input
                  v-model="picklists.keyword"
                  placeholder="请输入关键字"
                ></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="6">
              <el-form-item label="试题类型" prop="questionType">
                <el-select
                  v-model="picklists.questionType"
                  placeholder="请选择试题类型"
                  style="width: 100%"
                >
                  <el-option
                    v-for="item in questionType"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="难度" prop="difficulty">
                <el-select
                  v-model="picklists.difficulty"
                  placeholder="请选择难度"
                  style="width: 100%"
                >
                  <el-option
                    v-for="item in difficulty"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="方向" prop="direction">
                <el-select
                  v-model="picklists.direction"
                  placeholder="请选择方向"
                  style="width: 100%"
                >
                  <el-option
                    v-for="item in direction"
                    :key="item"
                    :label="item"
                    :value="item"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="录入人" prop="creatorID">
                <el-select
                  v-model="picklists.creatorID"
                  placeholder="请选择录入人"
                  style="width: 100%"
                >
                  <el-option
                    v-for="item in userList"
                    :key="item.id"
                    :label="item.username"
                    :value="item.id"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="6">
              <el-form-item label="题目备注" prop="remarks">
                <el-input
                  v-model="picklists.remarks"
                  placeholder="请输入题目备注"
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="企业简称" prop="shortName">
                <el-input
                  v-model="picklists.shortName"
                  placeholder="请输入企业简称"
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="城市" prop="province">
                <el-row type="flex" justify="space-between">
                  <el-col :span="12">
                    <el-select
                      v-model="picklists.province"
                      @change="handleProvince"
                      placeholder="请选择"
                      style="width: 100%;"
                    >
                      <el-option
                        v-for="item in citySelect.province"
                        :key="item"
                        :label="item"
                        :value="item"
                      ></el-option>
                    </el-select>
                  </el-col>
                  <el-col :span="11">
                    <el-form-item prop="city">
                      <el-select
                        v-model="picklists.city"
                        placeholder="请选择"
                        style="width: 100%"
                      >
                        <el-option
                          v-for="item in citySelect.cityDate"
                          :key="item"
                          :label="item"
                          :value="item"
                        ></el-option>
                      </el-select>
                    </el-form-item>
                  </el-col>
                </el-row>
              </el-form-item>
            </el-col>
            <el-col :span="6" style="text-align: right;">
              <el-form-item>
                <el-button @click="onclear">清除</el-button>
                <el-button type="primary" @click="onSearch">搜索</el-button>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
        <!-- /表单 -->
        <!-- tab切换 -->
        <el-tabs v-model="activeName" type="card" @tab-click="handleClick">
          <el-tab-pane label="全部" name="null"></el-tab-pane>
          <el-tab-pane label="待审核" name="0"></el-tab-pane>
          <el-tab-pane label="已审核" name="1"></el-tab-pane>
          <el-tab-pane label="已拒绝" name="2"></el-tab-pane>
        </el-tabs>
        <!-- tab切换 -->
        <el-alert
          :title="`数据一共${total}条`"
          show-icon
          :closable="false"
        ></el-alert>
        <!-- 列表 -->
        <div class="tabList">
          <el-table
            :data="choicesList"
            style="width: 100%"
            header-align="center"
          >
            <el-table-column
              prop="number"
              label="试题编号"
              width="120"
              align="center"
            >
            </el-table-column>
            <el-table-column
              prop="subject"
              label="学科"
              width="120"
              align="center"
            >
            </el-table-column>
            <el-table-column
              prop="catalog"
              label="目录"
              width="120"
              align="center"
            >
            </el-table-column>
            <el-table-column
              prop="questionType"
              label="题型"
              width="120"
              align="center"
            >
              <template slot-scope="scope">
                <div>
                  {{
                    scope.row.questionType === '1'
                      ? '单选'
                      : scope.row.questionType === '2'
                      ? '多选'
                      : '简答'
                  }}
                </div>
              </template>
            </el-table-column>
            <el-table-column
              prop="question"
              label="题干"
              width="260"
              align="center"
            >
              <template slot-scope="scope">
                <div v-html="scope.row.question"></div>
              </template>
            </el-table-column>
            <el-table-column
              prop=""
              label="录入时间"
              width="180"
              align="center"
            >
              <template slot-scope="scope">
                <div fit>
                  {{ dayjs(scope.row.addDate).format('YYYY-MM-DD hh:mm:ss') }}
                </div>
              </template>
            </el-table-column>
            <el-table-column
              prop="difficulty"
              label="难度"
              width="80"
              align="center"
            >
              <template slot-scope="scope">
                <div>
                  {{
                    scope.row.difficulty === '1'
                      ? '简单'
                      : scope.row.difficulty === '2'
                      ? '一般'
                      : '困难'
                  }}
                </div>
              </template>
            </el-table-column>
            <el-table-column
              prop="creator"
              label="录入人"
              width="120"
              align="center"
            >
            </el-table-column>
            <el-table-column
              prop="chkState"
              label="审核状态"
              width="120"
              align="center"
            >
              <template slot-scope="scope">
                <div v-if="scope.row.chkState === 0">待审核</div>
                <div v-else-if="scope.row.chkState === 1">已审核</div>
                <div v-else>已拒绝</div>
              </template>
            </el-table-column>
            <el-table-column
              prop="chkRemarks"
              label="审核意见"
              width="150"
              align="center"
            >
            </el-table-column>
            <el-table-column
              prop="chkUser"
              label="审核人"
              width="120"
              align="center"
            >
            </el-table-column>
            <el-table-column
              prop="publishState"
              label="发布状态"
              width="150"
              align="center"
            >
              <template slot-scope="scope">
                <span
                  v-if="
                    scope.row.chkState === 1 && scope.row.publishState === 1
                  "
                  >{{ '已发布' }}</span
                >
                <span
                  v-else-if="
                    scope.row.chkState === 1 && scope.row.publishState === 0
                  "
                  >{{ '已下架' }}</span
                >
                <span v-else>{{ '待发布' }}</span>
              </template>
            </el-table-column>
            <el-table-column
              fixed="right"
              label="操作"
              style="text-align: center"
              width="220"
              align="center"
            >
              <template slot-scope="scope">
                <el-button
                  @click="viewQuestions(scope.row)"
                  type="text"
                  size="small"
                  >预览</el-button
                >
                <el-button
                  type="text"
                  size="small"
                  @click="showView(scope.row)"
                  :disabled="
                    !(scope.row.publishState === 0 && scope.row.chkState === 0)
                  "
                  >审核</el-button
                >
                <el-button
                  type="text"
                  size="small"
                  :disabled="scope.row.publishState === 1"
                  @click="
                    $router.push({
                      path: '/new',
                      name: 'questions-new',
                      query: { id: scope.row.id }
                    })
                  "
                  >修改</el-button
                >
                <el-button
                  type="text"
                  size="small"
                  @click="onShelves(scope.row)"
                  >{{
                    scope.row.publishState === 0 ? '上架' : '下架'
                  }}</el-button
                >
                <el-button
                  type="text"
                  size="small"
                  @click="onRemove(scope.row.id)"
                  :disabled="scope.row.publishState === 1"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </div>
        <!-- /列表 -->
        <!-- 分页 -->
        <el-pagination
          style="margin: 20px; float: right"
          background
          @current-change="changePage"
          @size-change="changeSize"
          :current-page="picklists.page"
          :page-sizes="[5, 10, 15, 20]"
          :page-size="picklists.pagesize"
          layout="prev, pager, next, sizes, jumper"
          :total="total"
        >
        </el-pagination>
      </el-card>
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
      <!-- 审核弹出层 -->
      <el-dialog
        title="题目审核"
        :visible.sync="isAuditShow"
        width="30%"
        :model="auditForm"
      >
        <el-radio label="1" v-model="auditForm.chkState">通过</el-radio>
        <el-radio label="2" v-model="auditForm.chkState">拒绝</el-radio>
        <el-input
          type="textarea"
          style="margin-top: 30px"
          v-model="auditForm.chkRemarks"
          placeholder="请输入审核意见"
        ></el-input>
        <span slot="footer">
          <el-button @click="isAuditShow = false">取 消</el-button>
          <el-button type="primary" @click="onUpdateAudit()">确 定</el-button>
        </span>
      </el-dialog>
      <!-- /审核弹出层 -->
    </div>
  </div>
</template>

<script>
import { simple } from '@/api/hmmm/subjects'
import { simple as simples } from '@/api/hmmm/directorys'
import { simple as simpless } from '@/api/hmmm/tags'
import { questionType, difficulty, direction } from '@/api/hmmm/constants'
import { provinces, citys } from '@/api/hmmm/citys.js'
import {
  choice,
  choiceCheck,
  choicePublish,
  remove as removeQuestion
} from '@/api/hmmm/questions'
import viewQuestion from '../components/questions-preview'
import dayjs from 'dayjs'
// 用户接口
import { simple as simpleUser } from '@/api/base/users'
export default {
  name: 'QuestionChoice',
  components: { viewQuestion },
  data() {
    return {
      labelPosition: 'right',
      picklists: {
        page: 1, // 当前页面数
        pagesize: 10, // 页面尺寸，当前页面条数
        subjectID: null, // 学科
        twoLevelDirectory: null, // 学科二级目录
        difficulty: null, // 学科难度
        questionType: null, // 试题类型
        tags: null, // 标签名称
        province: null, // 企业所在省份
        city: null, // 企业所在城市
        keyword: null, // 关键字
        remarks: null, // 题目备注
        shortName: null, // 企业简称
        direction: null, // 方向
        creatorID: null, // 录入人
        catalogID: null, // 目录
        chkState: null // 审核状态
      },
      subjectsList: [], // 学科列表
      simples: [], // 标签列表
      tagrs: [],
      userList: [], // 录入人
      questionType: questionType,
      difficulty: difficulty,
      direction: direction,
      citySelect: {
        province: [], // 获取省份
        cityDate: [] // 获取城市
      },
      activeName: 'null',
      choicesList: [
        {
          number: null, // 试题编号
          subject: null, // 学科
          catalog: null, // 目录
          questionType: questionType.lable, // 题型
          question: null, // 题干
          addDate: null, // 录入时间
          difficulty: null, // 难度
          creator: null, // 录入人
          chkType: null, // 审核状态
          chkRemarks: null, // 审核意见
          chkUser: null, // 审核人
          publishState: null // 发布状态
        }
      ],
      dayjs, // 应用dayjs方法渲染日期
      total: 0, // 获取的总数居条数
      dialogVisible: false, // 控制预览弹出层显示与隐藏
      isAuditShow: false, // 控制审核弹出层显示与隐藏
      // inEditShow: false, // 控制修改弹出层的显示与隐藏
      numberID: null,
      auditForm: {
        chkState: '', // 通过和拒绝的意见数据
        chkRemarks: '' // 审核意见的文本数据
      },
      currentAuditId: null
    }
  },
  created() {
    // 获取学科的方法
    this.onSujects()
    //  获取城市的方法
    this.getCityData()
    // 获取用户
    this.getUser()
    // 获取精选题库列表
    this.getChoicesList()
  },
  methods: {
    // 获取学科列表
    async onSujects() {
      try {
        const { data } = await simple()
        this.subjectsList = data
      } catch (err) {
        this.$message('获取学科列表失败，请稍后再试')
      }
    },
    // 获取学科二级目录
    async onSimples(id) {
      try {
        const { data } = await simples({
          subjectID: id
        })
        this.simples = data
        const datas = await simpless({
          subjectID: id
        })
        this.tagrs = datas.data
      } catch (err) {
        this.$message('获取学科二级目录失败')
      }
    },
    // 获取省
    getCityData: function() {
      this.citySelect.province = provinces()
    },
    // 选省获取到市
    handleProvince: function(e) {
      this.citySelect.cityDate = citys(e)
      this.picklists.city = this.citySelect.cityDate[0]
    },
    // 获取录入人
    async getUser() {
      const { data } = await simpleUser({ keyword: this.picklists.keyword })
      this.userList = data
    },
    // 获取精选题库列表
    async getChoicesList() {
      const choicesListOBJ = { ...this.choicesList }
      if (choicesListOBJ.chkState === 'all') {
        choicesListOBJ.chkState = null
      }
      const { data } = await choice()
      this.choicesList = data.items
      this.total = data.counts
    },
    // 点击TAB 切换的事件
    async handleClick(tag) {
      if (tag.name == 'null') {
        const { data } = await choice()
        this.choicesList = data.items
        this.total = data.counts
      } else {
        const { data } = await choice({ chkState: tag.name })
        this.choicesList = data.items
        this.total = data.counts
      }
    },
    // 清除按钮
    onclear() {
      //  清除form表单
      this.$refs.choiceForm.resetFields()
      // 重新渲染列表
      this.getChoicesList()
    },
    // 搜索的方法
    async onSearch() {
      console.log(this.picklists)
      const { data } = await choice(this.picklists)
      this.choicesList = data.items
      this.total = data.counts
    },
    viewQuestions(item) {
      this.numberID = item.id
      this.dialogVisible = true
    },
    // 点击审核获取当前试题的两个状态
    showView(item) {
      this.currentAuditId = item.id
      console.log(item)
      this.auditForm.chkState = item.chkState.toString()
      this.isAuditShow = true
    },
    // 提交审核的方法
    async onUpdateAudit() {
      if (this.auditForm.chkRemarks === '')
        return this.$message('请填写审核意见')
      this.auditForm.chkState = parseInt(this.auditForm.chkState)
      const { data } = await choiceCheck({
        id: this.currentAuditId,
        chkState: this.auditForm.chkState,
        chkRemarks: this.auditForm.chkRemarks
      })
      this.getChoicesList()
      this.isAuditShow = false
      this.$message.success('审核成功')
    },
    // 上下架的方法
    onShelves(row) {
      const ISpublishState = row.publishState === 0 ? '1' : '0'
      const DDD = row.publishState === 0 ? '上架' : '下架'
      this.$confirm(`您确认${DDD}这道题目呢?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          await choicePublish({ id: row.id, publishState: ISpublishState })
          try {
            this.$message({
              type: 'success',
              message: `${DDD}成功!`
            })
            this.getChoicesList()
          } catch (err) {
            this.$message(`${DDD}失败!`)
          }
        })
        .catch(() => {})
    },
    // 删除的方法
    onRemove(id) {
      this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          try {
            await removeQuestion({ id })
            this.$message({ type: 'success', message: '删除成功!' })
            this.getChoicesList()
          } catch (err) {
            this.$message({
              message: '删除失败'
            })
          }
        })
        .catch(() => {})
    },
    // 分页
    async changePage(currentPage) {
      this.picklists.page = currentPage
      const { data } = await choice({
        page: this.picklists.page,
        pagesize: this.picklists.pagesize
      })
      this.choicesList = data.items
      // this.getChoicesList()
    },
    // 每页显示的条数
    async changeSize(currentSize) {
      this.picklists.pagesize = currentSize
      const { data } = await choice({
        page: this.picklists.page,
        pagesize: this.picklists.pagesize
      })
      this.choicesList = data.items
    }
  }
}
</script>

<style scoped lang="scss">
.head {
  display: flex;
  justify-content: space-between;
}
.el-alert--error {
  background-color: #fff;
}
.from-select {
  margin-top: 10px;
  font-size: 12px;
}
.foot-btn-clear {
  margin-left: 41px;
}
.select-btn {
  width: 48%;
}
.el-alert {
  margin-bottom: 15px;
}
.el-dialog__footer {
  text-align: right;
}
</style>
