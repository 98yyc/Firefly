
---
title: Vue3 自定义指令与插件开发
published: 2024-01-16
description: 掌握Vue3自定义指令和插件的开发技巧,构建可复用的功能模块
tags: [Vue3, 自定义指令, 插件, 进阶]
category: 前端开发
draft: true
---

自定义指令和插件是Vue3中重要的扩展机制,它们可以帮助我们封装可复用的功能。本文将深入讲解如何开发高质量的自定义指令和插件。

## 1. 自定义指令基础

### 1.1 指令钩子函数

Vue3 中的指令钩子函数:

```typescript
import { Directive, DirectiveBinding } from 'vue'

const myDirective: Directive = {
  // 在绑定元素的 attribute 前或事件监听器应用前调用
  created(el, binding, vnode, prevVnode) {},
  
  // 在元素被插入到 DOM 前调用
  beforeMount(el, binding, vnode, prevVnode) {},
  
  // 在绑定元素的父组件及自己的所有子节点都挂载完成后调用
  mounted(el, binding, vnode, prevVnode) {},
  
  // 绑定元素的父组件更新前调用
  beforeUpdate(el, binding, vnode, prevVnode) {},
  
  // 在绑定元素的父组件及自己的所有子节点都更新后调用
  updated(el, binding, vnode, prevVnode) {},
  
  // 绑定元素的父组件卸载前调用
  beforeUnmount(el, binding, vnode, prevVnode) {},
  
  // 绑定元素的父组件卸载后调用
  unmounted(el, binding, vnode, prevVnode) {}
}
```

### 1.2 指令参数说明

```typescript
interface DirectiveBinding {
  value: any           // 传递给指令的值
  oldValue: any        // 之前的值,仅在 beforeUpdate 和 updated 中可用
  arg: string          // 传递给指令的参数
  modifiers: object    // 包含修饰符的对象
  instance: any        // 使用该指令的组件实例
  dir: Directive       // 指令的定义对象
}
```

## 2. 常用自定义指令实现

### 2.1 v-loading 加载指令

```typescript
// directives/loading.ts
import { Directive } from 'vue'
import { createApp } from 'vue'
import LoadingComponent from './Loading.vue'

interface LoadingElement extends HTMLElement {
  loadingInstance?: any
}

const loadingDirective: Directive = {
  mounted(el: LoadingElement, binding) {
    if (binding.value) {
      appendLoading(el, binding)
    }
  },
  updated(el: LoadingElement, binding) {
    if (binding.value !== binding.oldValue) {
      binding.value ? appendLoading(el, binding) : removeLoading(el)
    }
  },
  unmounted(el: LoadingElement) {
    removeLoading(el)
  }
}

function appendLoading(el: LoadingElement, binding: any) {
  const style = getComputedStyle(el)
  if (style.position === 'static') {
    el.style.position = 'relative'
  }

  const loadingContainer = document.createElement('div')
  loadingContainer.className = 'loading-container'
  
  const app = createApp(LoadingComponent, {
    text: binding.arg || '加载中...'
  })
  
  app.mount(loadingContainer)
  el.appendChild(loadingContainer)
  el.loadingInstance = { app, container: loadingContainer }
}

function removeLoading(el: LoadingElement) {
  if (el.loadingInstance) {
    el.loadingInstance.app.unmount()
    el.loadingInstance.container.remove()
    el.loadingInstance = null
  }
}

export default loadingDirective
```

```vue
<!-- Loading.vue -->
<template>
  <div class="loading-mask">
    <div class="loading-spinner">
      <div class="spinner"></div>
      <p v-if="text">{{ text }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  text?: string
}>()
</script>

<style scoped>
.loading-mask {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.loading-spinner {
  text-align: center;
}

.spinner {
  width: 40px;
  height: 40px;
  margin: 0 auto 10px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
```

使用示例:

```vue
<template>
  <div v-loading="isLoading" v-loading:正在加载数据>
    <!-- 内容 -->
  </div>
</template>
```

### 2.2 v-permission 权限指令

```typescript
// directives/permission.ts
import { Directive } from 'vue'
import { useUserStore } from '@/stores/user'

const permissionDirective: Directive = {
  mounted(el, binding) {
    const { value } = binding
    const userStore = useUserStore()
    const permissions = userStore.permissions

    if (value && value.length > 0) {
      const hasPermission = permissions.some((permission: string) =>
        value.includes(permission)
      )

      if (!hasPermission) {
        el.parentNode?.removeChild(el)
      }
    }
  }
}

export default permissionDirective
```

使用示例:

```vue
<template>
  <!-- 只有拥有 'admin' 或 'editor' 权限的用户才能看到 -->
  <button v-permission="['admin', 'editor']">编辑</button>
  
  <!-- 使用修饰符实现不同的权限逻辑 -->
  <button v-permission.all="['admin', 'write']">全部权限</button>
</template>
```

### 2.3 v-lazy 图片懒加载指令

```typescript
// directives/lazy.ts
import { Directive } from 'vue'

interface LazyOptions {
  loading?: string
  error?: string
  observerOptions?: IntersectionObserverInit
}

const defaultOptions: LazyOptions = {
  loading: '/placeholder.png',
  error: '/error.png',
  observerOptions: {
    rootMargin: '0px',
    threshold: 0.1
  }
}

const createLazyDirective = (options: LazyOptions = {}): Directive => {
  const config = { ...defaultOptions, ...options }
  
  const observer = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const img = entry.target as HTMLImageElement
        const src = img.dataset.src

        if (src) {
          img.src = src
          img.removeAttribute('data-src')
          observer.unobserve(img)
        }
      }
    })
  }, config.observerOptions)

  return {
    mounted(el: HTMLImageElement, binding) {
      el.src = config.loading!
      el.dataset.src = binding.value
      
      el.onerror = () => {
        el.src = config.error!
      }
      
      observer.observe(el)
    },
    updated(el: HTMLImageElement, binding) {
      if (binding.value !== binding.oldValue) {
        el.dataset.src = binding.value
      }
    },
    unmounted(el) {
      observer.unobserve(el)
    }
  }
}

export default createLazyDirective()
```

### 2.4 v-debounce 防抖指令

```typescript
// directives/debounce.ts
import { Directive } from 'vue'

interface DebounceElement extends HTMLElement {
  debounceTimer?: number
}

const debounceDirective: Directive = {
  mounted(el: DebounceElement, binding) {
    const { value, arg } = binding
    const delay = parseInt(arg || '300')
    
    el.addEventListener('input', (e: Event) => {
      if (el.debounceTimer) {
        clearTimeout(el.debounceTimer)
      }
      
      el.debounceTimer = window.setTimeout(() => {
        value(e)
      }, delay)
    })
  },
  unmounted(el: DebounceElement) {
    if (el.debounceTimer) {
      clearTimeout(el.debounceTimer)
    }
  }
}

export default debounceDirective
```

使用示例:

```vue
<template>
  <!-- 默认300ms防抖 -->
  <input v-debounce="handleSearch" />
  
  <!-- 自定义延迟时间为500ms -->
  <input v-debounce:500="handleSearch" />
</template>

<script setup lang="ts">
const handleSearch = (e: Event) => {
  const value = (e.target as HTMLInputElement).value
  console.log('搜索:', value)
}
</script>
```

### 2.5 v-copy 复制指令

```typescript
// directives/copy.ts
import { Directive } from 'vue'
import { ElMessage } from 'element-plus'

interface CopyElement extends HTMLElement {
  copyData?: string
  copyHandler?: () => void
}

const copyDirective: Directive = {
  mounted(el: CopyElement, binding) {
    el.copyData = binding.value
    el.copyHandler = () => {
      if (!el.copyData) {
        ElMessage.warning('没有要复制的内容')
        return
      }

      // 创建临时文本区域
      const textarea = document.createElement('textarea')
      textarea.value = el.copyData
      textarea.style.position = 'fixed'
      textarea.style.opacity = '0'
      document.body.appendChild(textarea)
      
      // 选择并复制
      textarea.select()
      
      try {
        const successful = document.execCommand('copy')
        if (successful) {
          ElMessage.success('复制成功')
        } else {
          ElMessage.error('复制失败')
        }
      } catch (err) {
        // 使用现代 API
        navigator.clipboard.writeText(el.copyData).then(
          () => ElMessage.success('复制成功'),
          () => ElMessage.error('复制失败')
        )
      }
      
      document.body.removeChild(textarea)
    }
    
    el.addEventListener('click', el.copyHandler)
  },
  updated(el: CopyElement, binding) {
    el.copyData = binding.value
  },
  unmounted(el: CopyElement) {
    if (el.copyHandler) {
      el.removeEventListener('click', el.copyHandler)
    }
  }
}

export default copyDirective
```

### 2.6 v-longpress 长按指令

```typescript
// directives/longpress.ts
import { Directive } from 'vue'

interface LongpressElement extends HTMLElement {
  pressTimer?: number
  longpressHandler?: (e: MouseEvent | TouchEvent) => void
}

const longpressDirective: Directive = {
  mounted(el: LongpressElement, binding) {
    const { value, arg } = binding
    const duration = parseInt(arg || '500')
    
    const start = (e: MouseEvent | TouchEvent) => {
      if (e.type === 'click') return
      
      if (el.pressTimer) {
        clearTimeout(el.pressTimer)
      }
      
      el.pressTimer = window.setTimeout(() => {
        value(e)
      }, duration)
    }
    
    const cancel = () => {
      if (el.pressTimer) {
        clearTimeout(el.pressTimer)
        el.pressTimer = undefined
      }
    }
    
    el.addEventListener('mousedown', start)
    el.addEventListener('touchstart', start)
    el.addEventListener('click', cancel)
    el.addEventListener('mouseout', cancel)
    el.addEventListener('touchend', cancel)
    el.addEventListener('touchcancel', cancel)
    
    el.longpressHandler = cancel
  },
  unmounted(el: LongpressElement) {
    if (el.pressTimer) {
      clearTimeout(el.pressTimer)
    }
  }
}

export default longpressDirective
```

## 3. 插件开发

### 3.1 基础插件结构

```typescript
// plugins/myPlugin.ts
import { App, Plugin } from 'vue'

interface MyPluginOptions {
  message?: string
}

const myPlugin: Plugin = {
  install(app: App, options: MyPluginOptions = {}) {
    // 1. 添加全局方法或属性
    app.config.globalProperties.$myMethod = () => {
      console.log(options.message || 'Hello from plugin')
    }
    
    // 2. 添加全局指令
    app.directive('my-directive', {
      mounted(el, binding) {
        el.textContent = binding.value
      }
    })
    
    // 3. 注册全局组件
    app.component('MyComponent', {
      template: '<div>My Component</div>'
    })
    
    // 4. 提供注入
    app.provide('myPluginKey', {
      hello: () => console.log('Hello from inject')
    })
  }
}

export default myPlugin
```

### 3.2 消息通知插件

```typescript
// plugins/message/index.ts
import { App, createApp, Plugin } from 'vue'
import MessageComponent from './Message.vue'

type MessageType = 'success' | 'warning' | 'error' | 'info'

interface MessageOptions {
  message: string
  type?: MessageType
  duration?: number
  onClose?: () => void
}

class MessageManager {
  private instances: any[] = []
  private seed = 1

  show(options: MessageOptions | string) {
    const opts = typeof options === 'string'
      ? { message: options }
      : options

    const id = `message_${this.seed++}`
    const container = document.createElement('div')
    container.id = id
    document.body.appendChild(container)

    const instance = createApp(MessageComponent, {
      ...opts,
      onClose: () => {
        this.close(id)
        opts.onClose?.()
      }
    })

    instance.mount(container)
    this.instances.push({ id, instance, container })

    // 自动关闭
    if (opts.duration !== 0) {
      setTimeout(() => {
        this.close(id)
      }, opts.duration || 3000)
    }

    return {
      close: () => this.close(id)
    }
  }

  close(id: string) {
    const index = this.instances.findIndex(item => item.id === id)
    if (index > -1) {
      const { instance, container } = this.instances[index]
      instance.unmount()
      container.remove()
      this.instances.splice(index, 1)
    }
  }

  closeAll() {
    this.instances.forEach(({ instance, container }) => {
      instance.unmount()
      container.remove()
    })
    this.instances = []
  }
}

const messageManager = new MessageManager()

const messagePlugin: Plugin = {
  install(app: App) {
    const message = {
      show: (options: MessageOptions | string) => messageManager.show(options),
      success: (message: string, duration?: number) =>
        messageManager.show({ message, type: 'success', duration }),
      warning: (message: string, duration?: number) =>
        messageManager.show({ message, type: 'warning', duration }),
      error: (message: string, duration?: number) =>
        messageManager.show({ message, type: 'error', duration }),
      info: (message: string, duration?: number) =>
        messageManager.show({ message, type: 'info', duration })
    }

    app.config.globalProperties.$message = message
    app.provide('message', message)
  }
}

export default messagePlugin
```

```vue
<!-- Message.vue -->
<template>
  <Transition name="message-fade">
    <div v-if="visible" :class="['message', `message-${type}`]">
      <span class="message-icon">{{ icon }}</span>
      <span class="message-text">{{ message }}</span>
      <button class="message-close" @click="handleClose">×</button>
    </div>
  </Transition>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const props = defineProps<{
  message: string
  type?: 'success' | 'warning' | 'error' | 'info'
  onClose?: () => void
}>()

const visible = ref(false)

const icon = computed(() => {
  const icons = {
    success: '✓',
    warning: '⚠',
    error: '✕',
    info: 'ℹ'
  }
  return icons[props.type || 'info']
})

const handleClose = () => {
  visible.value = false
  setTimeout(() => {
    props.onClose?.()
  }, 300)
}

onMounted(() => {
  visible.value = true
})
</script>

<style scoped>
.message {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 20px;
  border-radius: 4px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
  display: flex;
  align-items: center;
  gap: 8px;
  background: white;
  z-index: 9999;
}

.message-success {
  color: #67c23a;
  border: 1px solid #67c23a;
}

.message-warning {
  color: #e6a23c;
  border: 1px solid #e6a23c;
}

.message-error {
  color: #f56c6c;
  border: 1px solid #f56c6c;
}

.message-info {
  color: #909399;
  border: 1px solid #909399;
}

.message-close {
  border: none;
  background: none;
  font-size: 20px;
  cursor: pointer;
  padding: 0;
  margin-left: 8px;
}

