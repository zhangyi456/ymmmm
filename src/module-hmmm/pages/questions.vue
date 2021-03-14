<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        <div class="btn_wrapper">
          <span style="font-size: 12px; color: pink;">
            说明：目前支持学科和关键字条件筛选
          </span>
          <button
            type="button"
            class="el-button el-button--success el-button--small"
            @click="$router.push('/questions/new')"
          >
            <i class="el-icon-edit"></i>
            <span>新增试题</span>
          </button>
        </div>

        <!-- 选择栏 -->
        <el-form
          class="selectbar"
          :label-position="labelPosition"
          label-width="80px"
          :model="formSelect"
          ref="formSelect"
        >
          <el-form-item label="学科" class="el-col el-col-6" prop="subjectID" >
            <el-select
              v-model="formSelect.subjectID"
              placeholder="请选择"
              @change="selectSubjectOne(formSelect.subjectID)"
              style="width: 100%"
            >
              <el-option
                :label="item.label"
                :value="item.value"
                v-for="(item, index) in subjectList"
                :key="index"
                @change="selectSubjectOne(item.value)"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item
            label="二级目录"
            class="el-col el-col-6"
            prop="catalogID"
          >
            <el-select v-model="formSelect.catalogID" placeholder="请选择" style="width: 100%">
              <el-option
                :label="item.label"
                :value="item.value"
                v-for="item in twoDirectory"
                :key="item.value"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="标签" class="el-col el-col-6" prop="tags">
            <el-select v-model="formSelect.tags" placeholder="请选择" style="width: 100%">
              <el-option
                :label="item.label"
                :value="item.value"
                v-for="item in taglist"
                :key="item.value"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="关键字" class="el-col el-col-6" prop="keyword">
            <el-input
              v-model="formSelect.keyword"
              placeholder="根据题干搜索"
            ></el-input>
          </el-form-item>

          <el-form-item
            label="试题类型"
            class="el-col el-col-6"
            prop="questionType"
          >
            <el-select v-model="formSelect.questionType" placeholder="请选择" style="width: 100%">
              <el-option label="单选" value="1"></el-option>
              <el-option label="多选" value="2"></el-option>
              <el-option label="简答" value="3"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="难度" class="el-col el-col-6" prop="difficulty">
            <el-select v-model="formSelect.difficulty" placeholder="请选择" style="width: 100%">
              <el-option label="简单" value="1"></el-option>
              <el-option label="一般" value="2"></el-option>
              <el-option label="困难" value="3"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="方向" class="el-col el-col-6" prop="direction">
            <el-select v-model="formSelect.direction" placeholder="请选择" style="width: 100%">
              <el-option
                :label="item"
                :value="index"
                v-for="(item, index) in directions"
                :key="index"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="录入人" class="el-col el-col-6" prop="creatorID">
            <el-select v-model="formSelect.creatorID" placeholder="请选择" style="width: 100%">
              <el-option label="超级管理员" value="0"></el-option>
              <el-option label="录入管理员" value="1"></el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="题目备注" class="el-col el-col-6" prop="remarks">
            <el-input v-model="formSelect.remarks"></el-input>
          </el-form-item>

          <el-form-item
            label="企业简称"
            class="el-col el-col-6"
            prop="shortName"
          >
            <el-input v-model="formSelect.shortName"></el-input>
          </el-form-item>

          <el-form-item label="城市" class="el-col el-col-3" prop="province">
            <el-select
              v-model="formSelect.province"
              placeholder="请选择"
              class="el-col el-col-12"
              @change="handleProvince"
              :style="{ width: '120%' }"
            >
              <el-option
                v-for="item in citySelect.province"
                :key="item"
                :label="item"
                :value="item"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item class="el-col-3 oness" prop="city">
            <el-select
              :style="{ margin: '0 0 0 50px', width: '75%' }"
              v-model="formSelect.city"
              placeholder="请选择"
              class="el-col el-col-12"
            >
              <el-option
                v-for="item in citySelect.cityDate"
                :key="item"
                :label="item"
                :value="item"
              ></el-option>
            </el-select>
          </el-form-item>

          <el-form-item class="el-col el-col-6">
            <div class="searchBtn">
              <el-button plain size="mini" @click="resetForm('formSelect')"
                >清除</el-button
              >
              <el-button type="primary" size="mini" @click="checkQuestion"
                >搜索
              </el-button>
            </div>
          </el-form-item>
        </el-form>

        <!-- 文字提示 -->
        <el-alert
          :title="`数据一共 ${count} 条`"
          type="info"
          :closable="false"
          show-icon
        >
        </el-alert>

        <!-- 表格 -->
        <div>
          <el-table
            :data="questionList"
            style="width: 100%"
            id="tableList"
            align="center"
            header-align="center"
          >
            <el-table-column
              prop="number"
              label="试题编号"
              width="220"
              align="center"
            >
            </el-table-column>
            <el-table-column prop="subject" label="学科" align="center">
            </el-table-column>
            <el-table-column prop="catalog" label="目录" align="center">
            </el-table-column>
            <el-table-column label="题型" align="center">
              <div slot-scope="scope">
                <span v-if="scope.row.questionType == 1">单选</span>
                <span v-else-if="scope.row.questionType == 2">多选</span>
                <span v-else>简答</span>
              </div>
            </el-table-column>
            <el-table-column label="题干" align="center">
              <div slot-scope="scope" v-html="scope.row.question"></div>
            </el-table-column>
            <el-table-column label="录入时间" width="220" align="center">
              <template slot-scope="scope">
                <span>{{ scope.row.addDate | parseTimeByString }}</span>
              </template>
            </el-table-column>
            <el-table-column label="难度" align="center">
              <div slot-scope="scope">
                <span v-if="scope.row.difficulty == 1">简单</span>
                <span v-else-if="scope.row.difficulty == 2">一般</span>
                <span v-else>困难</span>
              </div>
            </el-table-column>
            <el-table-column
              prop="creator"
              label="录入人"
              width="120"
              align="center"
            >
            </el-table-column>
            <el-table-column label="操作" width="220" align="center">
              <template slot-scope="scope">
                <div class="cell">
                  <button
                    type="button"
                    class="el-button el-button--primary el-button--small is-plain is-circle"
                    title="预览"
                    @click="viewQusetionItem(scope.row)"
                  >
                    <i class="el-icon-view"></i>
                  </button>
                  <button
                    type="button"
                    class="el-button el-button--success el-button--small is-plain is-circle"
                    title="修改"
                    @click="
                      $router.push({
                        path: '/new',
                        name: 'questions-new',
                        query: { id: scope.row.id }
                      })
                    "
                  >
                    <i class="el-icon-edit"></i>
                  </button>
                  <button
                    data-v-48e452ac=""
                    type="button"
                    class="el-button el-button--danger el-button--small is-plain is-circle"
                    title="删除"
                    @click="delQuestion(scope.row.id)"
                  >
                    <i class="el-icon-delete"></i>
                  </button>
                  <button
                    data-v-48e452ac=""
                    type="button"
                    class="el-button el-button--warning el-button--small is-plain is-circle"
                    title="加入精选"
                    @click="choiceAdd(scope.row.id)"
                  >
                    <i class="el-icon-check"></i>
                  </button>
                </div>
              </template>
            </el-table-column>
          </el-table>

          <!--题目预览弹出层  -->
          <el-dialog
            title="题目预览"
            :visible.sync="isViewQuestion"
            width="70%"
            v-if="isViewQuestion"
          >
            <viewQuestionItem
              :viewItemId="viewItem"
              @close="isViewQuestion = false"
            />
          </el-dialog>

          <!-- 分页 -->
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="formSelect.page"
            :page-sizes="[5, 10, 20, 50]"
            :page-size="formSelect.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="count"
          >
          </el-pagination>
        </div>
      </el-card>
    </div>
  </div>
</template>

<script>
import { simple } from '@/api/hmmm/subjects.js'
import { simple as simples } from '@/api/hmmm/directorys.js'
import { simple as simpless } from '@/api/hmmm/tags.js'
import { direction } from '@/api/hmmm/constants.js'
import { provinces, citys } from '@/api/hmmm/citys.js'
import { list, remove, choiceAdd } from '@/api/hmmm/questions.js'
import viewQuestionItem from '../components/viewQuestionItem'
import { parseTimeByString } from '@/filters/index.js'
export default {
  name: 'question',
  data() {
    return {
      formSelect: {
        page: 1,
        pagesize: 10,
        subjectID: null, // 选中的学科id
        questionType: null, // 试题类型
        difficulty: null, // 试题难度
        catalogID: null, // 目录
        tags: null, // 标签名称
        keyword: null, // 关键字
        direction: null, // 方向
        creatorID: null, // 录入人
        remarks: null, // 题目备注
        shortName: null, // 企业简称
        province: null, // 企业所在地省份
        city: null
      },
      citySelect: {
        province: [],
        cityDate: []
      },
      directions: direction, // 方向的数据
      subjectList: [], // 学科列表
      twoDirectory: [], // 二级目录列表
      taglist: [], // 标签列表
      labelPosition: 'right',
      count: 0, // 总数,
      questionList: [], // 表格数据的列表
      isViewQuestion: false, // VIEW视图层
      viewItem: [],
      parseTimeByString: parseTimeByString // 事件处理器
    }
  },
  methods: {
    // 获取所有1级学科
    async getSimple() {
      const { data } = await simple()
      this.subjectList = data
    },
    // 改变1级学科，获得2级目录
    async selectSubjectOne(id) {
      const { data } = await simples({
        subjectID: id
      })
      this.twoDirectory = data
      var res = await simpless({
        subjectID: id
      })
      this.taglist = res.data
    },
    // 获取省
    getCityData: function() {
      this.citySelect.province = provinces()
    },
    // 选省获取到市
    handleProvince: function(e) {
      this.citySelect.cityDate = citys(e)
      this.formSelect.cityDate = this.citySelect.cityDate[0]
      this.formSelect.city = null
    },
    // 清除
    resetForm(formName) {
      this.$refs[formName].resetFields()
      this.getQuestionList()
    },
    // 搜索
    checkQuestion() {
      this.getQuestionList()
    },

    // 获取table表格数据
    async getQuestionList() {
      const { data } = await list(this.formSelect)
      this.questionList = data.items
      this.count = data.counts
    },
    // 删除题目
    async delQuestion(id) {
      const flag = await this.$confirm(
        '此操作将永久删除该题目, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      if (flag) {
        await remove({ id })
        this.$message.success('删除成功')
        const tr = document.querySelectorAll('tr')
        if (tr.length === 2 && this.formSelect.page !== 1) {
          this.formSelect.page--
        }
        this.getQuestionList()
      }
    },
    // 加入精选
    async choiceAdd(id) {
      try {
        const flag = await this.$confirm(
          '此操作将该题目加入精选, 是否继续?',
          '提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        ).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消加入精选'
          })
        })
        if (flag) {
          await choiceAdd({
            id,
            choiceState: 1
          })
          this.$message.success('加入精选成功')
          this.getQuestionList()
        }
      } catch (err) {
        this.$message.err('加入精选失败')
      }
    },
    viewQusetionItem(item) {
      this.isViewQuestion = true
      this.viewItem = item.id
    },
    handleSizeChange(val) {
      this.formSelect.pagesize = val
      this.getQuestionList()
    },
    handleCurrentChange(val) {
      this.formSelect.page = val
      this.getQuestionList()
    }
  },
  created() {
    // 获取1级列表
    this.getSimple()
    // 获取城市
    this.getCityData()
    // 获取table表格数据
    this.getQuestionList()
  },
  components: {
    viewQuestionItem
  }
}
</script>

<style scoped lang="scss">
.btn_wrapper {
  display: flex;
  justify-content: space-between;
}
.selectbar {
  margin-top: 20px;
}
.searchBtn {
  float: right;
}
.tableList {
  margin-top: 20px;
}
::v-deep.oness .el-form-item__content {
  margin-left: 0 !important;
}
.el-pagination {
  padding: 20px 0;
  float: right;
}
::v-deep #tableList {
  .el-table td,
  .el-table th {
    text-align: center !important;
  }
}
// ::v-deep#tableList {
//   .el-table__body-wrapper {
//     .el-table__body {
//       width: 100% !important;
//     }
//   }
// .el-table__header {
//   width: 100% !important;
//   .has-gutter {
//     width: 100%;
//     tr {
//       display: flex;
//       width: 100%;
//       th {
//         flex: 1;
//       }
//     }
//   }
// }
// }
</style>
