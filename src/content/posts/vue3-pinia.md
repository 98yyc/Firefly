---
title: Vue3入门教程(四) - Pinia状态管理
published: 2024-01-10
description: 学习Vue3官方推荐的状态管理库Pinia,掌握现代化的状态管理方案和最佳实践
tags: [Vue3, Pinia, 状态管理, 前端, JavaScript]
category: 前端开发
draft: false
---

# Vue3入门教程(四) - Pinia状态管理

Pinia是Vue3官方推荐的状态管理库,它是Vuex的继任者,提供了更简洁的API和更好的TypeScript支持。本文将详细介绍Pinia的使用方法和最佳实践。

## Pinia简介

### 什么是Pinia?

Pinia是专为Vue3设计的状态管理库,由Vue核心团队成员开发并维护。

**核心特点**:
- 🍍 极简的API设计
- 📦 模块化的Store组织
- 🔥 完整的TypeScript支持
- 🔌 可扩展的插件系统
- 🛠️ 优秀的开发工具支持
- ⚡ 轻量级(约1KB)

### Pinia vs Vuex

| 特性 | Pinia | Vuex 4 |
|------|-------|--------|
| 语法 | 更简洁 | 较复杂 |
| TypeScript支持 | 原生支持 | 需要额外配置 |
| Mutations | 不需要 | 需要 |
| 模块嵌套 | 扁平化 | 支持嵌套 |
| 代码分割 | 自动 | 手动 |
| DevTools | 支持 | 支持 |

### 为什么选择Pinia?

```js
// Vuex写法 - 复杂
const store = createStore({
  state: { count: 0 },
  mutations: {
    increment(state) {
      state.count++
    }
  },
  actions: {
    incrementAsync({ commit }) {
      setTimeout(() => {
        commit('increment')
      }, 1000)
    }
  }
})

// Pinia写法 - 简洁
const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  actions: {
    increment() {
      this.count++
    },
    async incrementAsync() {
      await new Promise(resolve => setTimeout(resolve, 1000))
      this.count++
    }
  }
})
```

## 安装和配置

### 安装Pinia

```bash
# npm
npm install pinia

# yarn
yarn add pinia

# pnpm
pnpm add pinia
```

### 基础配置

```js
// src/main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)
const pinia = createPinia()

app.use(pinia)
app.mount('#app')
```

## 定义Store

### Option Store写法

类似于Vue的Options API:

```js
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  // state - 定义状态
  state: () => ({
    count: 0,
    name: '计数器'
  }),
  
  // getters - 计算属性
  getters: {
    doubleCount: (state) => state.count * 2,
    
    // 使用其他getter
    doubleCountPlusOne() {
      return this.doubleCount + 1
    }
  },
  
  // actions - 方法
  actions: {
    increment() {
      this.count++
    },
    
    decrement() {
      this.count--
    },
    
    async incrementAsync() {
      await new Promise(resolve => setTimeout(resolve, 1000))
      this.count++
    },
    
    reset() {
      this.count = 0
    }
  }
})
```

### Setup Store写法

类似于Vue3的Composition API:

```js
// stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // state
  const count = ref(0)
  const name = ref('计数器')
  
  // getters
  const doubleCount = computed(() => count.value * 2)
  const doubleCountPlusOne = computed(() => doubleCount.value + 1)
  
  // actions
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  async function incrementAsync() {
    await new Promise(resolve => setTimeout(resolve, 1000))
    count.value++
  }
  
  function reset() {
    count.value = 0
  }
  
  return {
    count,
    name,
    doubleCount,
    doubleCountPlusOne,
    increment,
    decrement,
    incrementAsync,
    reset
  }
})
```

### 在组件中使用

```vue
<template>
  <div>
    <h2>{{ counter.name }}</h2>
    <p>Count: {{ counter.count }}</p>
    <p>Double: {{ counter.doubleCount }}</p>
    <p>Double + 1: {{ counter.doubleCountPlusOne }}</p>
    
    <button @click="counter.increment">+1</button>
    <button @click="counter.decrement">-1</button>
    <button @click="counter.incrementAsync">异步+1</button>
    <button @click="counter.reset">重置</button>
  </div>
</template>

<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()
</script>
```

### 解构Store

```vue
<script setup>
import { storeToRefs } from 'pinia'
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// ❌ 错误 - 直接解构会失去响应式
const { count, doubleCount } = counter

// ✅ 正确 - 使用storeToRefs保持响应式
const { count, doubleCount } = storeToRefs(counter)

// ✅ actions可以直接解构
const { increment, decrement } = counter
</script>
```

## State状态管理

### 访问State

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// 直接访问
console.log(counter.count)

// 使用$state访问整个状态
console.log(counter.$state)
</script>
```

### 重置State

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// 重置到初始状态
counter.$reset()
</script>
```

### 修改State

```js
// 方式1: 直接修改
counter.count++

// 方式2: 使用$patch对象
counter.$patch({
  count: counter.count + 1,
  name: '新名称'
})

// 方式3: 使用$patch函数
counter.$patch((state) => {
  state.count++
  state.name = '新名称'
})

// 方式4: 替换整个state
counter.$state = {
  count: 10,
  name: '替换后的名称'
}
```

### 订阅State变化

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// 订阅state变化
counter.$subscribe((mutation, state) => {
  console.log('mutation类型:', mutation.type)
  console.log('变化的store:', mutation.storeId)
  console.log('新的state:', state)
  
  // 持久化到localStorage
  localStorage.setItem('counter', JSON.stringify(state))
})
</script>
```

## Getters计算属性

### 基础Getters

```js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    firstName: '张',
    lastName: '三',
    age: 25
  }),
  
  getters: {
    // 自动推断返回类型
    fullName: (state) => `${state.firstName}${state.lastName}`,
    
    // 访问其他getter
    greeting() {
      return `你好, ${this.fullName}!`
    },
    
    // 返回函数(可传参)
    isAdult: (state) => {
      return (minAge) => state.age >= minAge
    }
  }
})
```

使用:

```vue
<script setup>
import { useUserStore } from '@/stores/user'

const user = useUserStore()

console.log(user.fullName)      // "张三"
console.log(user.greeting)      // "你好, 张三!"
console.log(user.isAdult(18))   // true
</script>
```

### 访问其他Store的Getters

```js
import { defineStore } from 'pinia'
import { useUserStore } from './user'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: []
  }),
  
  getters: {
    summary() {
      const user = useUserStore()
      return `${user.fullName}的购物车有${this.items.length}件商品`
    }
  }
})
```

### TypeScript类型标注

```ts
import { defineStore } from 'pinia'

interface State {
  count: number
  name: string
}

export const useCounterStore = defineStore('counter', {
  state: (): State => ({
    count: 0,
    name: '计数器'
  }),
  
  getters: {
    // 自动推断类型
    doubleCount: (state) => state.count * 2,
    
    // 手动标注返回类型
    tripleCount(): number {
      return this.count * 3
    }
  }
})
```

## Actions异步操作

### 基础Actions

```js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    loading: false,
    error: null
  }),
  
  actions: {
    // 同步操作
    setUser(user) {
      this.user = user
    },
    
    // 异步操作
    async fetchUser(id) {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch(`/api/users/${id}`)
        this.user = await response.json()
      } catch (error) {
        this.error = error.message
      } finally {
        this.loading = false
      }
    },
    
    // 清除用户
    clearUser() {
      this.user = null
      this.error = null
    }
  }
})
```

### 调用其他Store的Actions

```js
import { defineStore } from 'pinia'
import { useAuthStore } from './auth'

export const useUserStore = defineStore('user', {
  actions: {
    async logout() {
      const auth = useAuthStore()
      
      // 调用auth store的logout
      await auth.logout()
      
      // 清除用户数据
      this.user = null
    }
  }
})
```

### 订阅Actions

```vue
<script setup>
import { useUserStore } from '@/stores/user'

const user = useUserStore()

// 订阅action执行
user.$onAction(({
  name,      // action名称
  store,     // store实例
  args,      // 传递给action的参数
  after,     // action成功后的钩子
  onError    // action失败后的钩子
}) => {
  console.log(`Action ${name} 开始执行`)
  
  after((result) => {
    console.log(`Action ${name} 执行成功`)
  })
  
  onError((error) => {
    console.error(`Action ${name} 执行失败:`, error)
  })
})
</script>
```

## 实战案例

### 用户认证Store

```js
// stores/auth.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import router from '@/router'

export const useAuthStore = defineStore('auth', () => {
  // state
  const token = ref(localStorage.getItem('token') || '')
  const user = ref(null)
  const loading = ref(false)
  
  // getters
  const isLoggedIn = computed(() => !!token.value)
  const userName = computed(() => user.value?.name || '游客')
  
  // actions
  async function login(credentials) {
    loading.value = true
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      })
      
      const data = await response.json()
      
      if (data.token) {
        token.value = data.token
        user.value = data.user
        localStorage.setItem('token', data.token)
        
        // 跳转到首页
        router.push('/')
        
        return { success: true }
      }
    } catch (error) {
      return { success: false, error: error.message }
    } finally {
      loading.value = false
    }
  }
  
  async function logout() {
    try {
      await fetch('/api/auth/logout', {
        method: 'POST',
        headers: { 'Authorization': `Bearer ${token.value}` }
      })
    } finally {
      token.value = ''
      user.value = null
      localStorage.removeItem('token')
      router.push('/login')
    }
  }
  
  async function fetchUser() {
    if (!token.value) return
    
    try {
      const response = await fetch('/api/auth/me', {
        headers: { 'Authorization': `Bearer ${token.value}` }
      })
      user.value = await response.json()
    } catch (error) {
      // token无效,清除登录状态
      logout()
    }
  }
  
  function updateUser(userData) {
    user.value = { ...user.value, ...userData }
  }
  
  return {
    token,
    user,
    loading,
    isLoggedIn,
    userName,
    login,
    logout,
    fetchUser,
    updateUser
  }
})
```

使用认证Store:

```vue
<!-- Login.vue -->
<template>
  <form @submit.prevent="handleLogin">
    <input v-model="form.email" type="email" placeholder="邮箱" />
    <input v-model="form.password" type="password" placeholder="密码" />
    <button type="submit" :disabled="auth.loading">
      {{ auth.loading ? '登录中...' : '登录' }}
    </button>
  </form>
</template>

<script setup>
import { reactive } from 'vue'
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()

const form = reactive({
  email: '',
  password: ''
})

async function handleLogin() {
  const result = await auth.login(form)
  
  if (!result.success) {
    alert('登录失败: ' + result.error)
  }
}
</script>

<!-- Header.vue -->
<template>
  <header>
    <div v-if="auth.isLoggedIn">
      <span>欢迎, {{ auth.userName }}</span>
      <button @click="auth.logout">退出</button>
    </div>
    <div v-else>
      <router-link to="/login">登录</router-link>
    </div>
  </header>
</template>

<script setup>
import { useAuthStore } from '@/stores/auth'

const auth = useAuthStore()
</script>
```

### 购物车Store

```js
// stores/cart.js
import { defineStore } from 'pinia'
import { useAuthStore } from './auth'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: [],
    loading: false
  }),
  
  getters: {
    // 商品总数
    itemCount: (state) => {
      return state.items.reduce((total, item) => total + item.quantity, 0)
    },
    
    // 总价
    totalPrice: (state) => {
      return state.items.reduce(
        (total, item) => total + item.price * item.quantity,
        0
      )
    },
    
    // 格式化总价
    formattedTotal() {
      return `¥${this.totalPrice.toFixed(2)}`
    },
    
    // 检查商品是否在购物车
    hasItem: (state) => {
      return (productId) => {
        return state.items.some(item => item.id === productId)
      }
    }
  },
  
  actions: {
    // 添加商品
    addItem(product) {
      const existingItem = this.items.find(item => item.id === product.id)
      
      if (existingItem) {
        existingItem.quantity++
      } else {
        this.items.push({
          ...product,
          quantity: 1
        })
      }
      
      // 同步到服务器
      this.syncToServer()
    },
    
    // 移除商品
    removeItem(productId) {
      const index = this.items.findIndex(item => item.id === productId)
      if (index > -1) {
        this.items.splice(index, 1)
        this.syncToServer()
      }
    },
    
    // 更新数量
    updateQuantity(productId, quantity) {
      const item = this.items.find(item => item.id === productId)
      if (item) {
        if (quantity <= 0) {
          this.removeItem(productId)
        } else {
          item.quantity = quantity
          this.syncToServer()
        }
      }
    },
    
    // 清空购物车
    clear() {
      this.items = []
      this.syncToServer()
    },
    
    

    // 从服务器加载
    async fetchCart() {
      const auth = useAuthStore()
      
      if (!auth.isLoggedIn) {
        this.items = []
        return
      }
      
      this.loading = true
      
      try {
        const response = await fetch('/api/cart', {
          headers: { 'Authorization': `Bearer ${auth.token}` }
        })
        this.items = await response.json()
      } catch (error) {
        console.error('加载购物车失败:', error)
      } finally {
        this.loading = false
      }
    },
    
    // 同步到服务器
    async syncToServer() {
      const auth = useAuthStore()
      
      if (!auth.isLoggedIn) return
      
      try {
        await fetch('/api/cart', {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${auth.token}`
          },
          body: JSON.stringify(this.items)
        })
      } catch (error) {
        console.error('同步购物车失败:', error)
      }
    }
  }
})
```

使用购物车Store:

```vue
<!-- Cart.vue -->
<template>
  <div class="cart">
    <h2>购物车 ({{ cart.itemCount }}件商品)</h2>
    
    <div v-if="cart.loading">加载中...</div>
    
    <div v-else-if="cart.items.length === 0">
      购物车是空的
    </div>
    
    <div v-else>
      <div 
        v-for="item in cart.items" 
        :key="item.id"
        class="cart-item"
      >
        <img :src="item.image" :alt="item.name" />
        <div class="info">
          <h3>{{ item.name }}</h3>
          <p>¥{{ item.price }}</p>
        </div>
        <div class="quantity">
          <button @click="cart.updateQuantity(item.id, item.quantity - 1)">
            -
          </button>
          <span>{{ item.quantity }}</span>
          <button @click="cart.updateQuantity(item.id, item.quantity + 1)">
            +
          </button>
        </div>
        <button @click="cart.removeItem(item.id)">删除</button>
      </div>
      
      <div class="total">
        <h3>总计: {{ cart.formattedTotal }}</h3>
        <button @click="handleCheckout">结算</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import { useCartStore } from '@/stores/cart'

const cart = useCartStore()

onMounted(() => {
  cart.fetchCart()
})

function handleCheckout() {
  // 跳转到结算页面
  router.push('/checkout')
}
</script>
```

### Todo应用Store

```js
// stores/todos.js
import { defineStore } from 'pinia'

export const useTodosStore = defineStore('todos', {
  state: () => ({
    todos: [],
    filter: 'all', // 'all' | 'active' | 'completed'
    nextId: 1
  }),
  
  getters: {
    // 过滤后的todos
    filteredTodos: (state) => {
      switch (state.filter) {
        case 'active':
          return state.todos.filter(todo => !todo.completed)
        case 'completed':
          return state.todos.filter(todo => todo.completed)
        default:
          return state.todos
      }
    },
    
    // 统计
    stats: (state) => {
      const total = state.todos.length
      const completed = state.todos.filter(t => t.completed).length
      const active = total - completed
      
      return { total, completed, active }
    }
  },
  
  actions: {
    addTodo(text) {
      this.todos.push({
        id: this.nextId++,
        text,
        completed: false,
        createdAt: new Date()
      })
    },
    
    removeTodo(id) {
      const index = this.todos.findIndex(t => t.id === id)
      if (index > -1) {
        this.todos.splice(index, 1)
      }
    },
    
    toggleTodo(id) {
      const todo = this.todos.find(t => t.id === id)
      if (todo) {
        todo.completed = !todo.completed
      }
    },
    
    updateTodo(id, text) {
      const todo = this.todos.find(t => t.id === id)
      if (todo) {
        todo.text = text
      }
    },
    
    setFilter(filter) {
      this.filter = filter
    },
    
    clearCompleted() {
      this.todos = this.todos.filter(t => !t.completed)
    },
    
    toggleAll() {
      const allCompleted = this.todos.every(t => t.completed)
      this.todos.forEach(t => {
        t.completed = !allCompleted
      })
    }
  }
})
```

## 插件系统

### 持久化插件

```js
// plugins/persistedState.js
export function createPersistedState(options = {}) {
  const {
    key = 'pinia',
    storage = localStorage,
    paths = null  // 需要持久化的路径
  } = options
  
  return (context) => {
    const { store } = context
    
    // 从storage恢复数据
    const stored = storage.getItem(`${key}-${store.$id}`)
    if (stored) {
      store.$patch(JSON.parse(stored))
    }
    
    // 监听变化并保存
    store.$subscribe((mutation, state) => {
      let dataToStore = state
      
      // 只保存指定路径
      if (paths) {
        dataToStore = {}
        paths.forEach(path => {
          dataToStore[path] = state[path]
        })
      }
      
      storage.setItem(
        `${key}-${store.$id}`,
        JSON.stringify(dataToStore)
      )
    })
  }
}
```

使用插件:

```js
// main.js
import { createPinia } from 'pinia'
import { createPersistedState } from './plugins/persistedState'

const pinia = createPinia()

// 使用插件
pinia.use(createPersistedState({
  key: 'my-app',
  storage: localStorage
}))

app.use(pinia)
```

或者使用现成的插件:

```bash
npm install pinia-plugin-persistedstate
```

```js
// main.js
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

app.use(pinia)
```

```js
// stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    name: '',
    token: ''
  }),
  
  // 启用持久化
  persist: true
  
  // 或者配置持久化选项
  persist: {
    key: 'my-user',
    storage: sessionStorage,
    paths: ['token']  // 只持久化token
  }
})
```

### DevTools插件

```js
// plugins/devtools.js
export function createDevtoolsPlugin() {
  return (context) => {
    const { store } = context
    
    // 在action执行时记录日志
    store.$onAction(({ name, args, after, onError }) => {
      const startTime = Date.now()
      
      console.log(`🚀 [${store.$id}] ${name}`, args)
      
      after(() => {
        const duration = Date.now() - startTime
        console.log(`✅ [${store.$id}] ${name} (${duration}ms)`)
      })
      
      onError((error) => {
        console.error(`❌ [${store.$id}] ${name}:`, error)
      })
    })
  }
}
```

## 组合多个Store

### Store之间的通信

```js
// stores/notifications.js
import { defineStore } from 'pinia'

export const useNotificationsStore = defineStore('notifications', {
  state: () => ({
    notifications: []
  }),
  
  actions: {
    add(notification) {
      const id = Date.now()
      this.notifications.push({ id, ...notification })
      
      // 3秒后自动移除
      setTimeout(() => {
        this.remove(id)
      }, 3000)
    },
    
    remove(id) {
      const index = this.notifications.findIndex(n => n.id === id)
      if (index > -1) {
        this.notifications.splice(index, 1)
      }
    }
  }
})

// stores/user.js
import { defineStore } from 'pinia'
import { useNotificationsStore } from './notifications'

export const useUserStore = defineStore('user', {
  actions: {
    async updateProfile(data) {
      const notifications = useNotificationsStore()
      
      try {
        await api.updateProfile(data)
        notifications.add({
          type: 'success',
          message: '个人资料更新成功'
        })
      } catch (error) {
        notifications.add({
          type: 'error',
          message: '更新失败: ' + error.message
        })
      }
    }
  }
})
```

### 组合Store Hook

```js
// composables/useApp.js
import { computed } from 'vue'
import { useAuthStore } from '@/stores/auth'
import { useCartStore } from '@/stores/cart'
import { useNotificationsStore } from '@/stores/notifications'

export function useApp() {
  const auth = useAuthStore()
  const cart = useCartStore()
  const notifications = useNotificationsStore()
  
  // 组合的计算属性
  const appReady = computed(() => {
    return auth.user !== null && !cart.loading
  })
  
  // 组合的方法
  async function initialize() {
    if (auth.isLoggedIn) {
      await Promise.all([
        auth.fetchUser(),
        cart.fetchCart()
      ])
    }
  }
  
  function showSuccess(message) {
    notifications.add({ type: 'success', message })
  }
  
  function showError(message) {
    notifications.add({ type: 'error', message })
  }
  
  return {
    auth,
    cart,
    notifications,
    appReady,
    initialize,
    showSuccess,
    showError
  }
}
```

使用:

```vue
<script setup>
import { onMounted } from 'vue'
import { useApp } from '@/composables/useApp'

const app = useApp()

onMounted(() => {
  app.initialize()
})
</script>
```

## TypeScript支持

### 类型化的Store

```ts
// stores/user.ts
import { defineStore } from 'pinia'

// 定义State类型
interface UserState {
  user: User | null
  loading: boolean
  error: string | null
}

interface User {
  id: number
  name: string
  email: string
  avatar: string
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    user: null,
    loading: false,
    error: null
  }),
  
  getters: {
    userName: (state): string => {
      return state.user?.name || '游客'
    },
    
    isAdmin(): boolean {
      return this.user?.role === 'admin'
    }
  },
  
  actions: {
    async fetchUser(id: number): Promise<void> {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch(`/api/users/${id}`)
        this.user = await response.json()
      } catch (error) {
        this.error = (error as Error).message
      } finally {
        this.loading = false
      }
    },
    
    updateUser(data: Partial<User>): void {
      if (this.user) {
        this.user = { ...this.user, ...data }
      }
    }
  }
})
```

### Setup Store的TypeScript

```ts
// stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // State - 自动推断类型
  const count = ref(0)
  const name = ref('计数器')
  
  // Getters - 自动推断类型
  const doubleCount = computed(() => count.value * 2)
  
  // Actions - 手动标注参数和返回类型
  function increment(): void {
    count.value++
  }
  
  function incrementBy(amount: number): void {
    count.value += amount
  }
  
  async function fetchCount(): Promise<number> {
    const response = await fetch('/api/count')
    const data = await response.json()
    count.value = data.count
    return count.value
  }
  
  return {
    count,
    name,
    doubleCount,
    increment,
    incrementBy,
    fetchCount
  }
})
```

### 在组件中使用类型

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// 类型自动推断
const { user, loading } = storeToRefs(userStore)
const { fetchUser, updateUser } = userStore

// TypeScript会检查参数类型
fetchUser(123)  // ✅ 正确
// fetchUser('123')  // ❌ 错误: 参数类型不匹配

updateUser({ name: '新名字' })  // ✅ 正确
// updateUser({ invalid: 'field' })  // ❌ 错误: 未知属性
</script>
```

## 测试

### 单元测试Store

```js
// stores/__tests__/counter.spec.js
import { setActivePinia, createPinia } from 'pinia'
import { useCounterStore } from '../counter'
import { describe, it, expect, beforeEach } from 'vitest'

describe('Counter Store', () => {
  beforeEach(() => {
    // 创建新的pinia实例
    setActivePinia(createPinia())
  })
  
  it('初始值为0', () => {
    const counter = useCounterStore()
    expect(counter.count).toBe(0)
  })
  
  it('increment增加count', () => {
    const counter = useCounterStore()
    counter.increment()
    expect(counter.count).toBe(1)
  })
  
  it('doubleCount是count的两倍', () => {
    const counter = useCounterStore()
    counter.count = 5
    expect(counter.doubleCount).toBe(10)
  })
  
  it('reset重置count', () => {
    const counter = useCounterStore()
    counter.count = 10
    counter.reset()
    expect(counter.count).toBe(0)
  })
  
  it('异步操作', async () => {
    const counter = useCounterStore()
    await counter.incrementAsync()
    expect(counter.count).toBe(1)
  })
})
```

### 组件测试

```js
// components/__tests__/Counter.spec.js
import { mount } from '@vue/test-utils'
import { createPinia, setActivePinia } from 'pinia'
import Counter from '../Counter.vue'
import { useCounterStore } from '@/stores/counter'

describe('Counter Component', () => {
  let wrapper
  let store
  
  beforeEach(() => {
    setActivePinia(createPinia())
    store = useCounterStore()
    wrapper = mount(Counter, {
      global: {
        plugins: [createPinia()]
      }
    })
  })
  
  it('显示当前count', () => {
    expect(wrapper.text()).toContain('0')
  })
  
  it('点击按钮增加count', async () => {
    await wrapper.find('button').trigger('click')
    expect(store.count).toBe(1)
    expect(wrapper.text()).toContain('1')
  })
})
```

## 最佳实践

### 1. Store命名规范

```js
// ✅ 推荐
export const useUserStore = defineStore('user', {})
export const useCartStore = defineStore('cart', {})
export const useAuthStore = defineStore('auth', {})

// ❌ 不推荐
export const UserStore = defineStore('user', {})
export const user = defineStore('user', {})
```

### 2. 文件组织

```
stores/
├── index.js              # 导出所有store
├── auth.js               # 认证相关
├── user.js               # 用户相关
├── cart.js               # 购物车
├── products.js           # 商品
└── modules/              # 复杂模块
    ├── order/
    │   ├── index.js
    │   ├── list.js
    │   └── detail.js
    └── admin/
        ├── index.js
        └── users.js
```

### 3. State设计原则

```js
// ✅ 扁平化的state
state: () => ({
  user: null,
  loading: false,
  error: null
})

// ❌ 过度嵌套
state: () => ({
  data: {
    user: {
      profile: {
        info: {}
      }
    }
  }
})
```

### 4. Getters使用

```js
// ✅ 使用getters进行计算
getters: {
  fullName: (state) => `${state.firstName} ${state.lastName}`
}

// ❌ 在组件中计算
// {{ user.firstName + ' ' + user.lastName }}
```

### 5. Actions错误处理

```js
actions: {
  async fetchData() {
    this.loading = true
    this.error = null
    
    try {
      const data = await api.fetch()
      this.data = data
    } catch (error) {
      this.error = error.message
      // 可以在这里统一处理错误
      console.error('fetchData failed:', error)
    } finally {
      this.loading = false
    }
  }
}
```

### 6. 
避免在Getters中修改State

```js
// ❌ 错误 - getters不应该修改state
getters: {
  doubleCount(state) {
    state.count++  // 不要这样做!
    return state.count * 2
  }
}

// ✅ 正确 - getters只返回计算值
getters: {
  doubleCount: (state) => state.count * 2
}
```

## 从Vuex迁移

### 对比表

| Vuex概念 | Pinia等价 |
|---------|----------|
| state | state |
| getters | getters |
| mutations | ❌ 不需要 |
| actions | actions |
| modules | 每个store都是独立的 |

### 迁移步骤

**Vuex代码**:

```js
// store/modules/user.js
export default {
  namespaced: true,
  
  state: {
    user: null
  },
  
  mutations: {
    SET_USER(state, user) {
      state.user = user
    }
  },
  
  actions: {
    async fetchUser({ commit }, id) {
      const user = await api.getUser(id)
      commit('SET_USER', user)
    }
  },
  
  getters: {
    userName: state => state.user?.name || '游客'
  }
}
```

**迁移到Pinia**:

```js
// stores/user.js
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null
  }),
  
  getters: {
    userName: (state) => state.user?.name || '游客'
  },
  
  actions: {
    async fetchUser(id) {
      // 直接修改state,不需要mutations
      this.user = await api.getUser(id)
    }
  }
})
```

**组件使用对比**:

```vue
<!-- Vuex -->
<script setup>
import { useStore } from 'vuex'
import { computed } from 'vue'

const store = useStore()

const user = computed(() => store.state.user.user)
const userName = computed(() => store.getters['user/userName'])

const fetchUser = (id) => {
  store.dispatch('user/fetchUser', id)
}
</script>

<!-- Pinia -->
<script setup>
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()
const { user, userName } = storeToRefs(userStore)
const { fetchUser } = userStore
</script>
```

## 常见问题

### 1. Store在setup外使用

```js
// ❌ 错误 - 在setup外使用
const userStore = useUserStore()

export default {
  setup() {
    // ...
  }
}

// ✅ 正确 - 在setup或函数内使用
export default {
  setup() {
    const userStore = useUserStore()
    // ...
  }
}
```

### 2. 响应式丢失

```js
// ❌ 错误 - 直接解构
const { count } = useCounterStore()

// ✅ 正确 - 使用storeToRefs
const { count } = storeToRefs(useCounterStore())
```

### 3. Actions中的this指向

```js
// ✅ 在option store中使用this
actions: {
  increment() {
    this.count++  // 正确
  }
}

// ❌ 箭头函数中this不可用
actions: {
  increment: () => {
    this.count++  // 错误! this是undefined
  }
}
```

### 4. 循环依赖

```js
// stores/a.js
import { useStoreB } from './b'

export const useStoreA = defineStore('a', () => {
  const storeB = useStoreB()  // ❌ 可能导致循环依赖
})

// ✅ 解决方案: 在需要时才调用
export const useStoreA = defineStore('a', {
  actions: {
    someAction() {
      const storeB = useStoreB()  // ✅ 在action中使用
    }
  }
})
```

## 性能优化

### 1. 按需导入

```js
// ✅ 只导入需要的部分
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const { userName, isLoggedIn } = storeToRefs(useUserStore())
```

### 2. 批量更新

```js
// ❌ 多次触发更新
store.firstName = 'John'
store.lastName = 'Doe'
store.age = 30

// ✅ 一次性更新
store.$patch({
  firstName: 'John',
  lastName: 'Doe',
  age: 30
})
```

### 3. 懒加载Store

```js
// 在需要时才导入store
async function loadUserData() {
  const { useUserStore } = await import('@/stores/user')
  const userStore = useUserStore()
  await userStore.fetchUser()
}
```

## 调试技巧

### 1. DevTools

Pinia完整支持Vue DevTools:

- 查看所有Store的状态
- 时间旅行调试
- 编辑State
- 查看Actions历史

### 2. 自定义日志

```js
// stores/user.js
export const useUserStore = defineStore('user', {
  actions: {
    async fetchUser(id) {
      console.log('[User Store] Fetching user:', id)
      
      try {
        this.user = await api.getUser(id)
        console.log('[User Store] User loaded:', this.user)
      } catch (error) {
        console.error('[User Store] Error:', error)
      }
    }
  }
})
```

### 3. 监听Store变化

```js
import { watch } from 'vue'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// 监听整个store
watch(
  () => userStore.$state,
  (state) => {
    console.log('Store changed:', state)
  },
  { deep: true }
)

// 监听特定属性
watch(
  () => userStore.user,
  (user) => {
    console.log('User changed:', user)
  }
)
```

## 学习资源

### 官方文档
- [Pinia官方文档](https://pinia.vuejs.org/)
- [Pinia中文文档](https://pinia.vuejs.org/zh/)
- [从Vuex迁移](https://pinia.vuejs.org/cookbook/migration-vuex.html)

### 推荐插件
- [pinia-plugin-persistedstate](https://github.com/prazdevs/pinia-plugin-persistedstate) - 持久化
- [pinia-plugin-history](https://github.com/nandi95/pinia-plugin-history) - 历史记录
- [@pinia/colada](https://github.com/posva/pinia-colada) - 数据获取

### 相关资源
- Vue3响应式原理
- TypeScript最佳实践
- 状态管理设计模式

## 总结

本文全面介绍了Pinia状态管理:

- ✅ Pinia的核心概念和优势
- ✅ State、Getters、Actions的使用
- ✅ Option Store vs Setup Store
- ✅ 实战案例(认证、购物车、Todo)
- ✅ 插件系统和持久化
- ✅ TypeScript支持
- ✅ 测试方法
- ✅ 最佳实践和性能优化
- ✅ 从Vuex迁移指南

Pinia作为Vue3官方推荐的状态管理方案,以其简洁的API和强大的功能,让状态管理变得更加轻松。掌握Pinia是开发大型Vue3应用的必备技能!

:::tip[继续学习]
👉 下一篇: [Vue3入门教程(五) - TypeScript实战](/posts/vue3-ts/)
:::

:::note[相关文章]
- [Vue3入门教程(一) - 基础与Composition API](/posts/vue3-basics/)
- [Vue3入门教程(二) - 组件与组合式函数](/posts/vue3-composables/)
- [Vue3入门教程(三) - Vite项目构建](/posts/vue3-vite/)
- [Vuex状态管理教程](/posts/vue2-vuex/)
:::