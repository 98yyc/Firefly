
---
title: Vue2 性能优化实战技巧
published: 2024-01-19
description: 深入学习Vue2性能优化方法,掌握函数式组件、keep-alive、懒加载等优化技术
tags: [Vue2, 性能优化, 前端, 进阶]
category: 前端开发
draft: true
---

Vue2虽然不如Vue3性能强劲,但通过合理的优化手段,依然可以构建高性能的应用。本文将介绍Vue2中的各种性能优化技巧。

## 1. 组件优化

### 1.1 函数式组件

函数式组件没有实例,没有响应式数据,渲染开销更小,适合纯展示型组件。

```vue
<!-- 普通组件 -->
<template>
  <div class="item">
    <span>{{ item.name }}</span>
    <span>{{ item.price }}</span>
  </div>
</template>

<script>
export default {
  name: 'ListItem',
  props: ['item']
}
</script>
```

```vue
<!-- 函数式组件 -->
<template functional>
  <div class="item">
    <span>{{ props.item.name }}</span>
    <span>{{ props.item.price }}</span>
  </div>
</template>

<script>
export default {
  name: 'ListItem',
  functional: true,
  props: ['item']
}
</script>
```

使用render函数:

```javascript
export default {
  name: 'FunctionalButton',
  functional: true,
  props: {
    type: String,
    text: String
  },
  render(h, context) {
    const { props, data, children } = context
    
    return h(
      'button',
      {
        class: ['btn', `btn-${props.type}`],
        on: data.on
      },
      props.text || children
    )
  }
}
```

### 1.2 使用 Object.freeze()

对于不需要响应式的大数据,使用 `Object.freeze()` 冻结对象:

```javascript
export default {
  data() {
    return {
      // 大量静态数据
      list: Object.freeze([
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        // ... 更多数据
      ])
    }
  },
  
  created() {
    // 从API获取的大数据
    this.$http.get('/api/data').then(res => {
      this.list = Object.freeze(res.data)
    })
  }
}
```

### 1.3 v-show vs v-if

```vue
<template>
  <div>
    <!-- 频繁切换使用 v-show -->
    <div v-show="isVisible">
      频繁切换的内容
    </div>

    <!-- 条件很少改变使用 v-if -->
    <div v-if="isAdmin">
      管理员才能看到的内容
    </div>

    <!-- 大组件延迟渲染 -->
    <heavy-component v-if="shouldRender" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      isVisible: true,
      isAdmin: false,
      shouldRender: false
    }
  },
  
  mounted() {
    // 延迟渲染重型组件
    this.$nextTick(() => {
      this.shouldRender = true
    })
  }
}
</script>
```

### 1.4 计算属性缓存

```javascript
export default {
  data() {
    return {
      items: [1, 2, 3, 4, 5]
    }
  },
  
  computed: {
    // ✅ 使用计算属性,有缓存
    filteredItems() {
      console.log('计算 filteredItems')
      return this.items.filter(item => item > 2)
    }
  },
  
  methods: {
    // ❌ 使用方法,每次都重新计算
    getFilteredItems() {
      console.log('计算 getFilteredItems')
      return this.items.filter(item => item > 2)
    }
  }
}
```

## 2. 列表渲染优化

### 2.1 key的正确使用

```vue
<template>
  <div>
    <!-- ❌ 不要使用index作为key -->
    <div v-for="(item, index) in list" :key="index">
      {{ item.name }}
    </div>

    <!-- ✅ 使用唯一ID作为key -->
    <div v-for="item in list" :key="item.id">
      {{ item.name }}
    </div>
  </div>
</template>
```

### 2.2 虚拟滚动

使用 `vue-virtual-scroller` 处理长列表:

```bash
npm install vue-virtual-scroller
```

```javascript
// main.js
import VueVirtualScroller from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

Vue.use(VueVirtualScroller)
```

```vue
<template>
  <RecycleScroller
    class="scroller"
    :items="items"
    :item-size="60"
    key-field="id"
    v-slot="{ item }"
  >
    <div class="item">
      {{ item.name }}
    </div>
  </RecycleScroller>
</template>

<script>
export default {
  data() {
    return {
      items: Array.from({ length: 10000 }, (_, i) => ({
        id: i,
        name: `Item ${i}`
      }))
    }
  }
}
</script>

<style scoped>
.scroller {
  height: 100vh;
}

.item {
  height: 60px;
  padding: 10px;
  border-bottom: 1px solid #eee;
}
</style>
```

### 2.3 分页加载

```vue
<template>
  <div>
    <div v-for="item in displayedItems" :key="item.id">
      {{ item.name }}
    </div>
    
    <button @click="loadMore" v-if="hasMore">
      加载更多
    </button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [],
      displayedItems: [],
      page: 1,
      pageSize: 20,
      hasMore: true
    }
  },
  
  created() {
    this.loadMore()
  },
  
  methods: {
    loadMore() {
      const start = (this.page - 1) * this.pageSize
      const end = start + this.pageSize
      const newItems = this.items.slice(start, end)
      
      this.displayedItems.push(...newItems)
      this.page++
      this.hasMore = end < this.items.length
    }
  }
}
</script>
```

## 3. keep-alive 缓存

### 3.1 基础使用

```vue
<template>
  <div>
    <keep-alive>
      <component :is="currentView" />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentView: 'Home'
    }
  }
}
</script>
```

### 3.2 条件缓存

```vue
<template>
  <div>
    <!-- 只缓存特定组件 -->
    <keep-alive include="Home,List">
      <router-view />
    </keep-alive>

    <!-- 排除特定组件 -->
    <keep-alive exclude="Detail">
      <router-view />
    </keep-alive>

    <!-- 动态控制 -->
    <keep-alive :include="cachedViews">
      <router-view />
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cachedViews: ['Home', 'List']
    }
  },
  
  methods {
    addCache(name) {
      if (!this.cachedViews.includes(name)) {
        this.cachedViews.push(name)
      }
    },
    
    removeCache(name) {
      const index = this.cachedViews.indexOf(name)
      if (index > -1) {
        this.cachedViews.splice(index, 1)
      }
    }
  }
}
</script>
```

### 3.3 生命周期钩子

```vue
<script>
export default {
  name: 'CachedComponent',
  
  activated() {
    console.log('组件被激活')
    // 刷新数据
    this.fetchData()
  },
  
  deactivated() {
    console.log('组件被缓存')
    // 清理定时器等
    if (this.timer) {
      clearInterval(this.timer)
    }
  }
}
</script>
```

## 4. 异步组件与懒加载

### 4.1 路由懒加载

```javascript
// router/index.js
const routes = [
  {
    path: '/home',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/about',
    component: () => import(/* webpackChunkName: "about" */ '@/views/About.vue')
  }
]
```

### 4.2 组件懒加载

```vue
<template>
  <div>
    <async-component v-if="shouldLoad" />
  </div>
</template>

<script>
export default {
  components: {
    AsyncComponent: () => import('./HeavyComponent.vue')
  },
  
  data() {
    return {
      shouldLoad: false
    }
  },
  
  mounted() {
    // 延迟加载
    setTimeout(() => {
      this.shouldLoad = true
    }, 1000)
  }
}
</script>
```

### 4.3 带加载状态的异步组件

```javascript
export default {
  components: {
    AsyncComponent: () => ({
      component: import('./HeavyComponent.vue'),
      loading: LoadingComponent,
      error: ErrorComponent,
      delay: 200,
      timeout: 3000
    })
  }
}
```

## 5. 事件优化

### 5.1 事件修饰符

```vue
<template>
  <div>
    <!-- 阻止默认行为 -->
    <form @submit.prevent="onSubmit">
      <input type="text">
    </form>

    <!-- 只触发一次 -->
    <button @click.once="init">初始化</button>

    <!-- 事件捕获 -->
    <div @click.capture="handler">
      <button>Click</button>
    </div>

    <!-- 只当事件在该元素本身触发时 -->
    <div @click.self="handler">
      <button>Click</button>
    </div>
  </div>
</template>
```

### 5.2 防抖和节流

```javascript
// utils/debounce.js
export function debounce(func, wait) {
  let timeout
  return function(...args) {
    clearTimeout(timeout)
    timeout = setTimeout(() => {
      func.apply(this, args)
    }, wait)
  }
}

export function throttle(func, wait) {
  let previous = 0
  return function(...args) {
    const now = Date.now()
    if (now - previous > wait) {
      func.apply(this, args)
      previous = now
    }
  }
}
```

```vue
<template>
  <div>
    <input @input="handleSearch" placeholder="搜索">
    <div @scroll="handleScroll" class="scroll-container">
      <!-- 内容 -->
    </div>
  </div>
</template>

<script>
import { debounce, throttle } from '@/utils/debounce'

export default {
  created() {
    this.handleSearch = debounce(this.search, 500)
    this.handleScroll = throttle(this.onScroll, 100)
  },
  
  methods: {
    search(e) {
      console.log('搜索:', e.target.value)
    },
    
    onScroll(e) {
      console.log('滚动:', e.target.scrollTop)
    }
  }
}
</script>
```

### 5.3 事件委托

```vue
<template>
  <ul @click="handleClick">
    <li v-for="item in items" :key="item.id" :data-id="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' }
      ]
    }
  },
  
  methods: {
    handleClick(e) {
      const target = e.target
      if (target.tagName === 'LI') {
        const id = target.dataset.id
        console.log('点击了:', id)
      }
    }
  }
}
</script>
```

## 6. 图片优化

### 6.1 图片懒加载

使用 `vue-lazyload`:

```bash
npm install vue-lazyload
```

```javascript
// main.js
import VueLazyload from 'vue-lazyload'

Vue.use(VueLazyload, {
  preLoad: 1.3,
  error: require('@/assets/error.png'),
  loading: require('@/assets/loading.gif'),
  attempt: 1
})
```

```vue
<template>
  <div>
    <!-- 使用 v-lazy 指令 -->
    <img v-lazy="imageSrc" alt="图片">
    
    <!-- 背景图 -->
    <div v-lazy:background-image="imageSrc" class="bg"></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      imageSrc: '/path/to/image.jpg'
    }
  }
}
</script>
```

### 6.2 响应式图片

```vue
<template>
  <picture>
    <source
      media="(max-width: 768px)"
      :srcset="mobileImage"
    >
    <source
      media="(max-width: 1200px)"
      :srcset="tabletImage"
    >
    <img :src="desktopImage" alt="响应式图片">
  </picture>
</template>
```

### 6.3 渐进式图片

```vue
<template>
  <div class="progressive-image">
    <img
      :src="placeholder"
      :data-src="original"
      @load="onLoad"
      class="image"
      :class="{ loaded: isLoaded }"
    >
  </div>
</template>

<script>
export default {
  props: {
    placeholder: String,
    original: String
  },
  
  data() {
    return {
      isLoaded: false
    }
  },
  
  mounted() {
    const img = new Image()
    img.src = this.original
    img.onload = () => {
      this.$el.querySelector('.image').src = this.original
      this.isLoaded = true
    }
  },
  
  methods: {
    onLoad() {
      this.isLoaded = true
    }
  }
}
</script>

<style scoped>
.image {
  filter: blur(10px);
  transition: filter 0.3s;
}

.image.loaded {
  filter: blur(0);
}
</style>
```

## 7. 打包优化

### 7.1 代码分割

```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.optimization.splitChunks({
      chunks: 'all',
      cacheGroups: {
        libs: {
          name: 'chunk-libs',
          test: /[\\/]node_modules[\\/]/,
          priority: 10,
          chunks: 'initial'
        },
        elementUI: {
          name: 'chunk-elementUI',
          priority: 20,
          test: /[\\/]node_modules[\\/]_?element-ui(.*)/
        },
        commons: {
          name: 'chunk-commons',
          test: resolve('src/components'),
          minChunks: 3,
          priority: 5,
          reuseExistingChunk: true
        }
      }
    })
  }
}
```

### 7.2 gzip压缩

```javascript
// vue.config.js
const CompressionPlugin = require('compression-webpack-plugin')

module.exports = {
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      config.plugins.push(
        new CompressionPlugin({
          algorithm: 'gzip',
          test: /\.(js|css|html|svg)$/,
          threshold: 10240,
          minRatio: 0.8
        })
      )
    }
  }
}
```

### 7.3 CDN加速

```javascript
// vue.config.js
const externals = {
  vue: 'Vue',
  'vue-router': 'VueRouter',
  vuex: 'Vuex',
  axios: 'axios',
  'element-ui': 'ELEMENT'
}

module.exports = {
  configureWebpack: {
    externals: process.env.NODE_ENV === 'production' ? externals : {}
  }
}
```

```html
<!-- public/index.html -->
<% if (process.env.NODE_ENV === 'production') { %>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/vue-router@3.5.3/dist/vue-router.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/vuex@3.6.2/dist/vuex.min.js"></script>
<% } %>
```

## 8. 运行时优化

### 8.1 减少不必要的更新

```javascript
export default {
  data() {
    return {
      user: {
        name: 'John',
        age: 30
      }
    }
  },
  
  methods: {
    // ❌ 触发整个对象更新
    updateName() {
      this.user = { ...this.user, name: 'Jane' }
    },
    
    // ✅ 只更新需要的属性
    updateNameOptimized() {
      this.$set(this.user, 'name', 'Jane')
      // 或
      this.user.name = 'Jane'
    }
  }
}
```

### 8.2 使用 v-once

```vue
<template>
  <div>
    <!-- 