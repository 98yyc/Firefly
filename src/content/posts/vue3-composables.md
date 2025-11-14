---
title: Vue3入门教程(二) - 组件与组合式函数
published: 2024-01-08
description: 深入学习Vue3组件系统、组合式函数(Composables)的设计与使用,掌握代码复用的最佳实践
tags: [Vue3, 前端, JavaScript, Composables, 组件化]
category: 前端开发
draft: false
---

# Vue3入门教程(二) - 组件与组合式函数

在上一篇文章中,我们学习了Vue3的基础知识和Composition API。本文将深入探讨Vue3的组件系统和组合式函数(Composables),这是Vue3中实现代码复用的核心方式。

## 组件基础回顾

### 定义组件

Vue3支持多种方式定义组件:

#### 1. 单文件组件(SFC)

```vue
<!-- MyComponent.vue -->
<template>
    <div class="my-component">
        <h2>{{ title }}</h2>
        <p>{{ content }}</p>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const title = ref('组件标题')
const content = ref('组件内容')
</script>

<style scoped>
.my-component {
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
}
</style>
```

#### 2. 函数式组件

```vue
<script setup>
// 函数式组件 - 无状态,纯展示
defineProps(['name', 'age'])
</script>

<template>
    <div>
        <p>姓名: {{ name }}</p>
        <p>年龄: {{ age }}</p>
    </div>
</template>
```

### 组件注册

#### 全局注册

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import MyComponent from './components/MyComponent.vue'

const app = createApp(App)

// 全局注册
app.component('MyComponent', MyComponent)

app.mount('#app')
```

#### 局部注册

```vue
<template>
    <div>
        <MyComponent />
    </div>
</template>

<script setup>
// 局部注册 - 直接导入即可使用
import MyComponent from './components/MyComponent.vue'
</script>
```

## Props深入

### Props类型定义

```vue
<script setup>
// 基础类型
defineProps({
    title: String,
    count: Number,
    isActive: Boolean,
    tags: Array,
    user: Object,
    callback: Function,
    any: null  // 任意类型
})

// 详细配置
defineProps({
    title: {
        type: String,
        required: true,
        default: '默认标题'
    },
    age: {
        type: Number,
        default: 0,
        validator: (value) => {
            return value >= 0 && value <= 150
        }
    },
    status: {
        type: String,
        default: 'pending',
        validator: (value) => {
            return ['pending', 'success', 'error'].includes(value)
        }
    }
})
</script>
```

### Props与TypeScript

```vue
<script setup lang="ts">
// 使用TypeScript定义Props
interface Props {
    title: string
    count?: number
    tags: string[]
    user: {
        name: string
        age: number
    }
}

// 带默认值
const props = withDefaults(defineProps<Props>(), {
    count: 0,
    tags: () => []
})
</script>
```

### Props解构

```vue
<script setup>
// ❌ 错误 - 直接解构会失去响应式
const { title, count } = defineProps(['title', 'count'])

// ✅ 正确方式1 - 不解构
const props = defineProps(['title', 'count'])
console.log(props.title)

// ✅ 正确方式2 - 使用toRefs
import { toRefs } from 'vue'

const props = defineProps(['title', 'count'])
const { title, count } = toRefs(props)
</script>
```

## Emits深入

### 基础用法

```vue
<!-- 子组件 -->
<template>
    <div>
        <button @click="handleClick">点击</button>
        <input @input="handleInput" />
    </div>
</template>

<script setup>
// 声明事件
const emit = defineEmits(['click', 'update', 'delete'])

const handleClick = () => {
    emit('click', { timestamp: Date.now() })
}

const handleInput = (e) => {
    emit('update', e.target.value)
}
</script>
```

### 事件验证

```vue
<script setup>
// 带验证的事件
const emit = defineEmits({
    // 无验证
    click: null,
    
    // 带验证
    submit: (payload) => {
        if (payload.email && payload.password) {
            return true
        } else {
            console.warn('Invalid submit payload!')
            return false
        }
    }
})

const handleSubmit = () => {
    emit('submit', {
        email: 'test@example.com',
        password: '123456'
    })
}
</script>
```

### TypeScript类型

```vue
<script setup lang="ts">
// 定义事件类型
const emit = defineEmits<{
    (e: 'change', id: number): void
    (e: 'update', value: string): void
    (e: 'delete', id: number): void
}>()

// 使用
emit('change', 1)
emit('update', 'new value')
emit('delete', 1)
</script>
```

## 双向绑定 v-model

### 基础v-model

```vue
<!-- 父组件 -->
<template>
    <CustomInput v-model="text" />
    <p>输入的内容: {{ text }}</p>
</template>

<script setup>
import { ref } from 'vue'

const text = ref('')
</script>

<!-- 子组件 CustomInput.vue -->
<template>
    <input
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)"
    />
</template>

<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>
```

### 多个v-model

```vue
<!-- 父组件 -->
<template>
    <UserForm
        v-model:firstName="first"
        v-model:lastName="last"
    />
    <p>全名: {{ first }} {{ last }}</p>
</template>

<script setup>
import { ref } from 'vue'

const first = ref('张')
const last = ref('三')
</script>

<!-- 子组件 UserForm.vue -->
<template>
    <input
        :value="firstName"
        @input="$emit('update:firstName', $event.target.value)"
        placeholder="名"
    />
    <input
        :value="lastName"
        @input="$emit('update:lastName', $event.target.value)"
        placeholder="姓"
    />
</template>

<script setup>
defineProps(['firstName', 'lastName'])
defineEmits(['update:firstName', 'update:lastName'])
</script>
```

### v-model修饰符

```vue
<!-- 父组件 -->
<template>
    <CustomInput v-model.capitalize="text" />
</template>

<!-- 子组件 -->
<script setup>
const props = defineProps({
    modelValue: String,
    modelModifiers: {
        default: () => ({})
    }
})

const emit = defineEmits(['update:modelValue'])

const handleInput = (e) => {
    let value = e.target.value
    
    // 处理capitalize修饰符
    if (props.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1)
    }
    
    emit('update:modelValue', value)
}
</script>

<template>
    <input
        :value="modelValue"
        @input="handleInput"
    />
</template>
```

## 插槽 Slots

### 默认插槽

```vue
<!-- 父组件 -->
<template>
    <Card>
        <h3>这是标题</h3>
        <p>这是内容</p>
    </Card>
</template>

<!-- 子组件 Card.vue -->
<template>
    <div class="card">
        <slot>默认内容</slot>
    </div>
</template>
```

### 具名插槽

```vue
<!-- 父组件 -->
<template>
    <Layout>
        <template #header>
            <h1>页面标题</h1>
        </template>
        
        <template #default>
            <p>主要内容</p>
        </template>
        
        <template #footer>
            <p>页脚信息</p>
        </template>
    </Layout>
</template>

<!-- 子组件 Layout.vue -->
<template>
    <div class="layout">
        <header>
            <slot name="header"></slot>
        </header>
        
        <main>
            <slot></slot>
        </main>
        
        <footer>
            <slot name="footer"></slot>
        </footer>
    </div>
</template>
```

### 作用域插槽

```vue
<!-- 父组件 -->
<template>
    <UserList>
        <template #default="{ user, index }">
            <div>
                <span>{{ index + 1 }}.</span>
                <strong>{{ user.name }}</strong>
                <span>({{ user.age }}岁)</span>
            </div>
        </template>
    </UserList>
</template>

<!-- 子组件 UserList.vue -->
<template>
    <div class="user-list">
        <div
            v-for="(user, index) in users"
            :key="user.id"
        >
            <slot :user="user" :index="index"></slot>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'

const users = ref([
    { id: 1, name: '张三', age: 25 },
    { id: 2, name: '李四', age: 30 },
    { id: 3, name: '王五', age: 28 }
])
</script>
```

## 组合式函数(Composables)

组合式函数是Vue3中最强大的代码复用方式,类似于React Hooks。

### 基础示例 - useMouse

```js
// composables/useMouse.js
import { ref, onMounted, onUnmounted } from 'vue'

export function useMouse() {
    const x = ref(0)
    const y = ref(0)
    
    function update(event) {
        x.value = event.pageX
        y.value = event.pageY
    }
    
    onMounted(() => {
        window.addEventListener('mousemove', update)
    })
    
    onUnmounted(() => {
        window.removeEventListener('mousemove', update)
    })
    
    return { x, y }
}
```

使用:

```vue
<template>
    <div>
        <p>鼠标位置: {{ x }}, {{ y }}</p>
    </div>
</template>

<script setup>
import { useMouse } from '@/composables/useMouse'

const { x, y } = useMouse()
</script>
```

### 异步状态管理 - useFetch

```js
// composables/useFetch.js
import { ref, watchEffect, toValue } from 'vue'

export function useFetch(url) {
    const data = ref(null)
    const error = ref(null)
    const loading = ref(false)
    
    const fetchData = async () => {
        loading.value = true
        data.value = null
        error.value = null
        
        try {
            const response = await fetch(toValue(url))
            data.value = await response.json()
        } catch (e) {
            error.value = e
        } finally {
            loading.value = false
        }
    }
    
    // 监听URL变化自动重新请求
    watchEffect(() => {
        fetchData()
    })
    
    return { data, error, loading }
}
```

使用:

```vue
<template>
    <div>
        <div v-if="loading">加载中...</div>
        <div v-else-if="error">错误: {{ error.message }}</div>
        <div v-else-if="data">
            <h3>{{ data.title }}</h3>
            <p>{{ data.body }}</p>
        </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { useFetch } from '@/composables/useFetch'

const id = ref(1)
const url = computed(() => `https://jsonplaceholder.typicode.com/posts/${id.value}`)

const { data, error, loading } = useFetch(url)
</script>
```

### 本地存储 - useLocalStorage

```js
// composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
    // 从localStorage读取初始值
    const storedValue = localStorage.getItem(key)
    const data = ref(
        storedValue ? JSON.parse(storedValue) : defaultValue
    )
    
    // 监听变化,自动保存到localStorage
    watch(data, (newValue) => {
        localStorage.setItem(key, JSON.stringify(newValue))
    }, { deep: true })
    
    return data
}
```

使用:

```vue
<template>
    <div>
        <input v-model="name" />
        <p>保存的名字: {{ name }}</p>
    </div>
</template>

<script setup>
import { useLocalStorage } from '@/composables/useLocalStorage'

// 自动从localStorage读取和保存
const name = useLocalStorage('user-name', '访客')
</script>
```

### 计数器 - useCounter

```js
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0, options = {}) {
    const {
        min = -Infinity,
        max = Infinity,
        step = 1
    } = options
    
    const count = ref(initialValue)
    
    const inc = (delta = step) => {
        count.value = Math.min(max, count.value + delta)
    }
    
    const dec = (delta = step) => {
        count.value = Math.max(min, count.value - delta)
    }
    
    const reset = () => {
        count.value = initialValue
    }
    
    const set = (value) => {
        count.value = Math.max(min, Math.min(max, value))
    }
    
    // 计算属性
    const isMin = computed(() => count.value <= min)
    const isMax = computed(() => count.value >= max)
    
    return {
        count,
        inc,
        dec,
        reset,
        set,
        isMin,
        isMax
    }
}
```

使用:

```vue
<template>
    <div>
        <button @click="dec" :disabled="isMin">-</button>
        <span>{{ count }}</span>
        <button @click="inc" :disabled="isMax">+</button>
        <button @click="reset">重置</button>
    </div>
</template>

<script setup>
import { useCounter } from '@/composables/useCounter'

const { count, inc, dec, reset, isMin, isMax } = useCounter(0, {
    min: 0,
    max: 10,
    step: 1
})
</script>
```

### 防抖和节流 - useDebounce

```js
// composables/useDebounce.js
import { ref, watch } from 'vue'

export function useDebounce(value, delay = 300) {
    const debouncedValue = ref(value.value)
    let timer = null
    
    watch(value, (newValue) => {
        clearTimeout(timer)
        timer = setTimeout(() => {
            debouncedValue.value = newValue
        }, delay)
    })
    
    return debouncedValue
}

// composables/useThrottle.js
import { ref, watch } from 'vue'

export function useThrottle(value, delay = 300) {
    const throttledValue = ref(value.value)
    let lastTime = 0
    
    watch(value, (newValue) => {
        const now = Date.now()
        if (now - lastTime >= delay) {
            throttledValue.value = newValue
            lastTime = now
        }
    })
    
    return throttledValue
}
```

使用:

```vue
<template>
    <div>
        <input v-model="searchText" placeholder="搜索..." />
        <p>防抖后的值: {{ debouncedSearch }}</p>
        <p>节流后的值: {{ throttledSearch }}</p>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { useDebounce } from '@/composables/useDebounce'
import { useThrottle } from '@/composables/useThrottle'

const searchText = ref('')

// 防抖 - 用户停止输入300ms后才更新
const debouncedSearch = useDebounce(searchText, 300)

// 节流 - 每300ms最多更新一次
const throttledSearch = useThrottle(searchText, 300)
</script>
```

### 暗黑模式 - useDark

```js
// composables/useDark.js
import { ref, watch } from 'vue'

export function useDark() {
    // 从localStorage读取或使用系统偏好
    const isDark = ref(
        localStorage.getItem('theme') === 'dark' ||
        (!localStorage.getItem('theme') && 
         window.matchMedia('(prefers-color-scheme: dark)').matches)
    )
    
    // 应用主题
    const applyTheme = (dark) => {
        if (dark) {
            document.documentElement.classList.add('dark')
        } else {
            document.documentElement.classList.remove('dark')
        }
        localStorage.setItem('theme', dark ? 'dark' : 'light')
    }
    
    // 初始应用
    applyTheme(isDark.value)
    
    // 监听变化
    watch(isDark, applyTheme)
    
    const toggle = () => {
        isDark.value = !isDark.value
    }
    
    return { isDark, toggle }
}
```

使用:

```vue
<template>
    <div>
        <button @click="toggle">
            切换到{{ isDark ? '浅色' : '深色' }}模式
        </button>
    </div>
</template>

<script setup>
import { useDark } from '@/composables/useDark'

const { isDark, toggle } = useDark()
</script>

<style>
/* 在CSS中使用 */

:root {
    --bg-color: white;
    --text-color: black;
}

.dark {
    --bg-color: #1a1a1a;
    --text-color: white;
}

body {
    background: var(--bg-color);
    color: var(--text-color);
}
</style>
```

## Composables最佳实践

### 1. 命名规范

```js
// ✅ 使用use前缀
export function useMouse() {}
export function useFetch() {}
export function useLocalStorage() {}

// ❌ 避免
export function mouse() {}
export function fetchData() {}
```

### 2. 返回值规范

```js
// ✅ 返回对象,方便解构和选择性使用
export function useCounter() {
    const count = ref(0)
    const inc = () => count.value++
    
    return { count, inc }  // 可以只用需要的
}

// ❌ 返回数组(不够语义化)
export function useCounter() {
    const count = ref(0)
    const inc = () => count.value++
    
    return [count, inc]
}
```

### 3. 接受ref作为参数

```js
import { ref, watch, toValue } from 'vue'

// ✅ 支持ref和普通值
export function useFetch(url) {
    const data = ref(null)
    
    watch(() => toValue(url), async (newUrl) => {
        const response = await fetch(newUrl)
        data.value = await response.json()
    }, { immediate: true })
    
    return { data }
}

// 使用
const url = ref('/api/user')
const { data } = useFetch(url)  // 支持响应式

const { data: data2 } = useFetch('/api/posts')  // 支持普通值
```

### 4. 副作用清理

```js
import { onUnmounted } from 'vue'

export function useEventListener(target, event, handler) {
    onUnmounted(() => {
        target.removeEventListener(event, handler)
    })
    
    target.addEventListener(event, handler)
}
```

### 5. 组合式函数的组合

```js
// useUser.js
export function useUser() {
    const user = useLocalStorage('user', null)
    const { data: profile, loading } = useFetch(
        computed(() => user.value ? `/api/users/${user.value.id}` : null)
    )
    
    return { user, profile, loading }
}
```

## 实战案例 - 完整的表单验证

```js
// composables/useForm.js
import { reactive, computed } from 'vue'

export function useForm(initialValues = {}) {
    const form = reactive({
        values: { ...initialValues },
        errors: {},
        touched: {}
    })
    
    const rules = reactive({})
    
    // 设置字段值
    const setFieldValue = (field, value) => {
        form.values[field] = value
        form.touched[field] = true
        validateField(field)
    }
    
    // 设置验证规则
    const setRules = (fieldRules) => {
        Object.assign(rules, fieldRules)
    }
    
    // 验证单个字段
    const validateField = (field) => {
        const fieldRules = rules[field]
        if (!fieldRules) return true
        
        const value = form.values[field]
        
        for (const rule of fieldRules) {
            const error = rule(value)
            if (error) {
                form.errors[field] = error
                return false
            }
        }
        
        delete form.errors[field]
        return true
    }
    
    // 验证所有字段
    const validateForm = () => {
        let isValid = true
        
        for (const field in rules) {
            const valid = validateField(field)
            if (!valid) isValid = false
        }
        
        return isValid
    }
    
    // 重置表单
    const resetForm = () => {
        form.values = { ...initialValues }
        form.errors = {}
        form.touched = {}
    }
    
    // 是否有错误
    const hasErrors = computed(() => {
        return Object.keys(form.errors).length > 0
    })
    
    // 是否可以提交
    const canSubmit = computed(() => {
        return !hasErrors.value && Object.keys(form.touched).length > 0
    })
    
    return {
        form,
        setFieldValue,
        setRules,
        validateField,
        validateForm,
        resetForm,
        hasErrors,
        canSubmit
    }
}

// 常用验证规则
export const validators = {
    required: (message = '此字段必填') => (value) => {
        return value ? null : message
    },
    
    minLength: (min, message) => (value) => {
        return value && value.length >= min 
            ? null 
            : message || `最少${min}个字符`
    },
    
    maxLength: (max, message) => (value) => {
        return value && value.length <= max 
            ? null 
            : message || `最多${max}个字符`
    },
    
    email: (message = '请输入有效的邮箱地址') => (value) => {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        return emailRegex.test(value) ? null : message
    },
    
    pattern: (regex, message) => (value) => {
        return regex.test(value) ? null : message
    }
}
```

使用表单验证:

```vue
<template>
    <form @submit.prevent="handleSubmit">
        <div>
            <label>用户名:</label>
            <input
                :value="form.values.username"
                @input="e => setFieldValue('username', e.target.value)"
                @blur="() => validateField('username')"
            />
            <span v-if="form.errors.username" class="error">
                {{ form.errors.username }}
            </span>
        </div>
        
        <div>
            <label>邮箱:</label>
            <input
                type="email"
                :value="form.values.email"
                @input="e => setFieldValue('email', e.target.value)"
                @blur="() => validateField('email')"
            />
            <span v-if="form.errors.email" class="error">
                {{ form.errors.email }}
            </span>
        </div>
        
        <div>
            <label>密码:</label>
            <input
                type="password"
                :value="form.values.password"
                @input="e => setFieldValue('password', e.target.value)"
                @blur="() => validateField('password')"
            />
            <span v-if="form.errors.password" class="error">
                {{ form.errors.password }}
            </span>
        </div>
        
        <button 
            type="submit" 
            :disabled="!canSubmit"
        >
            提交
        </button>
        
        <button 
            type="button" 
            @click="resetForm"
        >
            重置
        </button>
    </form>
</template>

<script setup>
import { useForm, validators } from '@/composables/useForm'

const {
    form,
    setFieldValue,
    setRules,
    validateField,
    validateForm,
    resetForm,
    canSubmit
} = useForm({
    username: '',
    email: '',
    password: ''
})

// 设置验证规则
setRules({
    username: [
        validators.required(),
        validators.minLength(3),
        validators.maxLength(20)
    ],
    email: [
        validators.required(),
        validators.email()
    ],
    password: [
        validators.required(),
        validators.minLength(6, '密码至少6位'),
        validators.pattern(/[A-Z]/, '密码必须包含大写字母')
    ]
})

const handleSubmit = () => {
    if (validateForm()) {
        console.log('提交表单:', form.values)
        // 发送请求...
    }
}
</script>

<style scoped>
.error {
    color: red;
    font-size: 14px;
}

button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}
</style>
```

## Provide/Inject 深层组件通信

### 基础用法

```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
const updateTheme = (newTheme) => {
    theme.value = newTheme
}

// 提供数据
provide('theme', theme)
provide('updateTheme', updateTheme)
</script>

<!-- 后代组件(任意层级) -->
<template>
    <div>
        <p>当前主题: {{ theme }}</p>
        <button @click="updateTheme('light')">切换到浅色</button>
    </div>
</template>

<script setup>
import { inject } from 'vue'

// 注入数据
const theme = inject('theme')
const updateTheme = inject('updateTheme')
</script>
```

### 使用Symbol作为key

```js
// keys.js
export const ThemeKey = Symbol('theme')
export const UserKey = Symbol('user')
```

```vue
<!-- 提供 -->
<script setup>
import { provide, ref } from 'vue'
import { ThemeKey } from './keys'

const theme = ref('dark')
provide(ThemeKey, theme)
</script>

<!-- 注入 -->
<script setup>
import { inject } from 'vue'
import { ThemeKey } from './keys'

const theme = inject(ThemeKey)
</script>
```

### 使用组合式函数封装

```js
// composables/useTheme.js
import { provide, inject, ref } from 'vue'

const ThemeSymbol = Symbol('theme')

// 提供主题
export function provideTheme() {
    const theme = ref('light')
    
    const setTheme = (newTheme) => {
        theme.value = newTheme
    }
    
    provide(ThemeSymbol, {
        theme,
        setTheme
    })
    
    return { theme, setTheme }
}

// 使用主题
export function useTheme() {
    const themeContext = inject(ThemeSymbol)
    
    if (!themeContext) {
        throw new Error('useTheme必须在provideTheme之后使用')
    }
    
    return themeContext
}
```

使用:

```vue
<!-- App.vue - 根组件 -->
<script setup>
import { provideTheme } from '@/composables/useTheme'

provideTheme()
</script>

<!-- 任意子组件 -->
<script setup>
import { useTheme } from '@/composables/useTheme'

const { theme, setTheme } = useTheme()
</script>
```

## 组件实战 - 可复用的Modal

```vue
<!-- components/Modal.vue -->
<template>
    <Teleport to="body">
        <Transition name="modal">
            <div 
                v-if="modelValue" 
                class="modal-overlay"
                @click="handleOverlayClick"
            >
                <div 
                    class="modal-content"
                    @click.stop
                >
                    <div class="modal-header">
                        <slot name="header">
                            <h3>{{ title }}</h3>
                        </slot>
                        <button 
                            class="close-btn"
                            @click="handleClose"
                        >
                            ✕
                        </button>
                    </div>
                    
                    <div class="modal-body">
                        <slot></slot>
                    </div>
                    
                    <div v-if="$slots.footer" class="modal-footer">
                        <slot name="footer"></slot>
                    </div>
                </div>
            </div>
        </Transition>
    </Teleport>
</template>

<script setup>
defineProps({
    modelValue: Boolean,
    title: String,
    closeOnOverlay: {
        type: Boolean,
        default: true
    }
})

const emit = defineEmits(['update:modelValue', 'close'])

const handleClose = () => {
    emit('update:modelValue', false)
    emit('close')
}

const handleOverlayClick = () => {
    if (props.closeOnOverlay) {
        handleClose()
    }
}
</script>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
}

.modal-content {
    background: white;
    border-radius: 12px;
    min-width: 400px;
    max-width: 90%;
    max-height: 90vh;
    overflow: auto;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px;
    border-bottom: 1px solid #eee;
}

.modal-body {
    padding: 20px;
}

.modal-footer {
    padding: 20px;
    border-top: 1px solid #eee;
    display: flex;
    justify-content: flex-end;
    gap: 10px;
}

.close-btn {
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: #999;
}

.close-btn:hover {
    color: #333;
}

/* 过渡动画 */
.modal-enter-active,
.modal-leave-active {
    transition: opacity 0.3s;
}

.modal-enter-from,
.modal-leave-to {
    opacity: 0;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
    transition: transform 0.3s;
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
    transform: scale(0.9);
}
</style>
```

使用Modal:

```vue
<template>
    <div>
        <button @click="showModal = true">打开弹窗</button>
        
        <Modal 
            v-model="showModal"
            title="确认操作"
            @close="handleClose"
        >
            <p>确定要删除这条记录吗?</p>
            
            <template #footer>
                <button @click="showModal = false">取消</button>
                <button @click="handleConfirm">确定</button>
            </template>
        </Modal>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import Modal from '@/components/Modal.vue'

const showModal = ref(false)

const handleClose = () => {
    console.log('弹窗关闭')
}

const handleConfirm = () => {
    console.log('确认删除')
    showModal.value = false
}
</script>
```

## 学习建议

### 1. 逐步掌握

- 先掌握基础组件通信(props、emits)
- 再学习插槽的使用
- 最后学习组合式函数的编写

### 2. 实践为主

- 自己编写常用的Composables
- 尝试重构现有代码
- 参考优秀的开源库(VueUse)

### 3. 设计原则

- 单一职责: 每个组合式函数只做一件事
- 可组合性: 组合式函数可以互相组合
- 可测试性: 易于单元测试
- 类型安全: 配合TypeScript使用

### 4. 推荐库

- [VueUse](https://vueuse.org/) - 优秀的组合式函数集合
- [Vue Macros](https://vue-macros.sxzz.moe/) - 实验性功能
- [Pinia](https://pinia.vuejs.org/) - 状态管理(下一篇会讲)

## 总结

本文深入讲解了Vue3组件系统和组合式函数:

- ✅ Props和Emits的高级用法
- ✅ 双向绑定v-model的实现
- ✅ 插槽的各种使用方式
- ✅ 组合式函数的设计与实践
- ✅ Provide/Inject深层通信
- ✅ 实战案例(表单验证、Modal组件)

组合式函数是Vue3最强大的特性之一,它让代码复用变得更加灵活和优雅。掌握好组合式函数的编写,将极大提升你的开发效率!

:::tip[继续学习]
👉 下一篇: [Vue3入门教程(三) - Vite项目构建](/posts/vue3-vite/)
:::

:::note[相关文章]
- [Vue3入门教程(一) - 基础与Composition API](/posts/vue3-basics/)
- [Vue2入门教程系列](/posts/vue2-basics/)
:::
