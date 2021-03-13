<template>
  <div class="el-dialog__body">
    <div class="el-row">
      <div class="el-col el-col-6">
        <span v-if="viewItem.questionType == 1">【题型】：单选题</span>
        <span v-else-if="viewItem.questionType == 2">【题型】：多选题</span>
        <span v-else>【题型】：简答题</span>
      </div>
      <div class="el-col el-col-6">【编号】：{{ viewItem.id }}</div>
      <div class="el-col el-col-6">
        <span v-if="viewItem.difficulty == 1">【难度】：简单</span>
        <span v-else-if="viewItem.difficulty == 2">【难度】：一般</span>
        <span v-else>【难度】：困难</span>
      </div>
      <div class="el-col el-col-6">【标签】：{{ viewItem.tags }}</div>
      <div class="el-col el-col-6">【学科】：{{ viewItem.subjectName }}</div>
      <div class="el-col el-col-6">【目录】：{{ viewItem.directoryName }}</div>
      <div class="el-col el-col-6">【方向】：{{ viewItem.direction }}</div>
    </div>
    <hr />
    【题干】：
    <div style="color: blue;">
      <div v-html="viewItem.question"></div>
    </div>
    <div>
      <div style="padding-bottom: 5px;">
        <span v-if="viewItem.questionType == 1"
          >单选题 选项：（以下选中的选项为正确答案）</span
        >
        <span v-else-if="viewItem.questionType == 2"
          >：多选题 选项：（以下选中的选项为正确答案）</span
        >
        <span v-else>简答题 选项：（以下选中的选项为正确答案）</span>
      </div>
      <!-- 选项栏 -->
      <div
        style="padding: 8px 0px;"
        v-for="(item, index) in viewItem.options"
        :key="index"
      >
        <label
          role="radio"
          tabindex="0"
          :class="item.isRight == 0 ? 'el-radio' : 'el-radio is-checked'"
        >
          <span
            :class="
              item.isRight == 0
                ? 'el-radio__input'
                : 'el-radio__input is-checked'
            "
          >
            <span class="el-radio__inner"></span>
            <input
              type="radio"
              aria-hidden="true"
              tabindex="-1"
              class="el-radio__original"
              :value="item.title"
            />
          </span>
          <span class="el-radio__label">{{ item.title }}</span>
        </label>
      </div>
    </div>
    <hr />
    【参考答案】：
    <button
      type="button"
      class="el-button el-button--danger el-button--small"
      @click="playVideo"
    >
      <span>视频答案预览</span>
    </button>
    <div class="video" style="display: none;" id="video">
      <video controls="controls" :src="viewItem.videoURL"></video>
    </div>
    <hr />
    【答案解析】：
    <div style="display: inline-block;">
      <div v-html="viewItem.answer"></div>
    </div>
    <hr />
    【题目备注】：{{ viewItem.remarks }}
    <div>
      <el-button
        type="primary"
        id="close"
        :style="{ float: 'right', margin: '10px 0' }"
        @click="$emit('close')"
        >关闭</el-button
      >
    </div>
  </div>
</template>

<script>
import { detail } from '@/api/hmmm/questions'
export default {
  name: '',
  props: {
    viewItemId: {
      type: [String, Number],
      require: true
    }
  },

  data() {
    return {
      viewItem: []
    }
  },
  created() {
    this.getViewQuestion()
  },

  methods: {
    playVideo() {
      document.getElementById('video').style.display = 'block'
    },
    async getViewQuestion() {
      try {
        const { data } = await detail({ id: this.viewItemId })
        this.viewItem = data
        // console.log(data)
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style scoped>
.el-col {
  padding-bottom: 10px;
}
.el-dialog__body {
  font-size: 15px;
}
video {
  padding-top: 10px;
  width: 50%;
}
</style>
