<template>
  <!-- 清除prop里面必须有数据 -->
  <div class="dashboard-container">
    <!-- //为什么有这个才有card出来 -->
    <div class="container">
      <el-card shadow="never" style="margin: 15px">
        <!-- <el-card shadow="never"> -->
        <el-row :gutter="20">
          <el-form
            label-width="80px"
            ref="requestParameters"
            :model="requestParameters"
          >
            <el-col :span="6">
              <el-form-item label="关键字" prop="keyword">
                <el-input
                  v-model="requestParameters.keyword"
                  placeholder="根据文章标题搜搜"
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="5">
              <el-form-item label="状态" prop="state">
                <el-select
                  v-model="requestParameters.state"
                  placeholder="请选择"
                >
                  <el-option label="启用" :value="1"></el-option>
                  <el-option label="禁用" :value="0"></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-form>
          <el-col :span="2">
            <el-button plain size="small" @click="clear">清除</el-button>
          </el-col>
          <el-col :span="1">
            <el-button type="primary" size="small" @click="handleFilter"
              >搜索</el-button
            >
          </el-col>
          <el-col :span="7" style="position: relative; top: 0; right: -400px;">
            <el-button
              type="success"
              size="small"
              class="el-icon-edit newskills"
              style="margin-left: -46px;"
              @click="handleChange('add'), (articleVisible = true)"
              >新增文章</el-button
            >
          </el-col>
        </el-row>
        <!-- 消息提示文案 -->
        <el-alert
          :title="alertText"
          type="info"
          class="alert"
          :closable="false"
          show-icon
        ></el-alert>

        <!-- 消息提示文案 -->
        <!-- 文章数据列表 -->

        <el-table v-loading="listLoading" :data="dataList" style="width: 100%">
          <el-table-column type="index" label="序号" align="center">
          </el-table-column>
          <el-table-column prop="title" label="文章标题">
            <template slot-scope="scope">
              <span>{{ scope.row.title }}</span>
              <span
                v-if="scope.row.videoURL"
                class="el-icon-film"
                @click="open(scope.row.videoURL)"
                style="color: blue"
              ></span>
            </template>
          </el-table-column>
          <el-table-column prop="visits" label="阅读数"> </el-table-column>
          <el-table-column prop="username" label="录入人"> </el-table-column>
          <el-table-column label="录入时间" prop="createTime">
            <template slot-scope="scope">
              <span>{{ scope.row.createTime | parseTimeByString }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="state" label="状态">
            <template slot-scope="scope">
              <!-- scope.row.state判断状态不知道为啥，试出来的 -->
              <p v-if="scope.row.state">已启用</p>
              <p v-else>已禁用</p>
            </template>
          </el-table-column>
          <el-table-column label="操作" prop="state">
            <!-- 操作模块 四个增删改看 -->
            <template slot-scope="scope" :data="dataList">
              <el-button type="text" @click="show(scope.row.id)"
                >预览</el-button
              >
              <el-button
                v-if="!scope.row.state"
                type="text"
                @click="changeStart(scope.row)"
                >启用</el-button
              >
              <!-- 上面这里显示修改删除按钮 -->
              <el-button v-else type="text" @click="changeStart(scope.row)"
                >禁用</el-button
              >
              <el-button
                type="text"
                :disabled="!scope.row.state"
                @click="handleChange(scope.row.id), (articleVisible = true)"
                >修改</el-button
              >
              <el-button
                type="text"
                :disabled="!scope.row.state"
                @click="deleteArticle(scope.row)"
                >删除</el-button
              >
            </template>
            <!-- 操作模块 -->
          </el-table-column>
        </el-table>
        <!-- 文章数据列表 -->
        <!-- 分页开始 -->
        <div class="pagination">
          <div class="pages">
            <el-pagination
              background
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="Number(requestParameters.page)"
              :total="Number(total)"
              :page-size="Number(requestParameters.pagesize)"
              :page-sizes="[10, 20, 30, 50]"
              layout="sizes, prev, pager, next, jumper"
            ></el-pagination>
          </div>
        </div>
        <!-- 分页结束 -->

        <!-- 预览弹框 -->
        <el-dialog
          :append-to-body="true"
          title="预览文章"
          :visible.sync="seeFlag"
          width="50%"
          height="100%"
        >
          <articles-preview :showData="showData"></articles-preview>
          <span slot="footer" class="dialog-footer">
            <el-button @click="seeFlag = false">取 消</el-button>
            <el-button type="primary" @click="seeFlag = false">确 定</el-button>
          </span>
        </el-dialog>
        <!-- 预览弹框 -->
        <!-- 弹框，修改添加模块 -->
        <el-dialog
          :append-to-body="true"
          title="提示"
          :visible.sync="articleVisible"
          width="50%"
          height="100%"
        >
          <!-- 修改 -->
          <el-form
            ref="newdataList"
            :model="newdataList"
            :rules="rules"
            label-width="80px"
          >
            <el-form-item label="文章标题" prop="title">
              <el-input v-model="newdataList.title"></el-input>
            </el-form-item>
            <el-form-item label="文章内容" prop="articleBody">
              <!-- 富文本编辑 -->
              <quill-editor
                ref="text"
                v-model="newdataList.articleBody"
                class="myQuillEditor"
                :options="editorOption"
              />
              <!-- 富文本编辑-->
            </el-form-item>
            <el-form-item label="视频地址" prop="videoURL">
              <el-input v-model="newdataList.videoURL"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button @click="articleVisiblef(), resetForm('newdataList')"
                >取 消</el-button
              >
              <el-button
                type="primary"
                @click="articleVisiblef(), submitForm('newdataList')"
                >确 定</el-button
              >
            </el-form-item>
          </el-form>
          <!-- 修改 -->
        </el-dialog>
        <!-- 视频弹窗 -->
        <el-dialog
          :append-to-body="true"
          class="namedia"
          :visible.sync="showVideo"
          width="0"
          height="0"
          :style="{ padding: 0 }"
        >
          <span class="el-icon-close" @click="closecl"></span>
          <video width="680" height="520" controls autoplay class="video">
            <source :src="src" type="video/ogg" />
            <object :data="src" width="330" height="260">
              <embed width="330" height="260" :src="src" />
            </object>
          </video>
        </el-dialog>
        <!-- 弹框，修改添加模块 -->
      </el-card>
    </div>
  </div>
</template>

<script>
import {
  list,
  remove,
  changeState,
  add,
  update,
  detail
} from '@/api/hmmm/articles'
import { quillEditor } from 'vue-quill-editor'
import ArticlesPreview from '@/module-hmmm/components/articles-preview'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
export default {
  data() {
    return {
      requestParameters: {
        page: 1,
        pagesize: 10
      },
      src: '',
      seeFlag: false,
      editorOption: {
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'], // toggled buttons
            [{ list: 'ordered' }, { list: 'bullet' }, 'blockquote'],
            ['code-block', 'image', 'link']
          ]
        }
      },
      showVideo: false,
      // 富文本编辑器内容
      form: {
        name: '111'
      },
      rules: {
        title: [{ required: true, message: '请输入活动名称', trigger: 'blur' }],
        articleBody: [
          { required: true, message: '请输入活动名称', trigger: 'blur' }
        ]
      },
      articleVisible: false, // 弹框修改
      isDelete: true, // 是否禁用按钮
      listLoading: false,
      dataList: [
        {
          id: 1,
          title: '',
          visits: 1,
          username: '1',
          creatTime: 1,
          state: true,
          articleBody: '',
          videoURL: ''
        }
      ],
      // alist: {},
      newdataList: {},
      val: 'false',
      currentPage: 1,
      total: null,
      alertText: '0条数据',
      // 浏览的数据
      showData: {}
    }
  },
  components: {
    quillEditor,
    ArticlesPreview
  },
  created() {
    this.getArticleList()
  },
  methods: {
    articleVisiblef() {
      this.articleVisible = false
    }, // 关闭弹窗
    async getArticleList() {
      this.listLoading = true
      // this.requestParameters
      const { data } = await list(this.requestParameters)
      this.alertText = `数据一共${data.counts}条` // 总共数据
      this.dataList = data.items // 渲染数据的变量
      this.total = data.counts
      this.listLoading = false
    },
    // 视频
    open(url) {
      this.showVideo = true
      this.src = url
    },
    // 搜索信息
    handleFilter() {
      this.requestParameters.page = 1
      this.getArticleList(this.requestParameters)
    },
    // 搜索的项目
    // 添加、编辑之后表单清空

    async handleChange(id) {
      if (id !== 'add') {
        // 、编辑用户
        // 默认值
        this.newdataList = {}
        const { data } = await detail({ id: id })
        this.newdataList = data
      } else {
        // 新增用户
        console.log('新增用户')
        this.newdataList = {}
      }
    },
    async submitForm() {
      this.$refs.newdataList.validate(async valid => {
        if (valid) {
          this.articleVisible = false
          // console.log(this.newdataList.id, '判断用户id是添加还是修改')
          if (this.newdataList.id) {
            console.log(this.newdataList, '修改的提交')
            await update(this.newdataList)
            this.getArticleList()
          } else {
            try {
              await add(this.newdataList)
              this.getArticleList()
            } catch (error) {
              console.log(error)
            }
          }
        } else {
          this.$message.error('为必填项!')
        }
      })
    },
    closecl() {
      this.showVideo = false
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
    },
    // 预览的操作
    async show(id) {
      // 显示预览窗口
      this.seeFlag = true
      this.showData = {}

      const { data } = await detail({ id: id })
      this.showData = data
    },
    handleSizeChange(val) {
      this.requestParameters.pagesize = val
      if (this.requestParameters.page === 1) {
        this.getList(this.requestParameters)
      }
    },
    handleCurrentChange(val) {
      this.requestParameters.page = val
      this.getArticleList()
    },
    // 清除内容
    clear() {
      this.$refs.requestParameters.resetFields()
    },
    async changeStart(val) {
      val.state = !val.state
      // 调用接口
      var obj = { id: val.id, state: val.state }

      try {
        await changeState(obj)
        this.$message.success('成功')
      } catch (error) {
        this.$message.success('失败了' + error)
      }
    },
    async deleteArticle(val) {
      // console.log(val + '删除')
      try {
        // console.log(val.id + 'id：')
        // await remove (val)
        await remove(val)
        this.$message.success('删除成功')
        this.getArticleList() // 刷新列表
      } catch (error) {
        this.$message.fail('删除失败' + error)
        // !删除bug
        if (document.querySelectorAll('.el-card tbody tr').length === 1) {
          this.queryInfo.pagenum =
            this.queryInfo.pagenum > 1 ? this.queryInfo.pagenum - 1 : 1
        }
      }
    }
  }
}
</script>
<style scoped lang="scss">
.text {
  font-size: 14px;
}

.item {
  margin-bottom: 18px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: '';
}
.clearfix:after {
  clear: both;
}

.box-card {
  width: 480px;
}
::v-deep .el-dialog__header .namedia {
  position: relative;
  top: -900px;
  left: -700px;
  padding: 0 !important;
  background-color: transparent !important;
  display: none !important;
  border: 0 !important;
  background-color: aliceblue;
}
::v-deep .el-dialog__footer .namedia {
  padding: 0 !important;
  background-color: transparent !important;
  display: none !important;
  border: 0 !important;
}
::v-deep .el-icon-close:before .namedia {
  padding: 0 !important;
  display: none;
  border: 0 !important;
}
::v-deep .el-dialog__body .namedia {
  padding: 0 !important;
  position: relative;
  top: -150px;
  left: -100px;
  border: 0 !important;
}
::v-deep .el-dialog__body .video {
  padding: 0 !important;
  border: 0 !important;
  position: absolute;
  // left: -310px;
  top: 200px;
  transform: translate(-50%, -50%);
}
::v-deep.namedia {
  .el-dialog {
    .el-dialog__header {
      padding: 0;
    }
  }
}
.el-icon-close {
  position: absolute;
  z-index: 999;
  color: white;
  font-size: 40px;
  top: -70px;
  left: 0;
}
</style>
