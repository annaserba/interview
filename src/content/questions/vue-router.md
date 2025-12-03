---
title: "Vue Router"
description: "Маршрутизация в Vue.js приложениях"
category: "Vue.js"
difficulty: "medium"
tags: ["vue", "router", "navigation", "guards"]
order: 19
---

## Основы Vue Router

Vue Router — официальная библиотека маршрутизации для Vue.js.

### Установка и настройка

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### Подключение в приложении

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App)
  .use(router)
  .mount('#app')
```

## Динамические маршруты

### Параметры маршрута

```javascript
const routes = [
  {
    path: '/user/:id',
    name: 'User',
    component: User
  },
  {
    path: '/post/:id/:slug',
    name: 'Post',
    component: Post
  }
]
```

### Доступ к параметрам

```vue
<script setup>
import { useRoute } from 'vue-router'
import { computed } from 'vue'

const route = useRoute()
const userId = computed(() => route.params.id)
</script>

<template>
  <div>User ID: {{ userId }}</div>
</template>
```

### Options API

```vue
<script>
export default {
  computed: {
    userId() {
      return this.$route.params.id
    }
  },
  watch: {
    '$route.params.id'(newId) {
      // Реагируем на изменение параметра
      this.fetchUser(newId)
    }
  }
}
</script>
```

## Навигация

### Программная навигация

```vue
<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

function goToUser(id) {
  // По пути
  router.push(`/user/${id}`)
  
  // По имени маршрута
  router.push({ name: 'User', params: { id } })
  
  // С query параметрами
  router.push({ 
    name: 'User', 
    params: { id },
    query: { tab: 'profile' }
  })
}

function goBack() {
  router.go(-1) // или router.back()
}

function replace() {
  router.replace('/home') // не добавляет в историю
}
</script>
```

### Декларативная навигация

```vue
<template>
  <!-- Простая ссылка -->
  <router-link to="/about">About</router-link>
  
  <!-- По имени маршрута -->
  <router-link :to="{ name: 'User', params: { id: 123 }}">
    User Profile
  </router-link>
  
  <!-- С query параметрами -->
  <router-link :to="{ path: '/search', query: { q: 'vue' }}">
    Search
  </router-link>
  
  <!-- Активный класс -->
  <router-link 
    to="/about" 
    active-class="active"
    exact-active-class="exact-active"
  >
    About
  </router-link>
</template>
```

## Вложенные маршруты

```javascript
const routes = [
  {
    path: '/user/:id',
    component: User,
    children: [
      {
        path: '', // /user/:id
        component: UserHome
      },
      {
        path: 'profile', // /user/:id/profile
        component: UserProfile
      },
      {
        path: 'posts', // /user/:id/posts
        component: UserPosts
      }
    ]
  }
]
```

### Родительский компонент

```vue
<!-- User.vue -->
<template>
  <div class="user">
    <h1>User {{ $route.params.id }}</h1>
    <nav>
      <router-link :to="`/user/${$route.params.id}`">Home</router-link>
      <router-link :to="`/user/${$route.params.id}/profile`">Profile</router-link>
      <router-link :to="`/user/${$route.params.id}/posts`">Posts</router-link>
    </nav>
    
    <!-- Вложенные маршруты рендерятся здесь -->
    <router-view />
  </div>
</template>
```

## Именованные представления

```javascript
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

```vue
<template>
  <router-view />
  <router-view name="sidebar" />
  <router-view name="footer" />
</template>
```

## Navigation Guards (Навигационные хуки)

### Глобальные хуки

```javascript
// router/index.js
const router = createRouter({ /* ... */ })

// Перед каждым переходом
router.beforeEach((to, from, next) => {
  const isAuthenticated = checkAuth()
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    next('/login')
  } else {
    next()
  }
})

// После каждого перехода
router.afterEach((to, from) => {
  // Аналитика, логирование
  console.log(`Navigated from ${from.path} to ${to.path}`)
})

// Перед разрешением навигации
router.beforeResolve((to, from, next) => {
  // Вызывается после всех in-component guards
  next()
})
```

### Хуки маршрута

```javascript
const routes = [
  {
    path: '/admin',
    component: Admin,
    beforeEnter: (to, from, next) => {
      if (isAdmin()) {
        next()
      } else {
        next('/forbidden')
      }
    }
  }
]
```

### Хуки компонента

```vue
<script>
export default {
  // Options API
  beforeRouteEnter(to, from, next) {
    // НЕТ доступа к this
    next(vm => {
      // Доступ через vm после создания компонента
      vm.fetchData()
    })
  },
  
  beforeRouteUpdate(to, from, next) {
    // Вызывается при изменении параметров того же маршрута
    this.fetchData(to.params.id)
    next()
  },
  
  beforeRouteLeave(to, from, next) {
    // Можно предотвратить уход со страницы
    if (this.hasUnsavedChanges) {
      const answer = window.confirm('Есть несохранённые изменения. Уйти?')
      next(answer)
    } else {
      next()
    }
  }
}
</script>
```

### Composition API

```vue
<script setup>
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'
import { ref } from 'vue'

const hasUnsavedChanges = ref(false)

onBeforeRouteUpdate(async (to, from) => {
  // Можно вернуть false для отмены навигации
  if (to.params.id !== from.params.id) {
    await fetchData(to.params.id)
  }
})

onBeforeRouteLeave((to, from) => {
  if (hasUnsavedChanges.value) {
    const answer = window.confirm('Есть несохранённые изменения. Уйти?')
    return answer
  }
})
</script>
```

## Ленивая загрузка маршрутов

```javascript
const routes = [
  {
    path: '/about',
    // Динамический импорт
    component: () => import('../views/About.vue')
  },
  {
    path: '/admin',
    // С именованным chunk
    component: () => import(/* webpackChunkName: "admin" */ '../views/Admin.vue')
  }
]
```

### Группировка маршрутов

```javascript
const routes = [
  {
    path: '/user',
    component: () => import(/* webpackChunkName: "user" */ '../views/User.vue')
  },
  {
    path: '/user/profile',
    component: () => import(/* webpackChunkName: "user" */ '../views/UserProfile.vue')
  }
]
```

## Meta поля

```javascript
const routes = [
  {
    path: '/admin',
    component: Admin,
    meta: { 
      requiresAuth: true,
      roles: ['admin'],
      title: 'Admin Panel'
    }
  }
]

router.beforeEach((to, from, next) => {
  // Проверка мета-полей
  if (to.meta.requiresAuth) {
    // Проверка авторизации
  }
  
  // Установка заголовка
  document.title = to.meta.title || 'Default Title'
  
  next()
})
```

## Режимы истории

### HTML5 History Mode

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes
})
```

Требует настройки сервера для fallback на index.html.

### Hash Mode

```javascript
const router = createRouter({
  history: createWebHashHistory(),
  routes
})
```

Использует # в URL, не требует настройки сервера.

### Memory Mode

```javascript
const router = createRouter({
  history: createMemoryHistory(),
  routes
})
```

Для SSR или тестирования.

## Scroll Behavior

```javascript
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      // Восстановить позицию при навигации назад
      return savedPosition
    } else if (to.hash) {
      // Скролл к якорю
      return {
        el: to.hash,
        behavior: 'smooth'
      }
    } else {
      // Скролл наверх
      return { top: 0 }
    }
  }
})
```

## Вопросы для собеседования

### 1. В чём разница между hash mode и history mode?

**Ответ:**
- **Hash mode**: использует `#` в URL, работает без настройки сервера, не поддерживает SSR
- **History mode**: чистые URL, требует настройки сервера (fallback на index.html), поддерживает SSR

### 2. Когда использовать beforeRouteUpdate?

**Ответ:**
Когда компонент переиспользуется при изменении параметров маршрута. Например, `/user/1` → `/user/2` — компонент тот же, но нужно загрузить новые данные.

### 3. Как предотвратить уход со страницы?

**Ответ:**
Использовать `beforeRouteLeave` или `onBeforeRouteLeave`:
```javascript
onBeforeRouteLeave(() => {
  if (hasUnsavedChanges.value) {
    return window.confirm('Уйти?')
  }
})
```

### 4. Зачем нужна ленивая загрузка маршрутов?

**Ответ:**
- Уменьшение размера начального бандла
- Быстрая загрузка приложения
- Загрузка кода только при необходимости
- Улучшение производительности

### 5. Как передать данные между маршрутами?

**Ответ:**
- **Query параметры**: `?tab=profile`
- **Route params**: `/user/:id`
- **State**: `router.push({ state: { data } })`
- **Store** (Pinia/Vuex)
- **Props**: через `props: true` в конфигурации маршрута
