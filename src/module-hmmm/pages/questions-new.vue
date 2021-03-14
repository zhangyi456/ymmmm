<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        <div slot="header" class="clearfix">
          <span>{{ $route.query.id ? '试题修改' : '试题录入' }}</span>
        </div>
        <div class="el-card__body">
          <el-form
            :label-position="labelPosition"
            label-width="80px"
            :model="addQuestion"
            show-message
            :rules="rules"
            ref="questionsForm"
          >
            <el-form-item label="学科:" prop="subjectID" required>
              <el-select
                v-model="addQuestion.subjectID"
                placeholder="请选择"
                @change="selectSubjectOne(addQuestion.subjectID)"
              >
                <el-option
                  :label="item.label"
                  :value="item.value"
                  v-for="(item, index) in subjectList"
                  :key="index"
                ></el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="二级目录" prop="catalogID">
              <el-select v-model="addQuestion.catalogID" placeholder="请选择">
                <el-option
                  :label="item.label"
                  :value="item.value"
                  v-for="item in twoDirectory"
                  :key="item.value"
                ></el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="企业:" prop="enterpriseID" required>
              <el-select
                v-model="addQuestion.enterpriseID"
                placeholder="请选择"
              >
                <el-option
                  :label="item.company"
                  :value="item.id"
                  v-for="item in companysList"
                  :key="item.id"
                ></el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="城市" prop="province" required id="city">
              <el-select
                :style="{ width: '22%' }"
                v-model="addQuestion.province"
                placeholder="请选择"
                @change="handleProvince"
              >
                <el-option
                  v-for="item in citySelect.province"
                  :key="item"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
              <el-select
                :style="{ width: '21%' }"
                v-model="addQuestion.city"
                placeholder="请选择"
              >
                <el-option
                  v-for="item in citySelect.cityDate"
                  :key="item"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="方向:" prop="direction" required>
              <el-select v-model="addQuestion.direction" placeholder="请选择">
                <el-option
                  :label="item"
                  :value="item"
                  v-for="(item, index) in direction"
                  :key="index"
                ></el-option>
              </el-select>
            </el-form-item>

            <el-form-item label="题型:" prop="questionType" required>
              <el-radio v-model="addQuestion.questionType" label="1">
                单选
              </el-radio>
              <el-radio v-model="addQuestion.questionType" label="2">
                多选
              </el-radio>
              <el-radio v-model="addQuestion.questionType" label="3">
                简答
              </el-radio>
            </el-form-item>

            <el-form-item label="难度:" prop="difficulty" required>
              <el-radio v-model="addQuestion.difficulty" label="1">
                简单
              </el-radio>
              <el-radio v-model="addQuestion.difficulty" label="2">
                一般
              </el-radio>
              <el-radio v-model="addQuestion.difficulty" label="3">
                困难
              </el-radio>
            </el-form-item>

            <el-form-item label="题干:" prop="question" required>
              <quill-editor
                :options="editorOption"
                v-model="addQuestion.question"
              ></quill-editor>
            </el-form-item>

            <el-form-item
              label="选项:"
              prop="options"
              v-if="addQuestion.questionType !== '3'"
            >
              <div
                class="one-item"
                v-for="(item, index) in addQuestion.options"
                :key="item.code"
              >
                <!-- 单选 -->
                <el-radio
                  v-if="addQuestion.questionType === '1'"
                  class="option-radio"
                  v-model="item.isRight"
                  @change="changeRadio(item)"
                  :label="true"
                  >{{ item.code }}：
                </el-radio>
                <!-- 多选 -->
                <el-checkbox
                  v-if="addQuestion.questionType === '2'"
                  class="option-radio"
                  v-model="item.isRight"
                  :label="true"
                  >{{ item.code }}：
                </el-checkbox>
                <!-- 输入框 -->
                <el-input v-model="item.title"></el-input>
                <!-- 上传 -->
                <div style="position: relative; width: 100px; hieght: 60px">
                  <el-upload
                    class="avatar-uploader"
                    :ref="`clearUpload${index}`"
                    action="http://localhost:3000/upload"
                    :on-success="
                      (res, file) => {
                        updateImg(item, res, file)
                      }
                    "
                    :show-file-list="false"
                  >
                    <img v-if="item.img" :src="item.img" class="avatar" />
                    <span v-if="!item.img">上传图片</span>
                  </el-upload>
                  <i
                    class="el-icon-circle-close ibtn"
                    style="position: absolute; top: -5px; left: 90px"
                    @click="clearPhotoFile(item, index)"
                  ></i>
                </div>
              </div>
              <!-- 增加试题按钮 -->
              <el-button
                :disabled="addQuestion.questionType === '1' || isDisabled"
                type="danger"
                icon="el-icon-plus"
                size="small"
                @click="addSelectInput"
                >增加选项与答案</el-button
              >
            </el-form-item>

            <el-form-item
              label="解析视频:"
              prop="videoURL"
              style="{width:100px}"
            >
              <el-input
                v-model="addQuestion.videoURL"
                class="videoIpt"
              ></el-input>
            </el-form-item>

            <el-form-item
              label="答案解析:"
              prop="answer"
              required
              class="answerQuestion"
            >
              <quill-editor
                :options="editorOption"
                v-model="addQuestion.answer"
              ></quill-editor>
            </el-form-item>

            <el-form-item label="题目备注:" prop="remarks">
              <el-input
                :style="{ width: '50%' }"
                id="remarksText"
                type="textarea"
                :rows="2"
                placeholder="请输入内容"
                v-model="addQuestion.remarks"
              >
              </el-input>
            </el-form-item>

            <el-form-item
              label="试题标签："
              prop="tags"
              required
              id="addQuestion"
            >
              <el-select v-model="addQuestion.tags" placeholder="请选择">
                <el-option
                  :label="item.label"
                  :value="item.label"
                  v-for="item in taglist"
                  :key="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
            <!-- 操作 -->
            <el-form-item label="">
              <el-button
                v-if="!$route.query.id"
                @click="SubmitQuestions"
                type="primary"
                size="medium"
                >确认提交</el-button
              >
              <el-button
                v-else
                @click="btnModification"
                type="success"
                size="medium"
                >确认修改</el-button
              >
            </el-form-item>
          </el-form>
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
import { detail, add, update } from '@/api/hmmm/questions.js'
import { provinces, citys } from '@/api/hmmm/citys.js'
import { addQuillTitle } from '@/utils/quill-title.js'
import { list } from '@/api/hmmm/companys.js'
export default {
  name: 'addQuestion',
  data() {
    return {
      labelPosition: 'right',
      addQuestion: {
        subjectID: null, // 选中的学科id
        questionType: '1', // 试题类型
        difficulty: '1', // 试题难度
        catalogID: null, // 目录
        direction: null, // 方向
        remarks: null, // 题目备注
        enterpriseID: null, // 企业
        province: null, // 企业所在地省份
        city: null,
        // 选项
        options: [
          { isRight: false, code: 'A', title: '', img: '' },
          { isRight: false, code: 'B', title: '', img: '' },
          { isRight: false, code: 'C', title: '', img: '' },
          { isRight: false, code: 'D', title: '', img: '' }
        ],
        question: null, // 题干
        videoURL: null, // 解析视频
        answer: null, // 答案解析
        tags: null
      },
      citySelect: {
        province: [],
        cityDate: []
      },
      // 富文本list
      editorOption: {
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'], // toggled buttons
            [{ list: 'ordered' }, { list: 'bullet' }, 'blockquote'],
            ['code-block', 'image', 'link']
          ]
        }
      },
      companysList: [], // 公司项目
      twoDirectory: [], // 二级目录
      tagList: [], // 标签列表
      direction: direction, // 方向的数据
      subjectList: [], // 学科列表
      isDisabled: false, // 控制是否禁用
      // 规则验证
      rules: {
        subjectID: [
          { required: true, message: '请选择学科', trigger: 'change' }
        ],
        catalogID: [
          { required: true, message: '请选择目录', trigger: 'change' }
        ],
        enterpriseID: [
          { required: true, message: '请选择企业', trigger: 'change' }
        ],
        province: [
          { required: true, message: '请选择城市', trigger: 'change' }
        ],
        city: [{ required: true, message: '请选择地区', trigger: 'change' }],
        direction: [
          { required: true, message: '请选择方向', trigger: 'change' }
        ],
        questionType: [
          { required: true, message: '请选择题型', trigger: 'change' }
        ],
        difficulty: [
          { required: true, message: '请选择题型', trigger: 'change' }
        ],
        question: [{ required: true, message: '请输入题干' }],
        answer: [
          { required: true, message: '请输入答案解析', trigger: 'blur' }
        ],
        tags: [{ required: true, message: '请选择标签', trigger: 'blur' }]
      },
      taglist: [], // 标签列表
      English: [
        'A',
        'B',
        'C',
        'D',
        'E',
        'F',
        'G',
        'H',
        'I',
        'J',
        'K',
        'L',
        'M',
        'N',
        'O',
        'P',
        'Q',
        'R',
        'S',
        'T',
        'U',
        'V',
        'W',
        'X',
        'Y',
        'Z'
      ]
    }
  },
  created() {
    // 获取1级列表
    this.getSimple()
    // 获取城市
    this.getCityData()
    // 获取企业
    this.getCompanys()
    // 获取修改过来的数据
    this.getEditQuestionList()
  },
  methods: {
    // 获取所有1级学科
    async getSimple() {
      const { data } = await simple()
      // console.log(data)
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
      this.addQuestion.city = this.citySelect.cityDate[0]
      this.addQuestion.city = null
    },
    // 获取企业
    async getCompanys() {
      const { data } = await list()
      this.companysList = data.items
    },
    // !点击修改时获取的数据
    async getEditQuestionList() {
      const ids = this.$route.query.id
      if (ids) {
        const { data } = await detail({ id: ids })
        // !因为获取radio需要选中，需要false和true，label和v-model都为true时是选中，而返回值为0，1，所以要进行转化
        data.options = data.options.map(item => {
          item.isRight = item.isRight === 1
          return item
        })
        // 修改试题获取的数据进行处理
        this.addQuestion = data
        const res = await simples({
          subjectID: this.addQuestion.subjectID
        })
        this.twoDirectory = res.data
        this.citySelect.cityDate = citys(this.addQuestion.province)
      }
    },
    // 选择了单选，只让当前生效
    changeRadio(CurrentItem) {
      this.addQuestion.options.forEach(item => {
        if (item !== CurrentItem) {
          item.isRight = false
        }
      })
      // 只让自己的选中，其余的不可以选中
      // CurrentItem.isRight = true
    },
    // 提交试题
    SubmitQuestions() {
      console.log(this.addQuestion)
      this.$refs.questionsForm.validate(async valid => {
        if (valid) {
          console.log(this.addQuestion)
          await add(this.addQuestion)
          this.$message.success('添加题库成功')
          this.$router.push('/questions/list')
        }
      })
    },
    // 修改试题
    async btnModification() {
      this.$refs.questionsForm.validate(async valid => {
        if (valid) {
          try {
            await update(this.addQuestion)
            this.$message.success('修改成功')
            this.$router.push('/questions/list')
          } catch (err) {
            console.log(err)
          }
        }
      })
      this.isEditOradd = false
    },
    // 多选状态下增加input数量
    addSelectInput() {
      const i = this.addQuestion.options.length
      // console.log(this.English[i])
      this.addQuestion.options.push({
        isRight: false,
        code: this.English[i],
        title: '',
        img: ''
      })
    },
    // 图片添加成功后的钩子函数
    // 上传成功, 把返回的图片地址，赋值预览图片
    updateImg(item, response, file) {
      // ! 拿到的file文件的名称被改变了，无法打开，    需要转成blob形式，传值必须为file.raw
      item.img = URL.createObjectURL(file.raw)
    },
    // 清除图片
    async clearPhotoFile(item) {
      const flag = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
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
      if (flag === 'confirm') {
        item.img = ''
      }
    }
  },
  mounted() {
    addQuillTitle()
  }
}
</script>

<style scoped lang='scss'>
.el-select {
  width: 43%;
}
::v-deep.city .el-form-item__content {
  margin-left: 25px !important;
}
.el-form-item__content {
  width: 50%;
}
::v-deep.videoIpt {
  width: 43% !important;
}
::v-deep.ql-editor {
  height: 200px !important;
}
::v-deep #remarksText {
  height: 100px;
}
/* // 选项样式------------- */
.one-item {
  display: flex;
  align-items: center;
  padding-bottom: 20px;
}
.avatar-uploader {
  vertical-align: middle;
  line-height: 1;
}
::v-deep .avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  width: 100px;
  height: 60px;
  line-height: 60px;
}
::v-deep .avatar-uploader .el-upload:hover {
  border-color: cornflowerblue;
}
.avatar {
  width: 100px;
  height: 60px;
}
/* // 单选框 */
.option-radio {
  margin-right: 0;
}
/* // 输入框 */
::v-deep.el-input {
  width: 230px;
  margin-right: 5px;
}
/* // 删除的小图标 */
.ibtn {
  cursor: pointer;
}

::v-deep.answerQuestion .el-form-item__label {
  width: 83px !important;
}
::v-deep #addQuestion .el-form-item__label {
  white-space: nowrap;
}
::v-deep.quill-editor {
  .ql-container {
    .ql-editor {
      height: 210px !important;
    }
  }
}

</style>
