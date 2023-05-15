<template lang="pug">
#about-container
  h1 关于

  h2 使用方法
  .card
    p 本站主架构基于Pica Comic Now项目
    p 源项目没做搜索栏，我也没那能力重开发，因此你只能使用以下方式使用搜索：
    p 访问：/search/{搜索内容}?sort={排序方式}
    p 排序方式：ua: 默认, dd: 新到旧, da: 旧到新, ld: 最多爱心, vd: 最多指名
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { setTitle } from '../utils/setTitle'

onMounted(() => {
  setTitle('About')
})
</script>

<style scoped lang="sass"></style>
