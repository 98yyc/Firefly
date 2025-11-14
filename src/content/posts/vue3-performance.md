
---
title: Vue3 性能优化实战指南
published: 2024-01-15
description: 深入学习Vue3性能优化技巧,包括代码分割、懒加载、虚拟列表、缓存策略等高级优化方案
tags: [Vue3, 性能优化, 前端, 进阶]
category: 前端开发
draft: true
---

Vue3在性能方面相比Vue2有了显著提升,但在实际项目中,我们仍需要掌握各种优化技巧来提升应用性能。本文将介绍Vue3中的各种性能优化方案。

## 1. 组件懒加载与代码分割

### 1.1 路由懒加载

使用动态导入实现路由级别的代码分割:

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'Home',
      component: () => import('/pages/Home.vue')
    },
    {
      path: '/about',
      name: 'About',
      // 使用注释指定chunk名称
      component: () => import(/* webpackChunkName: "about" */ '/pages/About.vue')
    },
    {
      path: '/dashboard',
      name: 'Dashboard',
      component: () => import('/pages/Dashboard.vue'),
      children: [
        {
          path: 'analytics',
          component: () => import('/pages/dashboard/Analytics.vue')
        }
      ]
    }
  ]
})

export default router
```

### 1.2 组件异步加载

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// 简单的异步组件
const AsyncComp = defineAsyncComponent(() =>
  import('./components/HeavyComponent.vue')
)

// 带选项的异步组件
const AsyncCompWithOptions = defineAsyncComponent({
  loader: () => import('./components/HeavyComponent.vue'),
  // 加载时显示的组件
  loadingComponent: LoadingComponent,
  // 加载失败显示的组件
  errorComponent: ErrorComponent,
  // 延迟显示加载组件的时间,默认200ms
  delay: 200,
  // 超时时间
  timeout: 3000
})
</script>

<template>
  <Suspense>
    <template #default>
      <AsyncComp />
    </template>
    <template #fallback>
      <div>Loading...</div>
    </template>
  </Suspense>
</template>
```

## 2. 虚拟列表优化

处理大量数据列表时,虚拟列表是必不可少的优化手段。

### 2.1 使用 vue-virtual-scroller

```bash
npm install vue-virtual-scroller
```

```vue
<script setup lang="ts">
import { RecycleScroller } from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

interface Item {
  id: number
  name: string
  description: string
}

const items = ref<Item[]>(
  Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`,
    description: `Description for item ${i}`
  }))
)
</script>

<template>
  <RecycleScroller
    class="scroller"
    :items="items"
    :item-size="80"
    key-field="id"
    v-slot="{ item }"
  >
    <div class="item">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </RecycleScroller>
</template>

<style scoped>
.scroller {
  height: 100vh;
}

.item {
  padding: 20px;
  border-bottom: 1px solid #eee;
}
</style>
```

### 2.2 自定义虚拟列表

```vue
<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

interface Props {
  items: any[]
  itemHeight: number
  containerHeight: number
}

const props = defineProps<Props>()

const scrollTop = ref(0)
const containerRef = ref<HTMLElement>()

// 可视区域内的起始索引
const startIndex = computed(() => 
  Math.floor(scrollTop.value / props.itemHeight)
)

// 可视区域内的结束索引
const endIndex = computed(() => 
  Math.min(
    startIndex.value + Math.ceil(props.containerHeight / props.itemHeight) + 1,
    props.items.length
  )
)

// 可见的列表项
const visibleItems = computed(() => 
  props.items.slice(startIndex.value, endIndex.value)
)

// 偏移量
const offsetY = computed(() => 
  startIndex.value * props.itemHeight
)

const handleScroll = (e: Event) => {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}

onMounted(() => {
  containerRef.value?.addEventListener('scroll', handleScroll)
})

onUnmounted(() => {
  containerRef.value?.removeEventListener('scroll', handleScroll)
})
</script>

<template>
  <div 
    ref="containerRef"
    class="virtual-list"
    :style="{ height: `${containerHeight}px` }"
  >
    <div 
      class="virtual-list-phantom"
      :style="{ height: `${items.length * itemHeight}px` }"
    />
    <div 
      class="virtual-list-content"
      :style="{ transform: `translateY(${offsetY}px)` }"
    >
      <div
        v-for="item in visibleItems"
        :key="item.id"
        class="virtual-list-item"
        :style="{ height: `${itemHeight}px` }"
      >
        <slot :item="item" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.virtual-list {
  overflow-y: auto;
  position: relative;
}

.virtual-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.virtual-list-content {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
}
</style>
```

## 3. 响应式数据优化

### 3.1 使用 shallowRef 和 shallowReactive

对于不需要深层响应式的数据,使用浅层响应式可以提升性能:

```typescript
import { shallowRef, shallowReactive } from 'vue'

// 只有根级别的属性是响应式的
const state = shallowReactive({
  user: {
    name: 'John',
    profile: {
      age: 30
    }
  }
})

// 只有 .value 的访问是响应式的
const bigList = shallowRef([
  /* 大量数据 */
])

// 更新整个列表
bigList.value = [...bigList.value, newItem]
```

### 3.2 使用 readonly 和 markRaw

```typescript
import { readonly, markRaw, reactive } from 'vue'

// 创建只读数据
const config = readonly({
  apiUrl: 'https://api.example.com',
  timeout: 5000
})

// 标记为非响应式
const nonReactiveData = markRaw({
  heavyData: /* ... */
})

const state = reactive({
  data: nonReactiveData // 不会被转换为响应式
})
```

### 3.3 computed 缓存优化

```typescript
import { ref, computed } from 'vue'

const list = ref([1, 2, 3, 4, 5])

// computed 会缓存计算结果
const expensiveSum = computed(() => {
  console.log('计算中...')
  return list.value.reduce((sum, num) => sum + num, 0)
})

// 只在依赖变化时重新计算
console.log(expensiveSum.value) // 计算中... 15
console.log(expensiveSum.value) // 15 (使用缓存)
```

## 4. v-memo 指令优化

Vue3.2+ 新增的 `v-memo` 指令可以缓存模板的一部分:

```vue
<script setup lang="ts">
const list = ref([
  { id: 1, name: 'Item 1', selected: false },
  { id: 2, name: 'Item 2', selected: false },
  // ... 更多项
])

const selectItem = (id: number) => {
  list.value = list.value.map(item =>
    item.id === id ? { ...item, selected: !item.selected } : item
  )
}
</script>

<template>
  <div v-for="item in list" :key="item.id" v-memo="[item.selected]">
    <!-- 只有当 item.selected 改变时才重新渲染 -->
    <div @click="selectItem(item.id)">
      <span>{{ item.name }}</span>
      <span v-if="item.selected">✓</span>
    </div>
  </div>
</template>
```

## 5. 事件处理优化

### 5.1 事件委托

```vue
<script setup lang="ts">
const handleClick = (e: MouseEvent) => {
  const target = e.target as HTMLElement
  if (target.matches('.item')) {
    const id = target.dataset.id
    console.log('Clicked item:', id)
  }
}
</script>

<template>
  <!-- 使用事件委托,只绑定一个事件处理器 -->
  <div class="list" @click="handleClick">
    <div v-for="item in items" :key="item.id" 
         class="item" :data-id="item.id">
      {{ item.name }}
    </div>
  </div>
</template>
```

### 5.2 防抖和节流

```typescript
import { ref } from 'vue'
import { useDebounceFn, useThrottleFn } from '@vueuse/core'

// 防抖搜索
const searchText = ref('')
const debouncedSearch = useDebounceFn((value: string) => {
  console.log('搜索:', value)
  // 执行搜索逻辑
}, 500)

// 节流滚动
const handleScroll = useThrottleFn((e: Event) => {
  console.log('滚动位置:', (e.target as HTMLElement).scrollTop)
}, 100)
```

## 6. 组件缓存策略

### 6.1 使用 KeepAlive

```vue
<script setup lang="ts">
import { ref } from 'vue'

const currentTab = ref('TabA')
const tabs = ['TabA', 'TabB', 'TabC']

// 缓存的组件列表
const cachedTabs = ref(['TabA', 'TabB'])

// 最大缓存数量
const maxCache = 3
</script>

<template>
  <div>
    <button 
      v-for="tab in tabs" 
      :key="tab"
      @click="currentTab = tab"
    >
      {{ tab }}
    </button>

    <KeepAlive :include="cachedTabs" :max="maxCache">
      <component :is="currentTab" />
    </KeepAlive>
  </div>
</template>
```

### 6.2 动态控制缓存

```vue
<script setup lang="ts">
import { ref, watch } from 'vue'

const currentView = ref('Home')
const cachedViews = ref<string[]>(['Home'])

// 根据条件动态添加/移除缓存
watch(currentView, (newView) => {
  if (!cachedViews.value.includes(newView)) {
    cachedViews.value.push(newView)
  }
  
  // 限制缓存数量
  if (cachedViews.value.length > 5) {
    cachedViews.value.shift()
  }
})

// 清除特定缓存
const removeCache = (viewName: string) => {
  const index = cachedViews.value.indexOf(viewName)
  if (index > -1) {
    cachedViews.value.splice(index, 1)
  }
}
</script>
```

## 7. 构建优化

### 7.1 Vite 配置优化

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    vue(),
    // 打包分析
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true
    })
  ],
  build: {
    // 代码分割策略
    rollupOptions: {
      output: {
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          'ui-vendor': ['element-plus'],
          'utils': ['lodash-es', 'dayjs']
        }
      }
    },
    // 生成 sourcemap
    sourcemap: false,
    // 压缩选项
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    },
    // chunk 大小警告限制
    chunkSizeWarningLimit: 1000
  },
  // 优化依赖预构建
  optimizeDeps: {
    include: ['vue', 'vue-router', 'pinia'],
    exclude: ['@vueuse/core']
  }
})
```

### 7.2 按需导入

```typescript
// 使用 unplugin-vue-components 实现组件自动导入
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  plugins: [
    Components({
      resolvers: [ElementPlusResolver()],
      dts: true
    })
  ]
})
```

## 8. 图片优化

### 8.1 图片懒加载

```vue
<script setup lang="ts">
import { useIntersectionObserver } from '@vueuse/core'

const imageRef = ref<HTMLImageElement>()
const isVisible = ref(false)

useIntersectionObserver(
  imageRef,
  ([{ isIntersecting }]) => {
    if (isIntersecting) {
      isVisible.value = true
    }
  }
)
</script>

<template>
  <img
    ref="imageRef"
    :src="isVisible ? actualSrc : placeholderSrc"
    alt="lazy loaded image"
  />
</template>
```

### 8.2 响应式图片

```vue
<template>
  <picture>
    <source
      media="(max-width: 768px)"
      :srcset="mobileSrc"
    />
    <source
      media="(max-width: 1200px)"
      :srcset="tabletSrc"
    />
    <img :src="desktopSrc" alt="responsive image" />
  </picture>
</template>
```

## 9. 网络请求优化

### 9.1 请求合并

```typescript
import { ref } from 'vue'

class RequestBatcher {
  private queue: Array<{
    id: string
    resolve: (value: any) => void
    reject: (reason: any) => void
  }> = []
  private timer: number | null = null

  add(id: string): Promise<any> {
    return new Promise((resolve, reject) => {
      this.queue.push({ id, resolve, reject })
      
      if (!this.timer) {
        this.timer = window.setTimeout(() => {
          this.flush()
        }, 50)
      }
    })
  }

  private async flush() {
    const items = [...this.queue]
    this.queue = []
    this.timer = null

    try {
      const ids = items.map(item => item.id)
      const results = await fetchBatch(ids)
      
      items.forEach((item, index) => {
        item.resolve(results[index])
      })
    } catch (error) {
      items.forEach(item => {
        item.reject(error)
      })
    }
  }
}

const batcher = new RequestBatcher()

// 使用
const data = await batcher.add('user-123')
```

### 9.2 请求缓存

```typescript
import { ref, reactive } from 'vue'

interface CacheItem<T> {
  data: T
  timestamp: number
}

class RequestCache {
  private cache = reactive(new Map<string, CacheItem<any>>())
  private maxAge = 5 * 60 * 1000 // 5分钟

  async get<T>(
    key: string,
    fetcher: () => Promise<T>,
    maxAge: number = this.maxAge
  ): Promise<T> {
    const cached = this.cache.get(key)
    
    if (cached && Date.now() - cached.timestamp < maxAge) {
      return cached.data
    }

    const data = await fetcher()
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    })

    return data
  }

  clear(key?: string) {
    if (key) {
      this.cache.delete(key)
    } else {
      