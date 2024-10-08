<template lang="pug">
component(
  :is='loaded ? "img" : "svg"',
  :width='width',
  :height='height',
  :src='src',
  :class='{ lazyload: true, isLoading: !loaded && !error, isLoaded: loaded, isError: error }',
  role='img',
  ref='imgRef'
)
</template>

<script lang="ts" setup>
import { nextTick, onBeforeUnmount, onMounted, ref } from 'vue'
const props = defineProps<{
  src: string
  width?: number
  height?: number
}>()
const loaded = ref(false)
const error = ref(false)
const imgRef = ref<HTMLImageElement>()
function init() {
  const img = new Image()
  img.src = props.src
  img.onload = () => {
    loaded.value = true
    error.value = false
  }
  img.onerror = () => {
    loaded.value = false
    error.value = true
  }
}
let observer: IntersectionObserver
onMounted(async () => {
  await nextTick()
  const img = imgRef.value
  if (!img) return
  observer = new IntersectionObserver((entries) => {
    const [entry] = entries
    if (entry.isIntersecting) {
      init()
      observer.disconnect()
    }
  })
  observer.observe(img)
})
onBeforeUnmount(() => {
  observer && observer.disconnect()
})
</script>

<style scoped lang="sass">
.isLoading
  animation: imgProgress 0.6s ease infinite alternate
</style>