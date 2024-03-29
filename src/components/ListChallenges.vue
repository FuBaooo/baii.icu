<script lang="ts" setup>
import { useRouter } from 'vue-router'

const props = defineProps<{ path: string }>()

const { t } = useI18n()

interface LevelListItem {
  title: string
  level: string
  children: {
    title: string
    date: string
    path: string
  }[]
}

const router = useRouter()
const list = router.getRoutes()
  .filter((i) => {
    return i.name && i.path.startsWith(props.path) && i.meta.frontmatter?.date
  })
const count = list.length
const routes = list
  .sort((a, b) => +new Date(b.meta.frontmatter.date) - +new Date(a.meta.frontmatter.date))
  .reduce((acc, cur) => {
    const { level, levelTitle, title, date } = cur.meta.frontmatter
    let levelIndex = acc.findIndex(i => level === i.level)
    if (levelIndex < 0) {
      acc.push({
        level,
        title: levelTitle,
        children: [],
      })
      levelIndex = acc.length - 1
    }
    acc[levelIndex].children.push({
      title,
      date,
      path: cur.path,
    })
    return acc
  }, [] as LevelListItem[])
</script>

<template>
  <nav class="prose m-auto">
    <template v-for="route in routes" :key="route.level">
      <h3 class="mt-10 font-bold">
        {{ route.title }}
      </h3>
      <ul class="ml-10 project-grid">
        <router-link
          v-for="child in route.children"
          :key="child.path"
          class="item block font-normal mb-6 mt-2 no-underline"
          :to="child.path"
        >
          <li class="no-underline">
            <div class="title text-lg">
              {{ child.title }}
            </div>
            <div class="time opacity-50 text-sm -mt-1">
              <!-- {{ formatDate(child.date) }} -->
            </div>
          </li>
        </router-link>
      </ul>
    </template>
    <h3>{{ t('title.count') }}: {{ count }}</h3>
  </nav>
</template>

<style scoped>
.project-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}
</style>
