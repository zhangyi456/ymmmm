<template>
  <div class="directory-add-container">
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogFormVisible"
      width="400px"
    >
      <el-form :model="dialogData" :rules="ruleForm" ref="dialogForm">
        <el-form-item label="所属学科" prop="subjectID" class="subjectName">
          <el-select v-model="dialogData.subjectID" placeholder="请选择">
            <el-option
              v-for="(subName, index) in subjectNameList"
              :key="index"
              :label="subName.subjectName"
              :value="subName.subjectId"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="目录名称" prop="directoryName">
          <el-input
            v-model="dialogData.directoryName"
            placeholder="请输入目录名称"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirmAdd">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { update, add } from '@/api/hmmm/directorys.js'

export default {
  name: 'directoryAdd',
  components: {},
  props: {
    dialogData: {
      type: Object,
      required: true
    },
    dialogTitle: {
      type: String,
      required: true
    },
    subjectNameList: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      dialogFormVisible: false,
      ruleForm: {
        directoryName: [
          { required: true, message: '学科名称不能为空', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {},
  methods: {
    // 弹层显示
    dialogFormV() {
      this.dialogFormVisible = true
    },
    // 弹层隐藏
    dialogFormH() {
      this.dialogFormVisible = false
    },
    // 添加目录
    confirmAdd() {
      this.$refs.dialogForm.validate(async valid => {
        if (valid) {
          this.dialogFormH()
          // console.log(this.dialogData, 222)
          // 如果数据中存在addDate，则执行修改操作，否则执行添加操作
          if (this.dialogData.addDate) {
            // console.log('执行修改操作')
            // 调用接口，更新数据
            await update(this.dialogData).then(() => {
              // 子向父传值，进行修改后的数据渲染
              this.$emit('newData', this.dialogData)
            })
          } else {
            await add(this.dialogData).then(() => {
              // 子向父传值，进行添加后的数据渲染
              this.$emit('newData', this.dialogData)
            })
            console.log(this.dialogData)
          }
        } else {
          this.$message.error('*号为必填项!')
        }
      })
    }
  }
}
</script>

<style scoped>
.el-form .el-input,
.el-form .el-select {
  width: 200px;
}
.subjectName {
  margin-left: 10px;
}
</style>
