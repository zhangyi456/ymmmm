65t56t<template>
  <div class="tags-container">
    <el-card class="box-card">
      <div slot="header" class="clearfix" v-if="$route.query.name">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item>学科管理</el-breadcrumb-item>
          <el-breadcrumb-item>{{ $route.query.name }}</el-breadcrumb-item>
          <el-breadcrumb-item>标签管理</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-row>
        <el-col :span="20">
          <el-form
            ref="TagsForm"
            :inline="true"
            :model="reqParams"
            label-width="80px"
            size="small"
          >
            <el-form-item label="标签名称" prop="tagName">
              <el-input
                v-model="reqParams.tagName"
                placeholder="请输入标签名称"
              ></el-input>
            </el-form-item>
            <el-form-item label="状态" prop="state">
              <el-select v-model="reqParams.state" placeholder="请选择">
                <el-option
                  v-for="item in statusList"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button @click="eliminate">清除</el-button>
              <el-button type="primary" @click="tagSearch">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="4" align="right">
          <el-button
            v-if="$route.query.id"
            @click="getBackSubject"
            type="text"
            icon="el-icon-back"
            >返回学科</el-button
          >
          <el-button
            type="success"
            @click="addTags({})"
            icon="el-icon-edit"
            size="small"
            >新增标签</el-button
          >
        </el-col>
      </el-row>
      <el-alert
        :title="`数据一共 ${total} 条`"
        type="info"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- 表格 -->
      <el-table :data="tableData" style="width: 100%; margin-top: 20px">
        <el-table-column label="序号" type="index" width="60"></el-table-column>
        <el-table-column label="所属学科" prop="subjectName"></el-table-column>
        <el-table-column label="标签名称" prop="tagName"></el-table-column>
        <el-table-column label="创建者" prop="username"></el-table-column>
        <el-table-column label="创建日期">
          <template slot-scope="scope">
            <span>{{ scope.row.addDate | parseTimeByString }}</span>
          </template>
        </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <span>{{ scope.row.state === 1 ? '已启用' : '已禁用' }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button @click="editState(scope.row)" type="text">{{
              scope.row.state === 1 ? '禁用' : '启用'
            }}</el-button>
            <el-button
              @click="editTag(scope.row)"
              type="text"
              :disabled="scope.row.state === 1"
              >修改</el-button
            >
            <el-button
              @click="removeTag(scope.row.id)"
              type="text"
              :disabled="scope.row.state === 1"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        style="float: right; margin: 20px"
        background
        @size-change="changeSize"
        @current-change="changePage"
        :current-page="reqParams.page"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="reqParams.pagesize"
        layout="prev, pager, next, sizes, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <tags-add ref="TagsAdd" @finish="finish" :TagData="TagData"></tags-add>
  </div>
</template>

<script>
// 标签相关接口
import { list as tagList, changeState, remove } from '@/api/hmmm/tags'
// 状态, 修改状态
import { status } from '@/api/hmmm/constants'
// 把新增和修改弹层组件引入
import TagsAdd from '../components/tags-add'
export default {
  name: 'Tags',
  components: {
    TagsAdd
  },
  data () {
    return {
      reqParams: {
        // 标签名
        tagName: null,
        // 学科id
        subjectID: this.$route.query.id,
        // 状态
        state: null,
        page: 1,
        pagesize: 10
      },
      total: 0,
      // 表格数据
      tableData: [],
      // 状态列表
      statusList: status,
      // 点击修改传给子组件的数据
      TagData: []
    }
  },
  watch: {
    '$route.query.id': function () {
      this.reqParams.subjectID = this.$route.query.id || null
      this.getTagLists()
    }
  },
  mounted () {
    this.getTagLists()
  },
  methods: {
    // 获取列表数据
    async getTagLists () {
      const { data } = await tagList(this.reqParams)
      this.total = data.counts
      this.tableData = data.items
    },
    // 清除
    eliminate () {
      this.$refs.TagsForm.resetFields()
      this.getTagLists()
    },
    // 搜索
    tagSearch () {
      this.reqParams.page = 1
      this.getTagLists()
    },
    // 如果是从学科过来的，就有返回学科按钮
    getBackSubject () {
      this.$router.push({ path: '/list', name: 'subjects-list' })
    },
    // 新增标签
    addTags (row) {
      // 新增传的是一个 {}
      this.TagData = row
      this.$nextTick(() => {
        this.$refs.TagsAdd.openDialog()
      })
    },
    // 修改标签
    editTag (row) {
      this.TagData = row
      this.$nextTick(() => {
        this.$refs.TagsAdd.openDialog()
      })
    },
    // 接收子组件的方法
    finish () {
      // 只有新增的时候，需要回到第一页
      if (!this.TagData.id) this.reqParams.page = 1
      this.getTagLists()
    },
    // 修改状态
    async editState (row) {
      const txt = row.state === 1 ? '禁用' : '启用'
      const newState = row.state === 1 ? 0 : 1
      await changeState({ id: row.id, state: newState })
      this.$message.success(`${txt}成功!`)
      this.getTagLists()
    },
    // 删除标签
    removeTag (id) {
      this.$confirm('此操作将永久删除该标签, 是否继续?', '温馨提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          try {
            await remove({ id })
            this.$message.success('删除成功！')
            const tr = document.querySelectorAll('tr')
            if (tr.length === 2 && this.reqParams.page !== 1) {
              this.reqParams.page--
            }
            this.getTagLists()
          } catch (e) {
            this.$message.error('删除失败！')
          }
        })
        .catch(() => {})
    },
    // 当前页改变
    changePage (currentPage) {
      this.reqParams.page = currentPage
      this.getTagLists()
    },
    // 页面条数改变
    changeSize (currentSize) {
      this.reqParams.page = 1
      this.reqParams.pagesize = currentSize
      this.getTagLists()
    }
  }
}
</script>

<style scoped lang='scss'>
::v-deep.el-card {
  margin: 10px;
}
</style>
