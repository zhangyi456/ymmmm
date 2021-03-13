<template>
  <div class="container">
    <!-- 文章内容 -->
    <!-- 文章内容 -->
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <h1>{{ showData.title }}</h1>
        <div>
          <span>{{ showData.createTime }}</span
          ><span>{{ showData.username }}</span
          ><span class="el-icon-view">{{ showData.visits }}</span>
        </div>
      </div>
      <div v-html="showData.articleBody"></div>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {}
  },
  props: {
    showData: {
      type: Object,
      required: true,
      default: () => []
    }
  },
  created() {},
  methods: {}
}
</script>

<style scoped lang="less"></style>
