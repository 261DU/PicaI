<template lang="pug">
mixin pagenator
  .pagenator
    button(@click='handlePageChange(page - 1)', :disabled='page <= 1')
      icon
        arrow-left
    .page(@click='handlePagePrompt')
      .cur-page {{ page }}
      | /
      .total-page {{ totalPages }}
    button(@click='handlePageChange(page + 1)', :disabled='page >= totalPages')
      icon
        arrow-right

#search-container
  .bread-crumb
    router-link.button(to='/categories') 
      icon
        arrow-left
      | 
      | Categories Index

  label
    strong Keyword
    input(v-model='Keyword', type='text')
  label
    strong Sort
    select(v-model="Sort")
      option(value="ua") 默认
      option(value="dd") 新到旧
      option(value="da") 旧到新
      option(value="ld") 最多爱心
      option(value="vd") 最多指名
    

  div
    button(@click.prevent='gotoUrl') Search

  h1(v-if='keyword') Search『{{ keyword }}』comics (page {{ page }})
  h1(v-else) Advanced Search

  .info.error(v-if='error')
    .title Failed to get comics data
    p {{ error }}

  .loading.align-center(v-if='loading && !comics.length')
    placeholder

  section(v-if='comics.length', :class='{ "loading-cover": loading }')
    +pagenator
    books-list(:data='comics', :backTo='"/search/" + keyword + "?sort=" + sort + "&page=" + page ')
    +pagenator
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

type SortTypes = 'ua' | 'dd' | 'da' | 'ld' | 'vd'
const keyword = ref('')
const Keyword = ref('')
const category = ref('')
const page = ref(1)
const totalPages = ref(1)
const sort = ref<SortTypes>('ua')
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
  keyword.value = (route.params.keyword as string) || ''
  category.value = (route.query.category as string) || ''
  page.value = parseInt(route.query.page as string) || 1
  sort.value = (route.query.sort as SortTypes) || 'ua'

  if (keyword.value) {
    setTitle(`${keyword.value} (page ${page.value})`, 'Search')
    Keyword.value = keyword.value
  } else {
    setTitle('Search')
  }
  
  Sort.value = sort.value

  loading.value = true
  error.value = ''

  axios
    .post<ApiResponseComics>(
      `${API_BASE}/comics/advanced-search`,
      {
        keyword: keyword.value,
        categories: category.value,
        sort: sort.value,
      },
      {
        params: {
          page: page.value,
        },
      }
    )
    .then(
      ({ data }) => {
        comics.value = data.body?.comics.docs
        totalPages.value = data.body?.comics.pages
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

function handlePageChange(toPage: number) {
  router.push({
    query: { page: Math.min(totalPages.value, Math.max(1, toPage)), sort: sort.value },
  })
}

function handlePagePrompt() {
  const p = prompt('Page jump to', '' + page.value) || ''
  if (!isNaN(parseInt(p))) {
    handlePageChange(parseInt(p))
  }
}

function gotoUrl() {
  if (!Keyword.value) return
  
  if (!Sort.value) Sort.value = 'ua'
  
  router.push(`/search/${Keyword.value}?sort=${Sort.value}`)
}

// Refresh when the keyword changes
router.afterEach((to, from) => {
  console.log('after route', { to })
  if (to.name === from.name && to !== from) {
    keyword.value = route.params.keyword as string
    page.value = parseInt(to.query.page as string) || 1
    init()
  }
})

onMounted(() => {
  init()
  setTitle('Search')
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
