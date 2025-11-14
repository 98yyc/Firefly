---
title: Vue2入门教程(三) - Vue Router路由管理
published: 2024-01-04
pinned: false
description: 学习Vue Router实现单页应用的路由管理,掌握路由配置、导航、嵌套路由、路由守卫等核心功能。
tags: [Vue, Vue2, Vue Router, 前端, 教程]
category: 技术教程
draft: false
---

## 什么是Vue Router？

Vue Router是Vue.js官方的路由管理器。它与Vue.js深度集成,让构建单页面应用(SPA)变得轻而易举。

### Vue Router的功能

- 🔗 **嵌套路由/视图表** - 支持复杂的路由结构
- 🎯 **模块化路由配置** - 基于组件的路由配置
- 📍 **路由参数、查询、通配符** - 灵活的路由匹配
- 🔐 **导航守卫** - 控制路由访问权限
- 🎨 **过渡动效** - 路由切换动画
- 📜 **滚动行为** - 控制页面滚动位置
- 🏷️ **命名路由和视图** - 更好的路由管理

## 安装Vue Router

### CDN引入

```html
<script src="https://cdn.jsdelivr.net/npm/vue@2.7.14/dist/vue.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue-router@3.6.5/dist/vue-router.js"></script>
```

### NPM安装

```bash
npm install vue-router@3.6.5
```

## 基础使用

### 第一个路由应用

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>Vue Router 基础</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.7.14/dist/vue.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue-router@3.6.5/dist/vue-router.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; }
        .nav { margin-bottom: 20px; }
        .nav a {
            display: inline-block;
            padding: 10px 20px;
            margin-right: 10px;
            background: #42b983;
            color: white;
            text-decoration: none;
            border-radius: 5px;
        }
        .nav a.router-link-active {
            background: #35495e;
        }
    </style>
</head>
<body>
    <div id="app">
        <h1>Vue Router 示例</h1>
        
        <!-- 导航链接 -->
        <div class="nav">
            <router-link to="/">首页</router-link>
            <router-link to="/about">关于</router-link>
            <router-link to="/contact">联系</router-link>
        </div>
        
        <!-- 路由视图 -->
        <router-view></router-view>
    </div>

    <script>
        // 1. 定义路由组件
        const Home = {
            template: '<div><h2>首页</h2><p>欢迎来到首页</p></div>'
        }
        
        const About = {
            template: '<div><h2>关于</h2><p>这是关于页面</p></div>'
        }
        
        const Contact = {
            template: '<div><h2>联系我们</h2><p>联系方式: contact@example.com</p></div>'
        }

        // 2. 定义路由配置
        const routes = [
            { path: '/', component: Home },
            { path: '/about', component: About },
            { path: '/contact', component: Contact }
        ]

        // 3. 创建router实例
        const router = new VueRouter({
            routes
        })

        // 4. 创建和挂载根实例
        const app = new Vue({
            router
        }).$mount('#app')
    </script>
</body>
</html>
```

## 路由配置

### 基础路由

```js
const routes = [
    {
        path: '/',          // 路由路径
        name: 'home',       // 路由名称
        component: Home     // 路由组件
    },
    {
        path: '/about',
        name: 'about',
        component: About
    }
]
```

### 动态路由

使用冒号`:`标记动态路径参数:

```js
const User = {
    template: `
        <div>
            <h2>用户信息</h2>
            <p>用户ID: {{ $route.params.id }}</p>
            <p>用户名: {{ $route.params.username }}</p>
        </div>
    `
}

const routes = [
    // 动态路径参数以冒号开头
    { path: '/user/:id', component: User },
    // 多个参数
    { path: '/user/:id/:username', component: User }
]
```

访问路由参数:

```js
// 在组件中访问
this.$route.params.id
this.$route.params.username

// 使用props解耦
const User = {
    props: ['id', 'username'],
    template: '<div>用户ID: {{ id }}, 用户名: {{ username }}</div>'
}

const routes = [
    {
        path: '/user/:id/:username',
        component: User,
        props: true  // 将路由参数作为props传递
    }
]
```

### 嵌套路由

实际应用中,通常由多层嵌套的组件组成:

```js
const User = {
    template: `
        <div class="user">
            <h2>用户中心</h2>
            <router-link to="/user/profile">个人资料</router-link>
            <router-link to="/user/posts">我的文章</router-link>
            <router-link to="/user/settings">设置</router-link>
            
            <!-- 子路由出口 -->
            <router-view></router-view>
        </div>
    `
}

const UserProfile = {
    template: '<div>用户个人资料</div>'
}

const UserPosts = {
    template: '<div>用户文章列表</div>'
}

const UserSettings = {
    template: '<div>用户设置</div>'
}

const routes = [
    {
        path: '/user',
        component: User,
        children: [
            {
                // 当 /user/profile 匹配成功
                // UserProfile 会被渲染在 User 的 <router-view> 中
                path: 'profile',
                component: UserProfile
            },
            {
                path: 'posts',
                component: UserPosts
            },
            {
                path: 'settings',
                component: UserSettings
            }
        ]
    }
]
```

## 编程式导航

除了使用`<router-link>`创建a标签来定义导航链接,还可以使用路由实例的方法:

### router.push

```js
// 字符串路径
router.push('/home')

// 对象
router.push({ path: '/home' })

// 命名路由
router.push({ name: 'user', params: { userId: '123' } })

// 带查询参数,变成 /register?plan=private
router.push({ path: '/register', query: { plan: 'private' } })

// 在组件中使用
this.$router.push('/home')
```

### router.replace

与`router.push`类似,但不会向history添加新记录:

```js
router.replace('/home')
this.$router.replace({ path: '/home' })
```

### router.go

在history记录中前进或后退:

```js
// 前进一步
router.go(1)

// 后退一步
router.go(-1)

// 前进3步
router.go(3)

// 在组件中使用
this.$router.go(-1)  // 相当于浏览器的后退按钮
```

## 命名路由

可以给路由命名,在导航时使用名称而不是路径:

```js
const routes = [
    {
        path: '/user/:userId',
        name: 'user',
        component: User
    }
]

// 使用名称导航
router.push({ name: 'user', params: { userId: 123 } })
```

```html
<!-- 在模板中使用 -->
<router-link :to="{ name: 'user', params: { userId: 123 }}">
    用户
</router-link>
```

## 命名视图

可以在同一个路由下定义多个命名视图:

```html
<router-view></router-view>
<router-view name="sidebar"></router-view>
<router-view name="footer"></router-view>
```

```js
const routes = [
    {
        path: '/',
        components: {
            default: Home,
            sidebar: Sidebar,
            footer: Footer
        }
    }
]
```

## 重定向和别名

### 重定向

```js
const routes = [
    // 从 /a 重定向到 /b
    { path: '/a', redirect: '/b' },
    
    // 重定向到命名路由
    { path: '/a', redirect: { name: 'foo' }},
    
    // 动态重定向
    {
        path: '/a',
        redirect: to => {
            // 方法接收目标路由作为参数
            // return 重定向的字符串路径/路径对象
            return { path: '/search', query: { q: to.params.searchText } }
        }
    }
]
```

### 别名

```js
const routes = [
    {
        path: '/home',
        component: Home,
        alias: '/index'  // /home 的别名是 /index
    },
    {
        path: '/user',
        component: User,
        alias: ['/people', '/person']  // 多个别名
    }
]
```

## 路由组件传参

使用props将路由参数传递给组件,解耦路由和组件:

```js
const User = {
    props: ['id'],
    template: '<div>User {{ id }}</div>'
}

const routes = [
    // 布尔模式:将params设置为props
    { 
        path: '/user/:id', 
        component: User, 
        props: true 
    },
    
    // 对象模式:静态props
    { 
        path: '/user', 
        component: User, 
        props: { id: '123' } 
    },
    
    // 函数模式:自定义props
    {
        path: '/search',
        component: SearchUser,
        props: route => ({ 
            query: route.query.q 
        })
    }
]
```

## 导航守卫

Vue Router提供了导航守卫来控制路由的访问。

### 全局前置守卫

```js
const router = new VueRouter({ /* ... */ })

router.beforeEach((to, from, next) => {
    // to: 即将要进入的目标路由对象
    // from: 当前导航正要离开的路由
    // next: 一定要调用这个方法来resolve钩子
    
    console.log('导航到:', to.path)
    console.log('来自:', from.path)
    
    // 必须调用next()
    next()
})
```

### 权限验证示例

```js
router.beforeEach((to, from, next) => {
    // 检查路由是否需要认证
    if (to.matched.some(record => record.meta.requiresAuth)) {
        // 检查用户是否已登录
        if (!isLoggedIn()) {
            // 未登录,重定向到登录页
            next({
                path: '/login',
                query: { redirect: to.fullPath }
            })
        } else {
            // 已登录,继续导航
            next()
        }
    } else {
        // 不需要认证,直接通过
        next()
    }
})

// 路由配置
const routes = [
    {
        path: '/dashboard',
        component: Dashboard,
        meta: { requiresAuth: true }  // 需要认证
    },
    {
        path: '/login',
        component: Login
    }
]
```

### 全局后置钩子

```js
router.afterEach((to, from) => {
    // 这些钩子不会接受next函数也不会改变导航本身
    console.log('导航完成')
    
    // 可以用于页面标题设置
    document.title = to.meta.title || '默认标题'
})
```

### 路由独享守卫

```js
const routes = [
    {
        path: '/foo',
        component: Foo,
        beforeEnter: (to, from, next) => {
            // 只针对这个路由的守卫
            console.log('进入/foo路由')
            next()
        }
    }
]
```

### 组件内守卫

```js
const Foo = {
    template: `<div>foo</div>`,
    beforeRouteEnter(to, from, next) {
        // 在渲染该组件的对应路由被confirm前调用
        // 不能访问this,因为组件实例还没被创建
        next(vm => {
            // 通过vm访问组件实例
        })
    },
    beforeRouteUpdate(to, from, next) {
        // 在当前路由改变,但该组件被复用时调用
        // 可以访问this
        next()
    },
    beforeRouteLeave(to, from, next) {
        // 导航离开该组件的对应路由时调用
        // 可以访问this
        const answer = window.confirm('确定要离开吗?')
        if (answer) {
            next()
        } else {
            next(false)
        }
    }
}
```

## 路由元信息

可以在路由配置中添加meta字段,用于存储自定义信息:

```js
const routes = [
    {
        path: '/admin',
        component: Admin,
        meta: {
            requiresAuth: true,
            title: '管理后台',
            roles: ['admin', 'superadmin']
        }
    }
]

// 在导航守卫中使用
router.beforeEach((to, from, next) => {
    if (to.meta.requiresAuth) {
        // 需要验证
    }
    
    // 设置页面标题
    document.title = to.meta.title || '默认标题'
    
    next()
})
```

## 路由懒加载

当打包构建应用时,JavaScript包会变得非常大,影响页面加载。如果能把不同路由对应的组件分割成不同的代码块,然后当路由被访问时才加载对应组件,这样会更高效。

```js
// 使用动态import
const routes = [
    {
        path: '/home',
        component: () => import('./components/Home.vue')
    },
    {
        path: '/about',
        component: () => import('./components/About.vue')
    },
    {
        path: '/user',
        component: () => import('./components/User.vue')
    }
]

// 把组件按组分块
const routes = [
    {
        path: '/home',
        component: () => import(/* webpackChunkName: "group-home" */ './components/Home.vue')
    },
    {
        path: '/about',
        component: () => import(/* webpackChunkName: "group-about" */ './components/About.vue')
    }
]
```

## 实战练习: 博客系统

创建一个完整的博客系统,包含首页、文章列表、文章详情、关于页面:

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>博客系统</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.7.14/dist/vue.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue-router@3.6.5/dist/vue-router.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: Arial, sans-serif;
            background: #f5f5f5;
        }
        #app {
            max-width: 1200px;
            margin: 0 auto;
        }
        .header {
            background: white;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        .header h1 {
            color: #333;
            margin-bottom: 15px;
        }
        .nav {
            display: flex;
            gap: 10px;
        }
        .nav a {
            
padding: 10px 20px;
            background: #42b983;
            color: white;
            text-decoration: none;
            border-radius: 5px;
            transition: background 0.3s;
        }
        .nav a:hover {
            background: #35495e;
        }
        .nav a.router-link-active {
            background: #35495e;
        }
        .content {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            min-height: 500px;
        }
        .post-list {
            list-style: none;
        }
        .post-item {
            padding: 20px;
            border-bottom: 1px solid #eee;
            cursor: pointer;
            transition: background 0.3s;
        }
        .post-item:hover {
            background: #f9f9f9;
        }
        .post-title {
            font-size: 20px;
            color: #333;
            margin-bottom: 10px;
        }
        .post-meta {
            color: #999;
            font-size: 14px;
        }
        .post-detail h2 {
            color: #333;
            margin-bottom: 20px;
        }
        .post-content {
            line-height: 1.8;
            color: #666;
        }
        .back-btn {
            display: inline-block;
            margin-top: 20px;
            padding: 10px 20px;
            background: #42b983;
            color: white;
            text-decoration: none;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div id="app">
        <div class="header">
            <h1>📝 我的博客</h1>
            <div class="nav">
                <router-link to="/">首页</router-link>
                <router-link to="/posts">文章</router-link>
                <router-link to="/about">关于</router-link>
            </div>
        </div>
        
        <div class="content">
            <router-view></router-view>
        </div>
    </div>

    <script>
        // 首页组件
        const Home = {
            template: `
                <div>
                    <h2>欢迎来到我的博客</h2>
                    <p>这里分享技术文章和生活感悟</p>
                    <router-link to="/posts" class="back-btn">
                        查看文章列表
                    </router-link>
                </div>
            `
        }

        // 文章列表组件
        const PostList = {
            data() {
                return {
                    posts: [
                        { id: 1, title: 'Vue.js入门指南', author: '张三', date: '2024-01-01' },
                        { id: 2, title: 'JavaScript高级技巧', author: '李四', date: '2024-01-05' },
                        { id: 3, title: 'CSS布局实战', author: '王五', date: '2024-01-10' },
                        { id: 4, title: 'React vs Vue对比', author: '赵六', date: '2024-01-15' }
                    ]
                }
            },
            template: `
                <div>
                    <h2>文章列表</h2>
                    <ul class="post-list">
                        <li 
                            v-for="post in posts" 
                            :key="post.id"
                            class="post-item"
                            @click="goToPost(post.id)"
                        >
                            <div class="post-title">{{ post.title }}</div>
                            <div class="post-meta">
                                作者: {{ post.author }} | 发布时间: {{ post.date }}
                            </div>
                        </li>
                    </ul>
                </div>
            `,
            methods: {
                goToPost(id) {
                    this.$router.push(`/posts/${id}`)
                }
            }
        }

        // 文章详情组件
        const PostDetail = {
            data() {
                return {
                    post: null
                }
            },
            template: `
                <div class="post-detail" v-if="post">
                    <h2>{{ post.title }}</h2>
                    <div class="post-meta">
                        作者: {{ post.author }} | 发布时间: {{ post.date }}
                    </div>
                    <div class="post-content">
                        <p>{{ post.content }}</p>
                    </div>
                    <router-link to="/posts" class="back-btn">
                        返回列表
                    </router-link>
                </div>
                <div v-else>
                    <p>加载中...</p>
                </div>
            `,
            created() {
                // 模拟从API获取文章数据
                this.fetchPost()
            },
            methods: {
                fetchPost() {
                    const id = this.$route.params.id
                    // 模拟数据
                    const posts = {
                        '1': {
                            id: 1,
                            title: 'Vue.js入门指南',
                            author: '张三',
                            date: '2024-01-01',
                            content: 'Vue.js是一个渐进式JavaScript框架,易于学习,功能强大。本文将带你从零开始学习Vue.js的基础知识...'
                        },
                        '2': {
                            id: 2,
                            title: 'JavaScript高级技巧',
                            author: '李四',
                            date: '2024-01-05',
                            content: 'JavaScript作为Web开发的核心语言,掌握一些高级技巧可以让你的代码更加优雅和高效...'
                        },
                        '3': {
                            id: 3,
                            title: 'CSS布局实战',
                            author: '王五',
                            date: '2024-01-10',
                            content: 'CSS布局是前端开发的基础,本文将介绍Flexbox和Grid等现代布局技术...'
                        },
                        '4': {
                            id: 4,
                            title: 'React vs Vue对比',
                            author: '赵六',
                            date: '2024-01-15',
                            content: 'React和Vue都是优秀的前端框架,本文将从多个角度对比两者的异同...'
                        }
                    }
                    
                    setTimeout(() => {
                        this.post = posts[id]
                    }, 300)
                }
            },
            watch: {
                // 监听路由变化,重新获取数据
                '$route'(to, from) {
                    this.fetchPost()
                }
            }
        }

        // 关于页面组件
        const About = {
            template: `
                <div>
                    <h2>关于我</h2>
                    <p>我是一名前端开发工程师,热爱编程和分享。</p>
                    <p>主要技术栈: Vue.js, React, Node.js</p>
                    <p>联系方式: example@email.com</p>
                </div>
            `
        }

        // 路由配置
        const routes = [
            {
                path: '/',
                name: 'home',
                component: Home,
                meta: { title: '首页' }
            },
            {
                path: '/posts',
                name: 'posts',
                component: PostList,
                meta: { title: '文章列表' }
            },
            {
                path: '/posts/:id',
                name: 'post-detail',
                component: PostDetail,
                meta: { title: '文章详情' }
            },
            {
                path: '/about',
                name: 'about',
                component: About,
                meta: { title: '关于' }
            },
            {
                // 404页面
                path: '*',
                component: {
                    template: '<div><h2>404 - 页面未找到</h2></div>'
                }
            }
        ]

        // 创建路由实例
        const router = new VueRouter({
            routes
        })

        // 全局前置守卫 - 设置页面标题
        router.beforeEach((to, from, next) => {
            document.title = to.meta.title ? `${to.meta.title} - 我的博客` : '我的博客'
            next()
        })

        // 创建Vue实例
        new Vue({
            router
        }).$mount('#app')
    </script>
</body>
</html>
```

## History模式 vs Hash模式

Vue Router默认使用hash模式,URL会带有`#`符号。如果不想要这个符号,可以使用history模式。

### Hash模式(默认)

```js
const router = new VueRouter({
    routes
})

// URL: http://example.com/#/user/123
```

### History模式

```js
const router = new VueRouter({
    mode: 'history',
    routes
})

// URL: http://example.com/user/123
```

:::warning[服务器配置]
使用history模式需要服务器配置支持,否则刷新页面会404。
:::

### 服务器配置示例

**Nginx配置:**

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

**Apache配置(.htaccess):**

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

## 滚动行为

使用路由时,可以自定义路由切换时页面如何滚动:

```js
const router = new VueRouter({
    routes,
    scrollBehavior(to, from, savedPosition) {
        if (savedPosition) {
            // 浏览器前进/后退时,恢复之前的滚动位置
            return savedPosition
        } else if (to.hash) {
            // 如果有hash,滚动到指定元素
            return {
                selector: to.hash
            }
        } else {
            // 否则滚动到页面顶部
            return { x: 0, y: 0 }
        }
    }
})
```

## 路由最佳实践

### 1. 使用命名路由

```js
// ✅ 推荐
this.$router.push({ name: 'user', params: { id: 123 } })

// ❌ 不推荐
this.$router.push('/user/' + 123)
```

### 2. 路由懒加载

```js
// ✅ 推荐 - 按需加载
const routes = [
    {
        path: '/about',
        component: () => import('./views/About.vue')
    }
]

// ❌ 不推荐 - 一次性加载所有组件
import About from './views/About.vue'
const routes = [
    { path: '/about', component: About }
]
```

### 3. 使用路由守卫控制权限

```js
// ✅ 推荐
router.beforeEach((to, from, next) => {
    if (to.meta.requiresAuth && !isLoggedIn()) {
        next('/login')
    } else {
        next()
    }
})
```

### 4. 合理使用嵌套路由

```js
// ✅ 推荐 - 逻辑清晰
{
    path: '/user',
    component: User,
    children: [
        { path: 'profile', component: UserProfile },
        { path: 'posts', component: UserPosts }
    ]
}
```

### 5. 使用Props解耦

```js
// ✅ 推荐 - 组件可复用
{
    path: '/user/:id',
    component: User,
    props: true
}

// ❌ 不推荐 - 组件与路由耦合
{
    path: '/user/:id',
    component: User
}
// 组件内: this.$route.params.id
```

## 常见问题

### Q1: 路由切换时数据不刷新?

**原因**: 组件被复用时不会重新创建

**解决方案**:

```js
// 方法1: 监听路由变化
watch: {
    '$route'(to, from) {
        this.fetchData()
    }
}

// 方法2: 使用beforeRouteUpdate
beforeRouteUpdate(to, from, next) {
    this.fetchData()
    next()
}

// 方法3: 给router-view添加key
<router-view :key="$route.fullPath"></router-view>
```

### Q2: 如何实现页面切换动画?

```html
<transition name="fade" mode="out-in">
    <router-view></router-view>
</transition>

<style>
.fade-enter-active, .fade-leave-active {
    transition: opacity 0.3s;
}
.fade-enter, .fade-leave-to {
    opacity: 0;
}
</style>
```

### Q3: 如何获取上一个路由?

```js
// 在组件的beforeRouteEnter中
beforeRouteEnter(to, from, next) {
    console.log('来自:', from.path)
    next()
}

// 或在全局守卫中
router.beforeEach((to, from, next) => {
    console.log('来自:', from.path)
    next()
})
```

## 学习建议

1. **理解SPA原理**: 了解单页应用的工作原理
2. **掌握基础路由**: 先学会基本的路由配置和导航
3. **学习路由守卫**: 理解各种守卫的执行时机
4. **实践项目**: 通过实际项目加深理解
5. **阅读文档**: Vue Router官方文档很详细

## 下一步学习

掌握Vue Router后,接下来学习:

- ✅ **Vuex** - 全局状态管理
- ✅ **Vue CLI** - 项目脚手架
- ✅ **Axios** - HTTP请求库
- ✅ **Element UI** - UI组件库

## 总结

本文详细介绍了Vue Router的核心功能:

- 路由的基础配置和使用
- 动态路由和嵌套路由
- 编程式导航
- 命名路由和命名视图
- 路由守卫实现权限控制
- 路由懒加载优化性能
- History模式和Hash模式
- 滚动行为定制

Vue Router是构建单页应用的核心工具,掌握它对于开发Vue应用至关重要。在下一篇教程中,我们将学习Vuex,实现应用的全局状态管理!

:::tip[继续学习]
👉 上一篇: [Vue2入门教程(二) - 组件与通信](/posts/vue2-components/)

👉 下一篇: [Vue2入门教程(四) - Vuex状态管理](/posts/vue2-vuex/)
:::