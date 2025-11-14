---
title: Vue3入门教程(三) - Vite项目构建
published: 2024-01-09
description: 学习使用Vite构建Vue3项目,掌握现代化的前端开发工具链和最佳实践
tags: [Vue3, Vite, 前端, 构建工具, 工程化]
category: 前端开发
draft: false
---

# Vue3入门教程(三) - Vite项目构建

Vite是Vue官方推荐的下一代前端构建工具,它利用浏览器原生ES模块特性,提供了极快的开发体验。本文将详细介绍如何使用Vite构建Vue3项目。

## Vite简介

### 什么是Vite?

Vite(法语"快速"的意思)是由Vue作者尤雨溪开发的新一代前端构建工具。

**核心特点**:
- ⚡ 极速的开发服务器启动
- 🔥 热模块替换(HMR)速度极快
- 📦 优化的生产构建
- 🎯 开箱即用的TypeScript支持
- 🔧 丰富的插件生态

### Vite vs Webpack

| 特性 | Vite | Webpack |
|------|------|---------|
| 开发服务器启动 | 毫秒级 | 秒级 |
| 热更新速度 | 快速 | 较慢 |
| 生产构建 | Rollup | Webpack |
| 配置复杂度 | 简单 | 复杂 |
| 学习曲线 | 平缓 | 陡峭 |

### Vite工作原理

**开发环境**:
```
浏览器请求 → Vite服务器 → 按需编译 → 返回ES模块
```

**生产环境**:
```
源代码 → Rollup打包 → 优化产物 → 部署
```

## 创建Vite项目

### 方式一: npm create

```bash
# 使用npm
npm create vite@latest my-vue-app

# 使用yarn
yarn create vite my-vue-app

# 使用pnpm
pnpm create vite my-vue-app
```

### 方式二: 指定模板

```bash
# Vue + JavaScript
npm create vite@latest my-vue-app -- --template vue

# Vue + TypeScript
npm create vite@latest my-vue-app -- --template vue-ts
```

### 初始化步骤

```bash
# 1. 创建项目
npm create vite@latest my-vue-app -- --template vue

# 2. 进入项目目录
cd my-vue-app

# 3. 安装依赖
npm install

# 4. 启动开发服务器
npm run dev
```

## 项目结构

```
my-vue-app/
├── node_modules/           # 依赖包
├── public/                 # 静态资源
│   └── favicon.ico
├── src/
│   ├── assets/            # 资源文件
│   │   └── logo.png
│   ├── components/        # 组件
│   │   └── HelloWorld.vue
│   ├── App.vue           # 根组件
│   ├── main.js           # 入口文件
│   └── style.css         # 全局样式
├── .gitignore
├── index.html            # HTML模板
├── package.json          # 项目配置
├── README.md
└── vite.config.js        # Vite配置
```

### 关键文件说明

#### index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vue3 + Vite App</title>
</head>
<body>
    <div id="app"></div>
    <!-- 注意: 直接引用JS文件 -->
    <script type="module" src="/src/main.js"></script>
</body>
</html>
```

#### main.js

```js
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

createApp(App).mount('#app')
```

#### package.json

```json
{
  "name": "my-vue-app",
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.3.4"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.3.4",
    "vite": "^4.4.9"
  }
}
```

## Vite配置

### 基础配置

```js
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  
  // 开发服务器配置
  server: {
    port: 3000,           // 端口
    open: true,           // 自动打开浏览器
    cors: true,           // 允许跨域
    
    // 代理配置
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  
  // 构建配置
  build: {
    outDir: 'dist',       // 输出目录
    assetsDir: 'assets',  // 静态资源目录
    minify: 'terser',     // 压缩器
    sourcemap: false,     // 是否生成sourcemap
    
    // Rollup配置
    rollupOptions: {
      output: {
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia']
        }
      }
    }
  }
})
```

### 路径别名配置

```js
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@api': resolve(__dirname, 'src/api'),
      '@assets': resolve(__dirname, 'src/assets')
    }
  }
})
```

使用别名:

```js
// 之前
import HelloWorld from '../../../components/HelloWorld.vue'

// 现在
import HelloWorld from '@components/HelloWorld.vue'
```

### 环境变量配置

创建环境变量文件:

```bash
# .env.development (开发环境)
VITE_APP_TITLE=开发环境
VITE_API_BASE_URL=http://localhost:8080/api
VITE_APP_PORT=3000

# .env.production (生产环境)
VITE_APP_TITLE=生产环境
VITE_API_BASE_URL=https://api.example.com
VITE_APP_PORT=8080

# .env.local (本地环境,不提交到git)
VITE_SECRET_KEY=your-secret-key
```

使用环境变量:

```js
// 在代码中使用
console.log(import.meta.env.VITE_APP_TITLE)
console.log(import.meta.env.VITE_API_BASE_URL)

// 在vite.config.js中使用
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())
  
  return {
    server: {
      port: env.VITE_APP_PORT
    }
  }
})
```

### CSS预处理器配置

```bash
# 安装Sass
npm install -D sass

# 安装Less
npm install -D less

# 安装Stylus
npm install -D stylus
```

使用:

```vue
<template>
  <div class="container">
    <h1>Hello Vite</h1>
  </div>
</template>

<style lang="scss" scoped>
$primary-color: #42b883;

.container {
  h1 {
    color: $primary-color;
  }
}
</style>
```

配置全局样式变量:

```js
// vite.config.js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    }
  }
})
```

## 常用插件

### 1. 自动导入

```bash
npm install -D unplugin-auto-import unplugin-vue-components
```

```js
// vite.config.js
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'

export default defineConfig({
  plugins: [
    vue(),
    
    // 自动导入Vue相关函数
    AutoImport({
      imports: ['vue', 'vue-router', 'pinia'],
      dts: 'src/auto-imports.d.ts'
    }),
    
    // 自动导入组件
    Components({
      dts: 'src/components.d.ts'
    })
  ]
})
```

使用效果:

```vue
<script setup>
// 不需要手动import
// import { ref, computed, onMounted } from 'vue'

// 直接使用
const count = ref(0)
const double = computed(() => count.value * 2)

onMounted(() => {
  console.log('mounted')
})
</script>
```

### 2. SVG图标

```bash
npm install -D vite-plugin-svg-icons
```

```js
// vite.config.js
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'
import { resolve } from 'path'

export default defineConfig({
  plugins: [
    vue(),
    createSvgIconsPlugin({
      iconDirs: [resolve(process.cwd(), 'src/assets/icons')],
      symbolId: 'icon-[dir]-[name]'
    })
  ]
})
```

```js
// main.js
import 'virtual:svg-icons-register'
```

使用SVG:

```vue
<template>
  <svg class="icon">
    <use xlink:href="#icon-user"></use>
  </svg>
</template>
```

### 3. 压缩插件

```bash
npm install -D vite-plugin-compression
```

```js
// vite.config.js
import compression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    vue(),
    compression({
      algorithm: 'gzip',
      ext: '.gz'
    })
  ]
})
```

### 4. 可视化分析

```bash
npm install -D rollup-plugin-visualizer
```

```js
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    vue(),
    visualizer({
      open: true,  // 构建完成后自动打开
      gzipSize: true,
      brotliSize: true
    })
  ]
})
```

## 实战项目结构

### 完整的项目结构

```
my-vue-app/
├── public/                    # 静态资源
│   ├── favicon.ico
│   └── images/
├── src/
│   ├── api/                   # API接口
│   │   ├── index.js
│   │   └── user.js
│   ├── assets/                # 资源文件
│   │   ├── images/
│   │   ├── icons/
│   │   └── styles/
│   │       ├── variables.scss
│   │       ├── mixins.scss
│   │       └── global.scss
│   ├── components/            # 公共组件
│   │   ├── common/
│   │   │   ├── Button.vue
│   │   │   └── Input.vue
│   │   └── layout/
│   │       ├── Header.vue
│   │       ├── Footer.vue
│   │       └── Sidebar.vue
│   ├── composables/           # 组合式函数
│   │   ├── useMouse.js
│   │   └── useFetch.js
│   ├── directives/            # 自定义指令
│   │   └── vFocus.js
│   ├── plugins/               # 插件
│   │   └── axios.js
│   ├── router/                # 路由
│   │   └── index.js
│   ├── stores/                # 状态管理
│   │   ├── index.js
│   │   └── user.js
│   ├── utils/                 # 工具函数
│   │   ├── request.js
│   │   └── storage.js
│   ├── views/                 # 页面组件
│   │   ├── Home.vue
│   │   ├── About.vue
│   │   └── User/
│   │       ├── List.vue
│   │       └── Detail.vue
│   ├── App.vue
│   └── main.js
├── .env.development
├── .env.production
├── .gitignore
├── index.html
├── package.json
├── README.md
└── vite.config.js
```

### API封装

```js
// src/utils/request.js
import axios from 'axios'

const request = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 5000
})

// 请求拦截器
request.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token')
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

// 响应拦截器
request.interceptors.response.use(
  response => {
    const { data } = response
    return data
  },
  error => {
    console.error('请求错误:', error)
    return Promise.reject(error)
  }
)

export default request
```

```js
// src/api/user.js
import request from '@/utils/request'

export const getUserList = () => {
  return request.get('/users')
}

export const getUserById = (id) => {
  return request.get(`/users/${id}`)
}

export const createUser = (data) => {
  return request.post('/users', data)
}

export const updateUser = (id, data) => {
  return request.put(`/users/${id}`, data)
}

export const deleteUser = (id) => {
  return request.delete(`/users/${id}`)
}
```

### 路由配置

```js
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue')
  },
  {
    path: '/user',
    name: 'User',
    component: () => import('@/views/User/List.vue'),
    meta: { requiresAuth: true }
  },
  {
    path: '/user/:id',
    name: 'UserDetail',
    component: () => import('@/views/User/Detail.vue'),
    meta: { requiresAuth: true }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 路由守卫
router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token')
  
  if (to.meta.requiresAuth && !token) {
    next('/login')
  } else {
    next()
  }
})

export default router
```

```js
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import '@/assets/styles/global.scss'

const app = createApp(App)

app.use(router)
app.mount('#app')
```

## 性能优化

### 1. 路由懒加载

```js
// 使用动态import
const routes = [
  {
    path: '/home',
    component: () => import('@/views/Home.vue')
  }
]
```

### 2. 组件懒加载

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

// 异步组件
const AsyncComponent = defineAsyncComponent(() =>
  import('@/components/HeavyComponent.vue')
)
</script>
```

### 3. 代码分割

```js
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // 将Vue相关库打包到一起
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          // UI组件库单独打包
          'ui-vendor': ['element-plus'],
          // 工具库单独打包
          'utils-vendor': ['axios', 'dayjs']
        }
      }
    }
  }
})
```

### 4. 图片优化

```bash
npm install -D vite-plugin-imagemin
```

```js
// vite.config.js
import viteImagemin from 'vite-plugin-imagemin'

export default defineConfig({
  plugins: [
    vue(),
    viteImagemin({
      gifsicle: {
        optimizationLevel: 7
      },
      optipng: {
        optimizationLevel: 7
      },
      mozjpeg: {
        quality: 80
      },
      pngquant: {
        quality: [0.8, 0.9],
        speed: 4
      },
      svgo: {
        plugins: [
          { name: 'removeViewBox', active: false }
        ]
      }
    })
  ]
})
```

### 5. CDN加速

```js
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      external: ['vue', 
'vue-router'],
      output: {
        globals: {
          vue: 'Vue',
          'vue-router': 'VueRouter'
        }
      }
    }
  }
})
```

```html
<!-- index.html -->
<head>
  <!-- 生产环境使用CDN -->
  <script src="https://unpkg.com/vue@3/dist/vue.global.prod.js"></script>
  <script src="https://unpkg.com/vue-router@4/dist/vue-router.global.prod.js"></script>
</head>
```

### 6. Gzip压缩

```js
// vite.config.js
import viteCompression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    vue(),
    viteCompression({
      verbose: true,
      disable: false,
      threshold: 10240,  // 大于10kb的文件才压缩
      algorithm: 'gzip',
      ext: '.gz'
    })
  ]
})
```

## 部署

### 构建生产版本

```bash
# 构建
npm run build

# 预览构建结果
npm run preview
```

### 部署到Nginx

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/my-vue-app/dist;
    index index.html;

    # 启用gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # SPA路由配置
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 部署到Vercel

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "vite",
  "rewrites": [
    {
      "source": "/(.*)",
      "destination": "/index.html"
    }
  ]
}
```

### 部署到GitHub Pages

```bash
# 安装gh-pages
npm install -D gh-pages
```

```json
// package.json
{
  "scripts": {
    "deploy": "npm run build && gh-pages -d dist"
  }
}
```

```js
// vite.config.js
export default defineConfig({
  base: '/your-repo-name/',  // GitHub仓库名
  plugins: [vue()]
})
```

## 开发技巧

### 1. 热更新配置

```js
// vite.config.js
export default defineConfig({
  server: {
    hmr: {
      overlay: true  // 显示错误覆盖层
    }
  }
})
```

### 2. 调试配置

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Vite Debug",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/src",
      "sourceMaps": true
    }
  ]
}
```

### 3. ESLint配置

```bash
npm install -D eslint eslint-plugin-vue
```

```js
// .eslintrc.js
module.exports = {
  extends: [
    'plugin:vue/vue3-recommended'
  ],
  rules: {
    'vue/multi-word-component-names': 'off'
  }
}
```

```json
// package.json
{
  "scripts": {
    "lint": "eslint src --ext .vue,.js,.ts",
    "lint:fix": "eslint src --ext .vue,.js,.ts --fix"
  }
}
```

### 4. Prettier配置

```bash
npm install -D prettier eslint-config-prettier
```

```js
// .prettierrc.js
module.exports = {
  semi: false,
  singleQuote: true,
  printWidth: 80,
  trailingComma: 'none',
  arrowParens: 'avoid'
}
```

### 5. Git Hooks

```bash
npm install -D husky lint-staged
npx husky install
```

```json
// package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{js,vue}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

```bash
# 添加pre-commit钩子
npx husky add .husky/pre-commit "npx lint-staged"
```

## 常见问题

### 1. 导入别名不生效

**解决方案**:

```js
// vite.config.js
import { resolve } from 'path'

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, './src')
    }
  }
})
```

```json
// jsconfig.json 或 tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### 2. 环境变量获取不到

**问题**: `process.env.XXX` 不生效

**解决方案**: Vite使用 `import.meta.env`

```js
// ❌ 错误
console.log(process.env.VITE_API_URL)

// ✅ 正确
console.log(import.meta.env.VITE_API_URL)
```

### 3. 静态资源引用

```vue
<template>
  <!-- ✅ public目录下的文件 -->
  <img src="/logo.png" />
  
  <!-- ✅ assets目录下的文件 -->
  <img :src="logoUrl" />
</template>

<script setup>
import logoUrl from '@/assets/logo.png'
// 或
const logoUrl = new URL('@/assets/logo.png', import.meta.url).href
</script>
```

### 4. 全局样式不生效

```js
// vite.config.js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        // 注意: 使用additionalData会自动注入到每个组件
        additionalData: `@import "@/styles/variables.scss";`
      }
    }
  }
})
```

```js
// main.js - 导入全局样式
import '@/styles/global.scss'
```

### 5. 第三方库报错

某些老旧的npm包可能不支持ES模块。

**解决方案**:

```js
// vite.config.js
export default defineConfig({
  optimizeDeps: {
    include: ['problematic-package']
  }
})
```

## 完整配置示例

```js
// vite.config.js
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { createSvgIconsPlugin } from 'vite-plugin-svg-icons'
import viteCompression from 'vite-plugin-compression'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())
  
  return {
    plugins: [
      vue(),
      
      // 自动导入
      AutoImport({
        imports: ['vue', 'vue-router', 'pinia'],
        dts: 'src/auto-imports.d.ts'
      }),
      
      // 组件自动注册
      Components({
        dts: 'src/components.d.ts'
      }),
      
      // SVG图标
      createSvgIconsPlugin({
        iconDirs: [resolve(process.cwd(), 'src/assets/icons')],
        symbolId: 'icon-[dir]-[name]'
      }),
      
      // Gzip压缩
      viteCompression({
        verbose: true,
        disable: false,
        threshold: 10240,
        algorithm: 'gzip',
        ext: '.gz'
      })
    ],
    
    // 路径解析
    resolve: {
      alias: {
        '@': resolve(__dirname, 'src'),
        '@components': resolve(__dirname, 'src/components'),
        '@utils': resolve(__dirname, 'src/utils'),
        '@api': resolve(__dirname, 'src/api'),
        '@assets': resolve(__dirname, 'src/assets')
      }
    },
    
    // CSS配置
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/styles/variables.scss";`
        }
      }
    },
    
    // 开发服务器
    server: {
      port: Number(env.VITE_APP_PORT) || 3000,
      open: true,
      cors: true,
      proxy: {
        '/api': {
          target: env.VITE_API_BASE_URL,
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        }
      }
    },
    
    // 构建配置
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      minify: 'terser',
      sourcemap: false,
      
      terserOptions: {
        compress: {
          drop_console: true,
          drop_debugger: true
        }
      },
      
      rollupOptions: {
        output: {
          chunkFileNames: 'js/[name]-[hash].js',
          entryFileNames: 'js/[name]-[hash].js',
          assetFileNames: '[ext]/[name]-[hash].[ext]',
          
          manualChunks: {
            'vue-vendor': ['vue', 'vue-router', 'pinia'],
            'utils-vendor': ['axios', 'dayjs']
          }
        }
      }
    },
    
    // 优化依赖
    optimizeDeps: {
      include: ['vue', 'vue-router', 'pinia', 'axios']
    }
  }
})
```

## 最佳实践总结

### 1. 项目结构规范

- ✅ 按功能模块组织代码
- ✅ 使用路径别名简化导入
- ✅ 分离配置文件(开发/生产)

### 2. 性能优化

- ✅ 路由和组件懒加载
- ✅ 合理的代码分割策略
- ✅ 启用Gzip压缩
- ✅ 图片优化和懒加载
- ✅ CDN加速静态资源

### 3. 开发效率

- ✅ 自动导入API和组件
- ✅ 配置ESLint和Prettier
- ✅ 使用Git Hooks保证代码质量
- ✅ 合理使用环境变量

### 4. 代码质量

- ✅ 统一的代码规范
- ✅ TypeScript类型检查
- ✅ 组件和函数的单元测试
- ✅ 提交前的代码检查

## 学习资源

### 官方文档
- [Vite官方文档](https://vitejs.dev/)
- [Vite中文文档](https://cn.vitejs.dev/)
- [Rollup文档](https://rollupjs.org/)

### 推荐阅读
- Vite插件开发指南
- Rollup打包原理
- ES模块规范

### 相关工具
- [vite-plugin-vue-devtools](https://github.com/webfansplz/vite-plugin-vue-devtools) - Vue开发者工具
- [VueUse](https://vueuse.org/) - Vue组合式函数集合
- [unplugin](https://github.com/unjs/unplugin) - 统一的插件系统

## 总结

本文详细介绍了Vite在Vue3项目中的使用:

- ✅ Vite的特点和优势
- ✅ 项目创建和基础配置
- ✅ 常用插件的使用
- ✅ 完整的项目结构
- ✅ 性能优化技巧
- ✅ 部署方案
- ✅ 开发技巧和最佳实践

Vite作为下一代前端构建工具,以其极快的启动速度和优秀的开发体验,正在成为Vue3项目的首选构建工具。掌握Vite的使用,将极大提升你的开发效率!

:::tip[继续学习]
👉 下一篇: [Vue3入门教程(四) - Pinia状态管理](/posts/vue3-pinia/)
:::

:::note[相关文章]
- [Vue3入门教程(一) - 基础与Composition API](/posts/vue3-basics/)
- [Vue3入门教程(二) - 组件与组合式函数](/posts/vue3-composables/)
:::