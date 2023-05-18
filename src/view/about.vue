<template lang="pug">
#about-container
  h1 关于

  h2 使用方法
  .card
    p 以普遍理性而论，相较于原版，现在这个版本是个人都会用
  
  h2 与原版相比…
  .card
    p 1.优化了search，comics页面的搜索以及排序功能
    p 2.添加了排行榜获取的功能
    p 3.添加并完善了哔咔签到功能
    p 4.完善了用户头像的获取
    p 5.微调了部分UI
    p 6.对接了私人搭建的图床，避免了无法访问哔咔官方图床的问题
    
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import { setTitle } from '../utils/setTitle'

onMounted(() => {
  setTitle('About')
})
</script>

<style scoped lang="sass"></style>
