
---
title: Vue3 SSR 服务端渲染完全指南
published: 2024-01-17
description: 深入学习Vue3服务端渲染技术,掌握SSR项目搭建、数据预取、性能优化等核心技能
tags: [Vue3, SSR, 服务端渲染, 进阶]
category: 前端开发
draft: true
---

服务端渲染(SSR)可以提升首屏加载速度和SEO效果。本文将全面介绍Vue3中实现SSR的方法和最佳实践。

## 1. SSR 基础概念

### 1.1 什么是SSR

SSR(Server-Side Rendering)是在服务器端将Vue组件渲染成HTML字符串,然后发送给浏览器。

**优势:**
- 更好的SEO(搜索引擎优化)
- 更快的首屏加载速度
- 更好的用户体验

**劣势:**
- 服务器负载增加
- 开发复杂度提高
- 部分浏览器API不可用

### 1.2 CSR vs SSR vs SSG

```typescript
// CSR (Client-Side Rendering) - 客户端渲染
// 浏览器下载JS -> 执行JS -> 渲染页面

// SSR (Server-Side Rendering) - 服务端渲染  
// 服务器渲染HTML -> 浏览器显示 -> 激活(Hydration)

// SSG (Static Site Generation) - 静态站点生成
// 构建时生成所有HTML -> 直接部署静态文件
```

## 2. Vue3 SSR 项目搭建

### 2.1 基础项目结构

```
vue3-ssr-app/
├── src/
│   ├── components/
│   ├── pages/
│   ├── router/
│   ├── store/
│   ├── app.vue
│   ├── entry-client.ts    # 客户端入口
│   ├── entry-server.ts    # 服务端入口
│   └── main.ts            # 通用入口
├── server.js              # Node服务器
├── index.html
├── package.json
└── vite.config.ts
```

### 2.2 安装依赖

```bash
npm install vue vue-router pinia
npm install -D vite @vitejs/plugin-vue
npm install express compression serve-static
npm install -D @types/express @types/compression
```

### 2.3 创建通用应用入口

```typescript
// src/main.ts
import { createSSRApp } from 'vue'
import { createRouter } from './router'
import { createStore } from './store'
import App from './app.vue'

export function createApp() {
  const app = createSSRApp(App)
  const router = createRouter()
  const store = createStore()

  app.use(router)
  app.use(store)

  return { app, router, store }
}
```

### 2.4 客户端入口

```typescript
// src/entry-client.ts
import { createApp } from './main'

const { app, router, store } = createApp()

// 等待路由准备好
router.isReady().then(() => {
  // 从服务器注入的状态恢复store
  if (window.__INITIAL_STATE__) {
    store.state.value = window.__INITIAL_STATE__
  }

  app.mount('#app')
})
```

### 2.5 服务端入口

```typescript
// src/entry-server.ts
import { renderToString } from 'vue/server-renderer'
import { createApp } from './main'

export async function render(url: string, manifest: any) {
  const { app, router, store } = createApp()

  // 设置服务器端路由位置
  await router.push(url)
  await router.isReady()

  // 获取当前路由组件
  const matchedComponents = router.currentRoute.value.matched
  
  // 调用组件的数据预取方法
  await Promise.all(
    matchedComponents.map(async (component) => {
      if (component.components?.default?.serverPrefetch) {
        await component.components.default.serverPrefetch({
          store,
          route: router.currentRoute.value
        })
      }
    })
  )

  // 渲染应用
  const html = await renderToString(app)
  
  // 序列化状态
  const state = store.state.value

  return { html, state }
}
```

## 3. Vite 配置

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  build: {
    rollupOptions: {
      input: {
        client: resolve(__dirname, 'src/entry-client.ts'),
        server: resolve(__dirname, 'src/entry-server.ts')
      }
    }
  },
  ssr: {
    noExternal: ['pinia']
  }
})
```

## 4. Express 服务器搭建

```typescript
// server.js
import express from 'express'
import compression from 'compression'
import serveStatic from 'serve-static'
import { readFileSync } from 'fs'
import { resolve } from 'path'

const app = express()
const isProd = process.env.NODE_ENV === 'production'
const port = process.env.PORT || 3000

// 压缩响应
app.use(compression())

let vite, render, manifest

if (!isProd) {
  // 开发环境
  const { createServer } = await import('vite')
  vite = await createServer({
    server: { middlewareMode: true },
    appType: 'custom'
  })
  app.use(vite.middlewares)
} else {
  // 生产环境
  app.use(serveStatic(resolve('dist/client'), { index: false }))
  manifest = JSON.parse(
    readFileSync(resolve('dist/client/ssr-manifest.json'), 'utf-8')
  )
}

app.use('*', async (req, res) => {
  try {
    const url = req.originalUrl

    let template, render
    
    if (!isProd) {
      // 开发环境:动态读取
      template = readFileSync(resolve('index.html'), 'utf-8')
      template = await vite.transformIndexHtml(url, template)
      render = (await vite.ssrLoadModule('/src/entry-server.ts')).render
    } else {
      // 生产环境:使用构建的文件
      template = readFileSync(resolve('dist/client/index.html'), 'utf-8')
      render = (await import('./dist/server/entry-server.js')).render
    }

    // 渲染应用
    const { html, state } = await render(url, manifest)

    // 注入渲染的应用HTML和状态
    const finalHtml = template
      .replace('<!--app-html-->', html)
      .replace(
        '<!--app-state-->',
        `<script>window.__INITIAL_STATE__=${JSON.stringify(state)}</script>`
      )

    res.status(200).set({ 'Content-Type': 'text/html' }).end(finalHtml)
  } catch (e) {
    !isProd && vite.ssrFixStacktrace(e)
    console.error(e.stack)
    res.status(500).end(e.stack)
  }
})

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`)
})
```

## 5. 数据预取

### 5.1 组件级数据预取

```vue
<script setup lang="ts">
import { ref, onServerPrefetch } from 'vue'
import { useRoute } from 'vue-router'
import { useStore } from '@/store'

const route = useRoute()
const store = useStore()
const data = ref(null)

// 服务端预取数据
onServerPrefetch(async () => {
  const id = route.params.id
  data.value = await fetchData(id)
  // 将数据存入store
  store.setData(data.value)
})

// 客户端获取数据(如果服务端没有预取)
onMounted(async () => {
  if (!data.value) {
    const id = route.params.id
    data.value = await fetchData(id)
  }
})

async function fetchData(id: string) {
  const response = await fetch(`/api/data/${id}`)
  return response.json()
}
</script>

<template>
  <div v-if="data">
    <h1>{{ data.title }}</h1>
    <p>{{ data.content }}</p>
  </div>
  <div v-else>Loading...</div>
</template>
```

### 5.2 路由级数据预取

```typescript
// router/index.ts
import { createRouter as _createRouter, createMemoryHistory, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/user/:id',
    component: () => import('@/pages/User.vue'),
    meta: {
      // 定义服务端预取函数
      async serverPrefetch(to, store) {
        const userId = to.params.id
        await store.dispatch('fetchUser', userId)
      }
    }
  }
]

export function createRouter() {
  return _createRouter({
    history: import.meta.env.SSR 
      ? createMemoryHistory() 
      : createWebHistory(),
    routes
  })
}
```

### 5.3 Store 数据预取

```typescript
// store/index.ts
import { createPinia, defineStore } from 'pinia'

export function createStore() {
  const pinia = createPinia()
  return pinia
}

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    posts: []
  }),
  
  actions: {
    async fetchUser(id: string) {
      if (import.meta.env.SSR) {
        // 服务端直接调用API
        this.user = await fetchUserFromDB(id)
      } else {
        // 客户端通过HTTP请求
        const response = await fetch(`/api/users/${id}`)
        this.user = await response.json()
      }
    },
    
    async fetchPosts(userId: string) {
      const response = await fetch(`/api/users/${userId}/posts`)
      this.posts = await response.json()
    }
  }
})
```

## 6. SEO 优化

### 6.1 动态Meta标签

```vue
<script setup lang="ts">
import { useHead } from '@vueuse/head'
import { computed } from 'vue'

const props = defineProps<{
  article: {
    title: string
    description: string
    image: string
    author: string
  }
}>()

useHead({
  title: computed(() => props.article.title),
  meta: [
    {
      name: 'description',
      content: computed(() => props.article.description)
    },
    {
      property: 'og:title',
      content: computed(() => props.article.title)
    },
    {
      property: 'og:description',
      content: computed(() => props.article.description)
    },
    {
      property: 'og:image',
      content: computed(() => props.article.image)
    },
    {
      name: 'twitter:card',
      content: 'summary_large_image'
    },
    {
      name: 'author',
      content: computed(() => props.article.author)
    }
  ]
})
</script>
```

### 6.2 结构化数据

```vue
<script setup lang="ts">
import { useHead } from '@vueuse/head'

const article = {
  title: 'Vue3 SSR Guide',
  datePublished: '2024-01-17',
  author: 'John Doe'
}

useHead({
  script: [
    {
      type: 'application/ld+json',
      children: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'Article',
        headline: article.title,
        datePublished: article.datePublished,
        author: {
          '@type': 'Person',
          name: article.author
        }
      })
    }
  ]
})
</script>
```

## 7. 性能优化

### 7.1 页面级代码分割

```typescript
// router/index.ts
const routes = [
  {
    path: '/',
    component: () => import('@/pages/Home.vue')
  },
  {
    path: '/about',
    component: () => import('@/pages/About.vue')
  }
]
```

### 7.2 组件级缓存

```typescript
// server.js
import LRU from 'lru-cache'

const cache = new LRU({
  max: 100,
  ttl: 1000 * 60 * 5 // 5分钟
})

app.use('*', async (req, res) => {
  const url = req.originalUrl
  
  // 检查缓存
  const cached = cache.get(url)
  if (cached) {
    return res.status(200).set({ 'Content-Type': 'text/html' }).end(cached)
  }

  // 渲染并缓存
  const html = await render(url)
  cache.set(url, html)
  
  res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
})
```

### 7.3 流式渲染

```typescript
// src/entry-server.ts
import { renderToNodeStream } from 'vue/server-renderer'

export async function renderStream(url: string) {
  const { app, router } = createApp()
  
  await router.push(url)
  await router.isReady()
  
  return renderToNodeStream(app)
}

// server.js
app.use('*', async (req, res) => {
  const stream = await renderStream(req.originalUrl)
  
  res.setHeader('Content-Type', 'text/html')
  
  // 发送头部
  res.write(templateStart)
  
  // 流式发送内容
  stream.pipe(res, { end: false })
  
  stream.on('end', () => {
    res.end(templateEnd)
  })
})
```

## 8. 常见问题处理

### 8.1 处理浏览器专用API

```typescript
// utils/client-only.ts
export function isClient() {
  return typeof window !== 'undefined'
}

// 使用示例
if (isClient()) {
  window.localStorage.setItem('key', 'value')
}
```

```vue
<script setup lang="ts">
import { onMounted } from 'vue'

// 只在客户端执行
onMounted(() => {
  // 安全使用浏览器API
  console.log(window.innerWidth)
})
</script>
```

### 8.2 第三方库兼容

```typescript
// plugins/external-lib.ts
import { Plugin } from 'vue'

export default {
  install(app) {
    if (typeof window !== 'undefined') {
      // 只在客户端加载
      import('some-browser-lib').then(lib => {
        app.config.globalProperties.$lib = lib
      })
    }
  }
} as Plugin
```

### 8.3 Hydration不匹配

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'

// ❌ 错误:服务端和客户端渲染不一致
const timestamp = ref(Date.now())

// ✅ 正确:使用 ClientOnly 组件
const timestamp = ref(0)
onMounted(() => {
  timestamp.value = Date.now()
})
</script>

<template>
  <ClientOnly>
    <div>{{ timestamp }}</div>
  </ClientOnly>
</template>
```

## 9. 部署

### 9.1 构建脚本

```json
{
  "scripts": {
    "dev": "node server.js",
    "build:client": "vite build --outDir dist/client --ssrManifest",
    "build:server": "vite build --outDir dist/server --ssr src/entry-server.ts",
    "build": "npm run build:client && npm run build:server",
    "preview": "NODE_ENV=production node server.js"
  }
}
```

### 9.2 Docker部署

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist
COPY server.js ./

EXPOSE 3000

CMD ["node", "server.js"]
```

### 9.3 PM2部署

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'vue-ssr-app',
    script: './server.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
}
```

## 10. 最佳实践

### 10.1 