<template>
  <div class='sonTag-container'>
    <el-dialog
      :title="TagData.id ? '修改标签' : '新增标签' "
      :visible.sync="dialogVisible"
      width="30%">
      <el-form ref="SonForm" :rules="rules" :model="TagSonForm" label-width="80px">
        <el-form-item label="所属学科" prop="subjectID" v-if="!$route.query.id">
          <el-select v-model="TagSonForm.subjectID" placeholder="请选择学科" style="width: 100%">
            <el-option v-for="item in subjectList" :key="item.value" :label="item.label" :value="item.value"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="标签名称" prop="tagName">
          <el-input v-model="TagSonForm.tagName" placeholder="请输入标签名称" ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="sonTagPresent">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 学科
import { simple as simpleSubject } from '@/api/hmmm/subjects'
// 标签
import { add as addTag, update as updateTag } from '@/api/hmmm/tags'
export default {
  name: 'TagsAdd',
  props: ['TagData'],
  data () {
    return {
      dialogVisible: false,
      // 子组件提交的参数
      TagSonForm: {
        // 学科id
        subjectID: null,
        // 目录id
        tagName: null
      },
      // 学科列表
      subjectList: [],
      // 效验
      rules: {
        tagName: [
          { required: true, message: '请输入标签名称', trigger: 'blur' }
        ]
      }
    }
  },
  mounted () {
    this.getSubject()
  },
  methods: {
    // 打开对话框
    openDialog () {
      this.dialogVisible = true
      // 有数据就代表是修改
      if (this.TagData) {
        this.TagSonForm.subjectID = this.TagData.subjectID
        this.TagSonForm.tagName = this.TagData.tagName
      }
      // 移除表单项的校验结果
      this.$nextTick(() => {
        this.$refs.SonForm.clearValidate()
      })
    },
    // 获取学科
    async getSubject () {
      const { data } = await simpleSubject()
      this.subjectList = data
    },
    // 提交
    sonTagPresent () {
      this.$refs.SonForm.validate(async (valid) => {
        if (valid) {
          // 如果有地址栏里有学科id，就直接把学科id添加进去
          if (this.$route.query.id) this.TagSonForm.subjectID = this.$route.query.id
          if (this.TagData.id) {
            await updateTag({ id: this.TagData.id, ...this.TagSonForm })
            this.$message.success('修改成功！')
          } else {
            await addTag(this.TagSonForm)
            this.$message.success('添加成功！')
          }
          this.$emit('finish')
          this.dialogVisible = false
        }
      })
    }
  }
}
</script>

<style scoped lang='scss'>
::v-deep .el-dialog__footer {
  text-align: right !important;
}
</style>
