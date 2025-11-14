---
title: Vue2企业级项目实战指南
published: 2024-01-21
description: 从零开始构建一个完整的Vue2企业级项目，包含权限管理、国际化、主题切换、错误处理、单元测试等实战内容
tags: [Vue2, 前端, 项目实战, 企业级应用]
category: 前端开发
draft: true
---

# Vue2企业级项目实战指南

通过一个完整的企业级项目实战，掌握Vue2在真实业务场景中的应用。

## 一、项目初始化

### 1.1 创建项目

```bash
# 使用Vue CLI创建项目
vue create vue2-admin

# 选择配置
? Please pick a preset: Manually select features
? Check the features needed for your project:
  (*) Babel
  (*) Router
  (*) Vuex
  (*) CSS Pre-processors
  (*) Linter / Formatter
  (*) Unit Testing

? Use history mode for router? Yes
? Pick a CSS pre-processor: Sass/SCSS
? Pick a linter / formatter config: ESLint + Standard
? Pick a unit testing solution: Jest
```

### 1.2 安装依赖

```bash
npm install element-ui axios vue-i18n
npm install --save-dev sass sass-loader
```

### 1.3 项目目录结构

```
src/
├── api/              # API接口
├── assets/           # 静态资源
├── components/       # 通用组件
├── directives/       # 自定义指令
├── layout/           # 布局组件
├── locales/          # 国际化
├── router/           # 路由配置
├── store/            # 状态管理
├── utils/            # 工具函数
├── views/            # 页面组件
├── App.vue
└── main.js
```

## 二、权限管理系统

### 2.1 路由权限配置

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'
import store from '@/store'

Vue.use(Router)

// 静态路由
const constantRoutes = [
  {
    path: '/login',
    component: () => import('@/views/login'),
    hidden: true
  }
]

// 动态路由
export const asyncRoutes = [
  {
    path: '/user',
    component: Layout,
    meta: { 
      title: '用户管理',
      roles: ['admin']
    },
    children: [
      {
        path: 'list',
        component: () => import('@/views/user/list'),
        meta: { title: '用户列表' }
      }
    ]
  }
]

const router = new Router({
  mode: 'history',
  routes: constantRoutes
})

// 路由守卫
router.beforeEach(async (to, from, next) => {
  const token = store.getters.token
  
  if (token) {
    if (to.path === '/login') {
      next('/')
    } else {
      const hasRoles = store.getters.roles?.length > 0
      if (hasRoles) {
        next()
      } else {
        try {
          const { roles } = await store.dispatch('user/getInfo')
          const routes = await store.dispatch('permission/generateRoutes', roles)
          router.addRoutes(routes)
          next({ ...to, replace: true })
        } catch (error) {
          await store.dispatch('user/logout')
          next('/login')
        }
      }
    }
  } else {
    next(to.path === '/login' ? undefined : '/login')
  }
})

export default router
```

### 2.2 权限指令

```javascript
// directives/permission.js
import store from '@/store'

export default {
  inserted(el, binding) {
    const { value } = binding
    const roles = store.getters.roles
    
    if (value && value instanceof Array) {
      const hasPermission = roles.some(role => value.includes(role))
      if (!hasPermission) {
        el.parentNode?.removeChild(el)
      }
    }
  }
}

// 使用
<button v-permission="['admin']">删除</button>
```

## 三、国际化实现

### 3.1 配置i18n

```javascript
// locales/index.js
import Vue from 'vue'
import VueI18n from 'vue-i18n'
import elementZh from 'element-ui/lib/locale/lang/zh-CN'
import elementEn from 'element-ui/lib/locale/lang/en'
import zhCN from './zh-CN'
import enUS from './en-US'

Vue.use(VueI18n)

const messages = {
  'zh-CN': { ...zhCN, ...elementZh },
  'en-US': { ...enUS, ...elementEn }
}

export default new VueI18n({
  locale: localStorage.getItem('lang') || 'zh-CN',
  messages
})
```

### 3.2 语言包

```javascript
// locales/zh-CN.js
export default {
  common: {
    search: '搜索',
    add: '新增',
    edit: '编辑',
    delete: '删除'
  },
  route: {
    dashboard: '首页',
    user: '用户管理'
  }
}
```

### 3.3 切换语言

```vue
<template>
  <el-dropdown @command="handleLang">
    <span>{{ currentLang }}</span>
    <el-dropdown-menu slot="dropdown">
      <el-dropdown-item command="zh-CN">中文</el-dropdown-item>
      <el-dropdown-item command="en-US">English</el-dropdown-item>
    </el-dropdown-menu>
  </el-dropdown>
</template>

<script>
export default {
  computed: {
    currentLang() {
      return this.$i18n.locale === 'zh-CN' ? '中文' : 'English'
    }
  },
  methods: {
    handleLang(lang) {
      this.$i18n.locale = lang
      localStorage.setItem('lang', lang)
    }
  }
}
</script>
```

## 四、主题切换

### 4.1 CSS变量方案

```scss
// styles/theme.scss
:root {
  --primary-color: #409eff;
  --bg-color: #ffffff;
  --text-color: #303133;
}

[data-theme='dark'] {
  --bg-color: #1a1a1a;
  --text-color: #e4e7ed;
}

.app-container {
  background: var(--bg-color);
  color: var(--text-color);
}
```

### 4.2 主题切换

```javascript
// utils/theme.js
export function setTheme(theme) {
  document.documentElement.setAttribute('data-theme', theme)
  localStorage.setItem('theme', theme)
}

export function getTheme() {
  return localStorage.getItem('theme') || 'light'
}
```

## 五、请求封装

### 5.1 Axios配置

```javascript
// utils/request.js
import axios from 'axios'
import { Message } from 'element-ui'
import store from '@/store'

const service = axios.create({
  baseURL: process.env.VUE_APP_API,
  timeout: 10000
})

// 请求拦截
service.interceptors.request.use(
  config => {
    if (store.getters.token) {
      config.headers.Authorization = `Bearer ${store.getters.token}`
    }
    return config
  },
  error => Promise.reject(error)
)

// 响应拦截
service.interceptors.response.use(
  response => {
    const res = response.data
    if (res.code !== 200) {
      Message.error(res.message)
      if (res.code === 401) {
        store.dispatch('user/logout')
      }
      return Promise.reject(res)
    }
    return res.data
  },
  error => {
    Message.error(error.message)
    return Promise.reject(error)
  }
)

export default service
```

### 5.2 API模块化

```javascript
// api/user.js
import request from '@/utils/request'

export function login(data) {
  return request({
    url: '/user/login',
    method: 'post',
    data
  })
}

export function getUserInfo() {
  return request({
    url: '/user/info',
    method: 'get'
  })
}

export function getUserList(params) {
  return request({
    url: '/user/list',
    method: 'get',
    params
  })
}
```

## 六、全局错误处理

### 6.1 Vue错误处理

```javascript
// main.js
Vue.config.errorHandler = (err, vm, info) => {
  console.error('全局错误:', err)
  // 上报错误
  reportError({ error: err.message, info })
}

window.addEventListener('unhandledrejection', e => {
  console.error('未处理的Promise:', e.reason)
})
```

### 6.2 错误边界组件

```vue
<!-- ErrorBoundary.vue -->
<template>
  <div>
    <div v-if="error">
      <h3>出错了</h3>
      <p>{{ error.message }}</p>
      <button @click="reload">重试</button>
    </div>
    <slot v-else />
  </div>
</template>

<script>
export default {
  data() {
    return { error: null }
  },
  errorCaptured(err) {
    this.error = err
    return false
  },
  methods: {
    reload() {
      this.error = null
    }
  }
}
</script>
```

## 七、单元测试

### 7.1 组件测试

```javascript
// tests/unit/Button.spec.js
import { shallowMount } from '@vue/test-utils'
import Button from '@/components/Button.vue'

describe('Button.vue', () => {
  it('renders text', () => {
    const wrapper = shallowMount(Button, {
      propsData: { text: '点击' }
    })
    expect(wrapper.text()).toBe('点击')
  })
  
  it('emits click', () => {
    const wrapper = shallowMount(Button)
    wrapper.trigger('click')
    expect(wrapper.emitted('click')).toBeTruthy()
  })
})
```

### 7.2 Vuex测试

```javascript
// tests/unit/store/user.spec.js
import store from '@/store'

describe('User Store', () => {
  it('SET_TOKEN', () => {
    store.commit('user/SET_TOKEN', 'test-token')
    expect(store.state.user.token).toBe('test-token')
  })
})
```

## 八、性能优化

### 8.1 路由懒加载

```javascript
const routes = [
  {
    path: '/user',
    component: () => import(/* webpackChunkName: "user" */ '@/views/user')
  }
]
```

### 8.2 组件按需加载

```javascript
// babel.config.js
module.exports = {
  plugins: [
    [
      'component',
      {
        libraryName: 'element-ui',
        styleLibraryName: 'theme-chalk'
      }
    ]
  ]
}

// main.js
import { Button, Table } from 'element-ui'
Vue.use(Button)
Vue.use(Table)
```

### 8.3 图片优化

```javascript
// vue.config.js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule('images')
      .use('url-loader')
      .loader('url-loader')
      .tap(options => ({
        ...options,
        limit: 10240
      }))
  }
}
```

## 九、构建优化

### 9.1 生产环境配置

```javascript
// vue.config.js
const CompressionPlugin = require('compression-webpack-plugin')

module.exports = {
  productionSourceMap: false,
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      config.plugins.push(
        new CompressionPlugin({
          test: /\.(js|css)$/,
          threshold: 10240
        })
      )
    }
  },
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
          test: /[\\/]node_modules[\\/]element-ui[\\/]/
        }
      }
    })
  }
}
```

### 9.2 CDN加速

```javascript
// vue.config.js
const cdn = {
  css: ['https://unpkg.com/element-ui/lib/theme-chalk/index.css'],
  js: [
    'https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.min.js',
    'https://unpkg.com/element-ui/lib/index.js'
  ]
}

module.exports = {
  configureWebpack: {
    externals: {
      vue: 'Vue',
      'element-ui': 'ELEMENT'
    }
  },
  chainWebpack: config => {
    config.plugin('html').tap(args => {
      args[0].cdn = cdn
      return args
    })
  }
}
```

## 十、项目部署

### 10.1 Nginx配置

```nginx
server {
  listen 80;
  server_name example.com;
  root /var/www/dist;
  index index.html;
  
  location / {
    try_files $uri $uri/ /index.html;
  }
  
  location /api {
    proxy_pass http://localhost:3000;
  }
  
  gzip on;
  gzip_types text/css application/javascript;
}
```

### 10.2 CI/CD配置

```yaml
# .gitlab-ci.yml
stages:
  - install
  - build
  - deploy

install:
  stage: install
  script:
    - npm install
  cache:
    paths:
      - node_modules/

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/

deploy:
  stage: deploy
  script:
    - scp -r dist/* user@server:/var/www/
  only:
    - master
```

## 十一、最佳实践

### 11.1 代码规范

```javascript
// .eslintrc.js
module.exports = {
  extends: ['plugin:vue/essential', '@vue/standard'],
  rules: {
    'vue/multi-word-component-names': 'off',
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off'
  }
}
```

### 11.2 Git提交规范

```bash
# commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'test', 'chore']
    ]
  }
}

# 使用
git commit -m "feat: 添加用户管理模块"
git commit -m "fix: 修复登录bug"
```

## 总结

通过这个完整的企业级项目实战，我们学习了：

1. ✅ **权限管理** - 路由和按钮级权限控制
2. ✅ **国际化** - 多语言支持
3. ✅ **主题切换** - 动态主题系统
4. ✅ **请求封装** - 统一的API管理
5. ✅ **错误处理** - 全局错误捕获
6. ✅ **单元测试** - 组件和Store测试
7. ✅ **性能优化** - 懒加载和代码分割
8. ✅ **项目部署** - CI/CD和服务器配置

这些技能将帮助你构建出高质量的企业级Vue应用！

## 相关文章

- [Vue2基础入门](/posts/vue2-basics/)
- [Vue2性能优化](/posts/vue2-performance/)
- [Vue2高级组件模式](/posts/vue2-advanced-patterns/)
- [前端开发入门导航](/posts/frontend-intro/)