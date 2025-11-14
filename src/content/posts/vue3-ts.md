---
title: Vue3入门教程(五) - TypeScript实战
published: 2024-01-11
description: 深入学习Vue3与TypeScript的结合使用,掌握类型安全的Vue3开发实践和最佳方案
tags: [Vue3, TypeScript, 前端, 类型安全, 工程化]
category: 前端开发
draft: false
---

# Vue3入门教程(五) - TypeScript实战

TypeScript为Vue3提供了完整的类型支持,让大型项目的开发更加可靠和高效。本文将详细介绍如何在Vue3项目中使用TypeScript。

## 为什么使用TypeScript?

### TypeScript的优势

- 🔍 **类型检查**: 在编译时发现错误
- 📝 **智能提示**: IDE提供更好的代码补全
- 🔧 **重构支持**: 安全地重命名和重构代码
- 📚 **自文档化**: 类型即文档
- 🛡️ **类型安全**: 减少运行时错误

### Vue3对TypeScript的支持

Vue3使用TypeScript重写,提供了:
- 完整的类型定义
- 原生TypeScript支持
- 更好的IDE体验
- Volar插件支持

## 项目配置

### 创建TypeScript项目

```bash
# 使用Vite创建
npm create vite@latest my-vue-app -- --template vue-ts

# 进入项目
cd my-vue-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

### tsconfig.json配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,

    /* 模块解析 */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "preserve",

    /* 类型检查 */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* 路径别名 */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Vite配置

```ts
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
  }
})
```

### VSCode配置

安装推荐插件:
- **Volar** - Vue3官方TypeScript插件
- **TypeScript Vue Plugin (Volar)** - 增强支持

```json
// .vscode/settings.json
{
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

## 基础类型使用

### 组件Props类型

```vue
<template>
  <div>
    <h2>{{ title }}</h2>
    <p>{{ count }}</p>
    <button @click="$emit('increment')">+1</button>
  </div>
</template>

<script setup lang="ts">
// 方式1: 使用defineProps的类型参数
interface Props {
  title: string
  count: number
  tags?: string[]  // 可选属性
}

const props = defineProps<Props>()

// 方式2: 带默认值
interface PropsWithDefaults {
  title: string
  count?: number
  tags?: string[]
}

const props = withDefaults(defineProps<PropsWithDefaults>(), {
  count: 0,
  tags: () => []
})
</script>
```

### Emits类型

```vue
<script setup lang="ts">
// 定义emit的类型
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
  (e: 'delete', id: number, confirmed: boolean): void
}>()

// 使用
emit('change', 1)
emit('update', 'new value')
emit('delete', 1, true)

// 或者使用更简洁的语法(Vue 3.3+)
const emit = defineEmits<{
  change: [id: number]
  update: [value: string]
  delete: [id: number, confirmed: boolean]
}>()
</script>
```

### Ref类型

```vue
<script setup lang="ts">
import { ref, Ref } from 'vue'

// 自动推断类型
const count = ref(0)  // Ref<number>
const message = ref('hello')  // Ref<string>

// 显式指定类型
const count: Ref<number> = ref(0)

// 或使用泛型
const count = ref<number>(0)

// 复杂类型
interface User {
  id: number
  name: string
  email: string
}

const user = ref<User | null>(null)
// 或
const user: Ref<User | null> = ref(null)

// 使用
user.value = {
  id: 1,
  name: '张三',
  email: 'zhang@example.com'
}
</script>
```

### Reactive类型

```vue
<script setup lang="ts">
import { reactive } from 'vue'

// 自动推断
const state = reactive({
  count: 0,
  message: 'hello'
})

// 使用接口定义类型
interface State {
  count: number
  message: string
  user: User | null
}

interface User {
  id: number
  name: string
}

const state: State = reactive({
  count: 0,
  message: 'hello',
  user: null
})

// 或使用泛型
const state = reactive<State>({
  count: 0,
  message: 'hello',
  user: null
})
</script>
```

### Computed类型

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

const count = ref(0)

// 自动推断返回类型
const doubleCount = computed(() => count.value * 2)  // ComputedRef<number>

// 显式指定类型
const doubleCount = computed<number>(() => count.value * 2)

// 可写的computed
const plusOne = computed({
  get: (): number => count.value + 1,
  set: (value: number) => {
    count.value = value - 1
  }
})
</script>
```

## 组件类型

### 组件实例类型

```vue
<!-- Child.vue -->
<script setup lang="ts">
const count = ref(0)

function increment() {
  count.value++
}

// 暴露给父组件
defineExpose({
  count,
  increment
})
</script>

<!-- Parent.vue -->
<template>
  <Child ref="childRef" />
  <button @click="handleClick">调用子组件方法</button>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Child from './Child.vue'

// 获取组件实例类型
const childRef = ref<InstanceType<typeof Child>>()

function handleClick() {
  childRef.value?.increment()
  console.log(childRef.value?.count)
}
</script>
```

### 组件Props类型复用

```ts
// types/components.ts
export interface ButtonProps {
  type?: 'primary' | 'secondary' | 'danger'
  size?: 'small' | 'medium' | 'large'
  disabled?: boolean
  loading?: boolean
}

export interface InputProps {
  modelValue: string
  placeholder?: string
  type?: 'text' | 'password' | 'email'
  disabled?: boolean
}
```

```vue
<!-- Button.vue -->
<script setup lang="ts">
import type { ButtonProps } from '@/types/components'

const props = withDefaults(defineProps<ButtonProps>(), {
  type: 'primary',
  size: 'medium',
  disabled: false,
  loading: false
})
</script>
```

### 泛型组件

```vue
<!-- List.vue -->
<template>
  <div>
    <div v-for="item in items" :key="item.id">
      <slot :item="item"></slot>
    </div>
  </div>
</template>

<script setup lang="ts" generic="T extends { id: number }">
// Vue 3.3+ 支持泛型组件
interface Props {
  items: T[]
}

const props = defineProps<Props>()
</script>

<!-- 使用 -->
<template>
  <List :items="users">
    <template #default="{ item }">
      {{ item.name }} - {{ item.email }}
    </template>
  </List>
</template>

<script setup lang="ts">
interface User {
  id: number
  name: string
  email: string
}

const users: User[] = [
  { id: 1, name: '张三', email: 'zhang@example.com' }
]
</script>
```

## 组合式函数类型

### 基础Composable

```ts
// composables/useCounter.ts
import { ref, Ref, computed, ComputedRef } from 'vue'

interface UseCounterOptions {
  min?: number
  max?: number
  step?: number
}

interface UseCounterReturn {
  count: Ref<number>
  doubleCount: ComputedRef<number>
  increment: () => void
  decrement: () => void
  reset: () => void
}

export function useCounter(
  initialValue: number = 0,
  options: UseCounterOptions = {}
): UseCounterReturn {
  const {
    min = -Infinity,
    max = Infinity,
    step = 1
  } = options

  const count = ref(initialValue)

  const doubleCount = computed(() => count.value * 2)

  const increment = () => {
    count.value = Math.min(max, count.value + step)
  }

  const decrement = () => {
    count.value = Math.max(min, count.value - step)
  }

  const reset = () => {
    count.value = initialValue
  }

  return {
    count,
    doubleCount,
    increment,
    decrement,
    reset
  }
}
```

### 异步Composable

```ts
// composables/useFetch.ts
import { ref, Ref, unref, watchEffect } from 'vue'

interface UseFetchOptions {
  immediate?: boolean
  refetch?: boolean
}

interface UseFetchReturn<T> {
  data: Ref<T | null>
  error: Ref<Error | null>
  loading: Ref<boolean>
  execute: () => Promise<void>
}

export function useFetch<T>(
  url: Ref<string> | string,
  options: UseFetchOptions = {}
): UseFetchReturn<T> {
  const {
    immediate = true,
    refetch = false
  } = options

  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)

  const execute = async () => {
    loading.value = true
    error.value = null

    try {
      const response = await fetch(unref(url))
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      data.value = await response.json()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }

  if (immediate) {
    execute()
  }

  if (refetch) {
    watchEffect(() => {
      execute()
    })
  }

  return {
    data,
    error,
    loading,
    execute
  }
}
```

使用:

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useFetch } from '@/composables/useFetch'

interface User {
  id: number
  name: string
  email: string
}

const userId = ref(1)
const url = computed(() => `/api/users/${userId.value}`)

const { data, error, loading } = useFetch<User>(url, {
  refetch: true
})
</script>
```

## Pinia与TypeScript

### 类型化的Store

```ts
// stores/user.ts
import { defineStore } from 'pinia'

// 定义State接口
export interface UserState {
  user: User | null
  token: string
  loading: boolean
  error: string | null
}

export interface User {
  id: number
  name: string
  email: string
  avatar: string
  role: 'user' | 'admin'
}

// 定义登录参数
export interface LoginCredentials {
  email: string
  password: string
}

// 定义响应类型
interface LoginResponse {
  token: string
  user: User
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    user: null,
    token: localStorage.getItem('token') || '',
    loading: false,
    error: null
  }),

  getters: {
    isLoggedIn: (state): boolean => !!state.token,
    
    userName: (state): string => {
      return state.user?.name || '游客'
    },
    
    isAdmin(): boolean {
      return this.user?.role === 'admin'
    }
  },

  actions: {
    async login(credentials: LoginCredentials): Promise<boolean> {
      this.loading = true
      this.error = null

      try {
        const response = await fetch('/api/auth/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(credentials)
        })

        if (!response.ok) {
          throw new Error('登录失败')
        }

        const data: LoginResponse = await response.json()
        
        this.token = data.token
        this.user = data.user
        localStorage.setItem('token', data.token)

        return true
      } catch (error) {
        this.error = (error as Error).message
        return false
      } finally {
        this.loading = false
      }
    },

    logout(): void {
      this.user = null
      this.token = ''
      localStorage.removeItem('token')
    },

    updateUser(userData: Partial<User>): void {
      if (this.user) {
        this.user = { ...this.user, ...userData }
      }
    }
  }
})
```

### Setup Store类型

```ts
// stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // State
  const count = ref<number>(0)
  const name = ref<string>('计数器')

  // Getters
  const doubleCount = computed<number>(() => count.value * 2)
  const isPositive = computed<boolean>(() => count.value > 0)

  // Actions
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

  function reset(): void {
    count.value = 0
  }

  return {
    count,
    name,
    doubleCount,
    isPositive,
    increment,
    incrementBy,
    fetchCount,
    reset
  }
})
```

## 路由类型

### 类型化的路由

```ts
// router/index.ts
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'

// 扩展路由meta类型
declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    title?: string
    icon?: string
    roles?: string[]
  }
}

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue'),
    meta: {
      title: '首页'
    }
  },
  {
    path: '/admin',
    name: 'Admin',
    component: () => import('@/views/Admin.vue'),
    meta: {
      requiresAuth: true,
      roles: ['admin']
    }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 类型安全的路由守卫
router.beforeEach((to, from, next) => {
  const requiresAuth = to.meta.requiresAuth
  
  if (requiresAuth) {
    // 检查认证
    const token = localStorage.getItem('token')
    if (!token) {
      next('/login')
      return
    }
  }
  
  next()
})

export default router
```

### 编程式导航类型

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 类型安全的导航
function goToUser(id: number) {
  router.push({
    name: 'UserDetail',
    params: { id: id.toString() }
  })
}

// 获取路由参数(需要类型断言)
const userId = Number(route.params.id)

// 获取查询参数
const page = Number(route.query.page) || 1
</script>
```

## API请求类型

### Axios配置

```ts
// utils/request.ts
import axios, { AxiosInstance, 
AxiosRequestConfig, AxiosResponse } from 'axios'

// 通用响应接口
interface ApiResponse<T = any> {
  code: number
  message: string
  data: T
}

// 创建axios实例
const request: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 5000
})

// 请求拦截器
request.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    const token = localStorage.getItem('token')
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => {
    return Promise.reject(error)
  }
)

// 响应拦截器
request.interceptors.response.use(
  (response: AxiosResponse<ApiResponse>) => {
    const { data } = response
    
    if (data.code !== 200) {
      return Promise.reject(new Error(data.message))
    }
    
    return data.data
  },
  (error) => {
    return Promise.reject(error)
  }
)

export default request
```

### API类型定义

```ts
// api/user.ts
import request from '@/utils/request'

// 用户接口
export interface User {
  id: number
  name: string
  email: string
  avatar: string
  role: 'user' | 'admin'
  createdAt: string
}

// 用户列表查询参数
export interface UserListParams {
  page?: number
  pageSize?: number
  keyword?: string
  role?: string
}

// 用户列表响应
export interface UserListResponse {
  list: User[]
  total: number
  page: number
  pageSize: number
}

// 创建用户参数
export interface CreateUserParams {
  name: string
  email: string
  password: string
  role?: 'user' | 'admin'
}

// API函数
export const getUserList = (params: UserListParams) => {
  return request.get<any, UserListResponse>('/users', { params })
}

export const getUserById = (id: number) => {
  return request.get<any, User>(`/users/${id}`)
}

export const createUser = (data: CreateUserParams) => {
  return request.post<any, User>('/users', data)
}

export const updateUser = (id: number, data: Partial<User>) => {
  return request.put<any, User>(`/users/${id}`, data)
}

export const deleteUser = (id: number) => {
  return request.delete<any, boolean>(`/users/${id}`)
}
```

使用API:

```vue
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { getUserList, type User, type UserListParams } from '@/api/user'

const users = ref<User[]>([])
const loading = ref(false)
const total = ref(0)

const params: UserListParams = {
  page: 1,
  pageSize: 10
}

async function fetchUsers() {
  loading.value = true
  
  try {
    const response = await getUserList(params)
    users.value = response.list
    total.value = response.total
  } catch (error) {
    console.error('获取用户列表失败:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchUsers()
})
</script>
```

## 高级类型应用

### 工具类型

```ts
// types/utils.ts

// 使用Partial - 所有属性变为可选
interface User {
  id: number
  name: string
  email: string
}

type PartialUser = Partial<User>
// 等同于: { id?: number; name?: string; email?: string }

// 使用Required - 所有属性变为必选
type RequiredUser = Required<PartialUser>

// 使用Readonly - 所有属性变为只读
type ReadonlyUser = Readonly<User>

// 使用Pick - 选择特定属性
type UserPreview = Pick<User, 'id' | 'name'>
// { id: number; name: string }

// 使用Omit - 排除特定属性
type UserWithoutId = Omit<User, 'id'>
// { name: string; email: string }

// 使用Record - 创建键值对类型
type UserRoles = Record<string, User[]>
// { [key: string]: User[] }

// 组合使用
type UpdateUserParams = Partial<Omit<User, 'id'>>
// { name?: string; email?: string }
```

### 联合类型和类型守卫

```ts
// types/index.ts

// 联合类型
type Status = 'pending' | 'success' | 'error'

interface PendingState {
  status: 'pending'
}

interface SuccessState {
  status: 'success'
  data: any
}

interface ErrorState {
  status: 'error'
  error: Error
}

type AsyncState = PendingState | SuccessState | ErrorState

// 类型守卫
function isSuccess(state: AsyncState): state is SuccessState {
  return state.status === 'success'
}

function isError(state: AsyncState): state is ErrorState {
  return state.status === 'error'
}

// 使用
function handleState(state: AsyncState) {
  if (isSuccess(state)) {
    console.log(state.data)  // TypeScript知道这里有data属性
  } else if (isError(state)) {
    console.error(state.error)  // TypeScript知道这里有error属性
  }
}
```

### 泛型约束

```ts
// composables/useLocalStorage.ts
import { ref, Ref, watch } from 'vue'

// 泛型约束 - T必须是可序列化的类型
function useLocalStorage<T extends object | string | number | boolean>(
  key: string,
  defaultValue: T
): Ref<T> {
  // 从localStorage读取
  const storedValue = localStorage.getItem(key)
  
  const data = ref<T>(
    storedValue ? JSON.parse(storedValue) : defaultValue
  ) as Ref<T>
  
  // 监听变化并保存
  watch(
    data,
    (newValue) => {
      localStorage.setItem(key, JSON.stringify(newValue))
    },
    { deep: true }
  )
  
  return data
}

// 使用
interface UserSettings {
  theme: 'light' | 'dark'
  language: 'zh' | 'en'
  notifications: boolean
}

const settings = useLocalStorage<UserSettings>('settings', {
  theme: 'light',
  language: 'zh',
  notifications: true
})
```

### 条件类型

```ts
// types/utils.ts

// 条件类型示例
type IsString<T> = T extends string ? true : false

type A = IsString<string>  // true
type B = IsString<number>  // false

// 提取Promise的返回类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T

type C = UnwrapPromise<Promise<string>>  // string
type D = UnwrapPromise<number>  // number

// 实用工具 - 深度只读
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? DeepReadonly<T[P]>
    : T[P]
}

interface Config {
  api: {
    baseURL: string
    timeout: number
  }
  features: {
    auth: boolean
  }
}

type ReadonlyConfig = DeepReadonly<Config>
```

## 最佳实践

### 1. 使用类型别名

```ts
// types/index.ts

// ✅ 推荐 - 使用类型别名
export type ID = number | string
export type Timestamp = number
export type Email = string

// 使用
interface User {
  id: ID
  email: Email
  createdAt: Timestamp
}
```

### 2. 接口vs类型别名

```ts
// 接口 - 可扩展
interface User {
  id: number
  name: string
}

interface User {  // 可以声明多次,会自动合并
  email: string
}

// 类型别名 - 不可扩展
type User = {
  id: number
  name: string
}

// ✅ 推荐用法
// 对象类型使用interface
interface User {
  id: number
  name: string
}

// 联合类型、工具类型使用type
type Status = 'idle' | 'loading' | 'success' | 'error'
type Nullable<T> = T | null
```

### 3. 避免any

```ts
// ❌ 不推荐
function process(data: any) {
  return data.value
}

// ✅ 推荐 - 使用泛型
function process<T>(data: T): T {
  return data
}

// ✅ 或使用unknown
function process(data: unknown) {
  if (typeof data === 'object' && data !== null) {
    // 类型守卫后才能使用
  }
}
```

### 4. 使用严格模式

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true
  }
}
```

### 5. 类型导出和导入

```ts
// types/user.ts
export interface User {
  id: number
  name: string
}

export type UserRole = 'admin' | 'user'

// 使用
import type { User, UserRole } from '@/types/user'
// 或
import { type User, type UserRole } from '@/types/user'
```

## 实战案例

### 完整的CRUD组件

```vue
<!-- views/UserList.vue -->
<template>
  <div class="user-list">
    <div class="header">
      <h2>用户管理</h2>
      <button @click="showCreateDialog = true">新增用户</button>
    </div>
    
    <div v-if="loading" class="loading">加载中...</div>
    
    <table v-else-if="users.length > 0">
      <thead>
        <tr>
          <th>ID</th>
          <th>姓名</th>
          <th>邮箱</th>
          <th>角色</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" :key="user.id">
          <td>{{ user.id }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>{{ user.role }}</td>
          <td>
            <button @click="handleEdit(user)">编辑</button>
            <button @click="handleDelete(user.id)">删除</button>
          </td>
        </tr>
      </tbody>
    </table>
    
    <div v-else class="empty">暂无数据</div>
    
    <!-- 分页 -->
    <div class="pagination">
      <button 
        :disabled="params.page === 1"
        @click="changePage(params.page! - 1)"
      >
        上一页
      </button>
      <span>{{ params.page }} / {{ totalPages }}</span>
      <button 
        :disabled="params.page === totalPages"
        @click="changePage(params.page! + 1)"
      >
        下一页
      </button>
    </div>
    
    <!-- 新增/编辑对话框 -->
    <UserDialog
      v-model="showDialog"
      :user="currentUser"
      @save="handleSave"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, reactive } from 'vue'
import { 
  getUserList, 
  createUser, 
  updateUser, 
  deleteUser,
  type User,
  type UserListParams,
  type CreateUserParams
} from '@/api/user'
import UserDialog from './UserDialog.vue'

// 状态
const users = ref<User[]>([])
const loading = ref(false)
const total = ref(0)
const showDialog = ref(false)
const showCreateDialog = ref(false)
const currentUser = ref<User | null>(null)

// 查询参数
const params = reactive<Required<UserListParams>>({
  page: 1,
  pageSize: 10,
  keyword: '',
  role: ''
})

// 计算属性
const totalPages = computed(() => {
  return Math.ceil(total.value / params.pageSize)
})

// 获取用户列表
async function fetchUsers() {
  loading.value = true
  
  try {
    const response = await getUserList(params)
    users.value = response.list
    total.value = response.total
  } catch (error) {
    console.error('获取用户列表失败:', error)
  } finally {
    loading.value = false
  }
}

// 编辑
function handleEdit(user: User) {
  currentUser.value = { ...user }
  showDialog.value = true
}

// 保存
async function handleSave(data: CreateUserParams | Partial<User>) {
  try {
    if (currentUser.value) {
      // 更新
      await updateUser(currentUser.value.id, data)
    } else {
      // 新增
      await createUser(data as CreateUserParams)
    }
    
    showDialog.value = false
    currentUser.value = null
    await fetchUsers()
  } catch (error) {
    console.error('保存失败:', error)
  }
}

// 删除
async function handleDelete(id: number) {
  if (!confirm('确定要删除吗?')) return
  
  try {
    await deleteUser(id)
    await fetchUsers()
  } catch (error) {
    console.error('删除失败:', error)
  }
}

// 翻页
function changePage(page: number) {
  params.page = page
  fetchUsers()
}

onMounted(() => {
  fetchUsers()
})
</script>

<style scoped>
.user-list {
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 20px;
  margin-top: 20px;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
```

### 表单验证Hook

```ts
// composables/useForm.ts
import { reactive, computed, UnwrapNestedRefs } from 'vue'

// 验证规则类型
export type ValidationRule<T = any> = (value: T) => string | null

// 字段配置
export interface FieldConfig<T> {
  value: T
  rules?: ValidationRule<T>[]
}

// 表单配置
export type FormConfig<T extends Record<string, any>> = {
  [K in keyof T]: FieldConfig<T[K]>
}

// 表单状态
export interface FormState<T extends Record<string, any>> {
  values: T
  errors: Partial<Record<keyof T, string>>
  touched: Partial<Record<keyof T, boolean>>
}

export function useForm<T extends Record<string, any>>(
  config: FormConfig<T>
) {
  // 提取初始值
  const initialValues = Object.keys(config).reduce((acc, key) => {
    acc[key as keyof T] = config[key as keyof T].value
    return acc
  }, {} as T)
  
  // 表单状态
  const state = reactive<FormState<T>>({
    values: { ...initialValues },
    errors: {},
    touched: {}
  })
  
  // 验证单个字段
  function validateField(field: keyof T): boolean {
    const fieldConfig = config[field]
    if (!fieldConfig.rules) return true
    
    const value = state.values[field]
    
    for (const rule of fieldConfig.rules) {
      const error = rule(value)
      if (error) {
        state.errors[field] = error
        return false
      }
    }
    
    delete state.errors[field]
    return true
  }
  
  // 验证所有字段
  function validateForm(): boolean {
    let isValid = true
    
    for (const field in config) {
      const valid = validateField(field as keyof T)
      if (!valid) isValid = false
    }
    
    return isValid
  }
  
  // 设置字段值
  function setFieldValue<K extends keyof T>(field: K, value: T[K]) {
    state.values[field] = value
    state.touched[field] = true
    validateField(field)
  }
  
  // 重置表单
  function resetForm() {
    state.values = { ...initialValues } as UnwrapNestedRefs<T>
    state.errors = {}
    state.touched = {}
  }
  
  // 计算属性
  const hasErrors = computed(() => {
    return Object.keys(state.errors).length > 0
  })
  
  const isValid = computed(() => {
    return !hasErrors.value && Object.keys(state.touched).length > 0
  })
  
  return {
    state,
    setFieldValue,
    validateField,
    validateForm,
    resetForm,
    hasErrors,
    isValid
  }
}

// 常用验证规则
export const validators = {
  required: (message = '此字段必填'): ValidationRule => {
    return (value) => {
      return value ? null : message
    }
  },
  
  minLength: (min: number, message?: string): ValidationRule<string> => {
    return (value) => {
      return value && value.length >= min
        ? null
        : message || `最少${min}个字符`
    }
  },
  
  maxLength: (max: number, message?: string): ValidationRule<string> => {
    return (value) => {
      return value && value.length <= max
        ? null
        : message || `最多${max}个字符`
    }
  },
  
  email: (message = 
'邮箱格式不正确'): ValidationRule<string> => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    return (value) => {
      return emailRegex.test(value) ? null : message
    }
  },
  
  pattern: (regex: RegExp, message: string): ValidationRule<string> => {
    return (value) => {
      return regex.test(value) ? null : message
    }
  }
}
```

使用表单验证:

```vue
<script setup lang="ts">
import { useForm, validators } from '@/composables/useForm'

interface LoginForm {
  email: string
  password: string
}

const { state, setFieldValue, validateForm, resetForm, isValid } = useForm<LoginForm>({
  email: {
    value: '',
    rules: [
      validators.required('邮箱必填'),
      validators.email()
    ]
  },
  password: {
    value: '',
    rules: [
      validators.required('密码必填'),
      validators.minLength(6, '密码至少6位')
    ]
  }
})

async function handleSubmit() {
  if (!validateForm()) return
  
  console.log('提交数据:', state.values)
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <div>
      <input
        :value="state.values.email"
        @input="e => setFieldValue('email', (e.target as HTMLInputElement).value)"
        placeholder="邮箱"
      />
      <span v-if="state.errors.email" class="error">
        {{ state.errors.email }}
      </span>
    </div>
    
    <div>
      <input
        type="password"
        :value="state.values.password"
        @input="e => setFieldValue('password', (e.target as HTMLInputElement).value)"
        placeholder="密码"
      />
      <span v-if="state.errors.password" class="error">
        {{ state.errors.password }}
      </span>
    </div>
    
    <button type="submit" :disabled="!isValid">登录</button>
  </form>
</template>
```

## 常见问题

### 1. 类型"EventTarget"上不存在属性

```vue
<script setup lang="ts">
// ❌ 错误
function handleInput(e: Event) {
  console.log(e.target.value)  // 类型"EventTarget"上不存在属性"value"
}

// ✅ 解决方案1: 类型断言
function handleInput(e: Event) {
  const target = e.target as HTMLInputElement
  console.log(target.value)
}

// ✅ 解决方案2: 使用InputEvent
function handleInput(e: Event) {
  if (e.target instanceof HTMLInputElement) {
    console.log(e.target.value)
  }
}

// ✅ 解决方案3: 指定更具体的事件类型
function handleInput(e: InputEvent) {
  const target = e.target as HTMLInputElement
  console.log(target.value)
}
</script>
```

### 2. ref的类型推断

```ts
// ❌ 问题: ref类型推断为never[]
const list = ref([])

// ✅ 解决: 指定泛型类型
const list = ref<string[]>([])
// 或
const list: Ref<string[]> = ref([])
```

### 3. 可选链和空值合并

```ts
interface User {
  name: string
  profile?: {
    avatar?: string
  }
}

const user: User | null = getUser()

// ✅ 使用可选链
const avatar = user?.profile?.avatar

// ✅ 使用空值合并
const displayName = user?.name ?? '游客'

// ✅ 组合使用
const avatarUrl = user?.profile?.avatar ?? '/default-avatar.png'
```

### 4. 非空断言

```ts
// 当你确定值不为null时使用
const user = getUser()
console.log(user!.name)  // 告诉TS: user一定不为null

// ⚠️ 谨慎使用,可能导致运行时错误
```

## 性能优化

### 1. 类型导入

```ts
// ✅ 使用type关键字导入类型
import type { User } from '@/types/user'

// 或在import语句中使用type
import { type User, type UserRole } from '@/types/user'

// 这样TypeScript会在编译时移除这些导入,减小打包体积
```

### 2. 按需导入类型

```ts
// ❌ 导入整个模块
import * as UserTypes from '@/types/user'

// ✅ 只导入需要的类型
import type { User, UserRole } from '@/types/user'
```

### 3. 使用const断言

```ts
// 更精确的类型推断
const colors = ['red', 'blue', 'green'] as const
// 类型: readonly ["red", "blue", "green"]

// 而不是
const colors: string[] = ['red', 'blue', 'green']
// 类型: string[]
```

## 调试技巧

### 1. VSCode中查看类型

```ts
// 鼠标悬停在变量上查看类型
const user = { name: '张三', age: 25 }

// 或使用// @ts-ignore查看类型错误
// @ts-ignore
const result: string = 123  // 会显示类型错误
```

### 2. 使用类型查询

```ts
// 查看变量的类型
type UserType = typeof user

// 查看函数返回值类型
type ReturnType = ReturnType<typeof fetchUser>

// 查看Promise解包后的类型
type AwaitedType = Awaited<Promise<User>>
```

### 3. TypeScript错误调试

```bash
# 查看详细的类型错误
tsc --noEmit --pretty

# 查看类型推断过程
tsc --noEmit --extendedDiagnostics
```

## 学习资源

### 官方文档
- [TypeScript官方文档](https://www.typescriptlang.org/)
- [TypeScript中文文档](https://www.tslang.cn/)
- [Vue3 TypeScript指南](https://cn.vuejs.org/guide/typescript/overview.html)

### 推荐工具
- [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) - Vue3 TypeScript插件
- [Type Challenges](https://github.com/type-challenges/type-challenges) - TypeScript类型挑战
- [ts-node](https://github.com/TypeStrong/ts-node) - 直接运行TS代码

### 相关资源
- TypeScript高级类型
- 装饰器和元数据
- 类型体操练习

## 总结

本文全面介绍了Vue3与TypeScript的结合使用:

- ✅ 项目配置和环境搭建
- ✅ 基础类型使用(Props、Emits、Ref、Reactive)
- ✅ 组件类型定义
- ✅ 组合式函数类型
- ✅ Pinia与TypeScript
- ✅ 路由类型定义
- ✅ API请求类型
- ✅ 高级类型应用
- ✅ 实战案例和最佳实践
- ✅ 常见问题解决

TypeScript为Vue3开发带来了强大的类型安全保障,虽然有一定的学习曲线,但掌握后能显著提升开发效率和代码质量。在大型项目中,TypeScript几乎是必不可少的!

:::tip[系列完结]
🎉 恭喜你完成了Vue3入门系列的学习!现在你已经掌握了:
- Vue3基础与Composition API
- 组件开发与组合式函数
- Vite项目构建
- Pinia状态管理  
- TypeScript实战

继续实践,成为Vue3高手! 💪
:::

:::note[相关文章]
- [Vue3入门教程(一) - 基础与Composition API](/posts/vue3-basics/)
- [Vue3入门教程(二) - 组件与组合式函数](/posts/vue3-composables/)
- [Vue3入门教程(三) - Vite项目构建](/posts/vue3-vite/)
- [Vue3入门教程(四) - Pinia状态管理](/posts/vue3-pinia/)
- [前端开发入门指南](/posts/frontend-intro/)
:::