<template>
  <div class="subject-add-container">
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogFormVisible"
      width="400px"
    >
      <el-form :model="dialogData" :rules="ruleForm" ref="dialogForm">
        <el-form-item label="学科名称" prop="subjectName">
          <el-input
            v-model="dialogData.subjectName"
            placeholder="请输入学科名称"
          ></el-input>
        </el-form-item>
        <el-form-item
          label="是否显示"
          prop="isFrontDisplay"
          class="switchLabel"
        >
          <el-switch
            v-model="dialogData.isFrontDisplay"
            active-color="#13ce66"
            inactive-color="#ff4949"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
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
import { update, add } from '@/api/hmmm/subjects.js'

export default {
  name: 'subjectAdd',
  components: {},
  props: {
    dialogData: {
      type: Object,
      required: true
    },
    dialogTitle: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      dialogFormVisible: false,
      ruleForm: {
        subjectName: [
          { required: true, message: '学科不能为空', trigger: 'blur' }
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
    // 添加学科
    confirmAdd() {
      this.$refs.dialogForm.validate(async valid => {
        if (valid) {
          this.dialogFormH()
          // 如果数据中存在id，则执行修改操作，否则执行添加操作
          if (this.dialogData.id) {
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
.el-form .el-input {
  width: 200px;
}
.switchLabel {
  margin-left: 10px;
}
</style>
