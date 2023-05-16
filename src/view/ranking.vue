<template lang="pug">

#search-container
  .bread-crumb
    router-link.button(to='/categories') 
      icon
        arrow-left
      | 
      | Categories Index

  label
    strong Sort
    select(v-model="Sort")
      option(value="H24") 近24小时
      option(value="D7") 近7天
      option(value="D30") 近30天
    

  div
    button(@click.prevent='gotoUrl') GO

  h1 Ranking

  .info.error(v-if='error')
    .title Failed to get comics data
    p {{ error }}

  .loading.align-center(v-if='loading && !comics.length')
    placeholder
    
  section(v-if='comics.length', :class='{ "loading-cover": loading }')
    books-list(:data='comics', :backTo='"/ranking" + "?sort=" + sort + "&page=" + page ')
</template>

<script setup lang="ts">
import axios from 'axios'
import { onMounted, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { getErrMsg } from '../utils/getErrMsg'
import { setTitle } from '../utils/setTitle'
import { ArrowLeft, ArrowRight } from '@vicons/fa'
import BooksList from '../components/BooksList.vue'
import { API_BASE } from '../config'
import type { ApiResponseComics } from '../types'
const route = useRoute()
const router = useRouter()

// const components = defineComponent({ BooksList })

type SortTypes = 'H24' | 'D7' | 'D30'
const category = ref('')
const page = ref(1)
const totalPages = ref(1)
const sort = ref<SortTypes>('H24')
const Sort = ref('')

const comics = ref<any[]>([])
const loading = ref(false)
const error = ref('')

/**
 * @param arg1.category 分区名字，categories里面的title，如"嗶咔漢化"
 * @param arg1.page 分页，从1开始
 * @param arg1.sort 排序依据：
 * ```
 * ua: 默认
 * dd: 新到旧
 * da: 旧到新
 * ld: 最多爱心
 * vd: 最多指名
 * ```
 */
function init() {
  sort.value = (route.query.sort as SortTypes) || 'H24'

  if (sort.value) {
    setTitle(`${sort.value}`, 'Ranking')
    Sort.value = sort.value
  } else {
    setTitle('Ranking')
  }

  loading.value = true
  error.value = ''

  axios
    .get(
      `${API_BASE}/comics/leaderboard?ct=VC&tt=${sort.value}`,
      {
        ct: 'VC',
        tt: sort.value,
      },
      {
        params: {
        },
      }
    )
    .then(
      ({ data }) => {
        comics.value = data.body?.comics
      },
      (err) => {
        console.warn('Failed to get comics data', err)
        error.value = getErrMsg(err)
      }
    )
    .finally(() => {
      loading.value = false
    })
}


function gotoUrl() {
  
  if (!Sort.value) Sort.value = 'H24'
  
  router.push(`/ranking?sort=${Sort.value}`)
}

// Refresh when the keyword changes
router.afterEach((to, from) => {
  console.log('after route', { to })
  if (to.name === from.name && to !== from) {
    init()
  }
})

onMounted(() => {
  setTitle('Ranking')
  init()
})
</script>

<style scoped lang="sass">
.pagenator
  text-align: center
  > *
    display: inline-block
  .page
    margin-left: 1rem
    margin-right: 1rem
    background-color: var(--theme-accent-color)
    color: #fff
    padding: 0.25rem 0.6rem
    border-radius: 1em
    display: inline-flex
    gap: 0.4rem
    cursor: pointer
</style>
