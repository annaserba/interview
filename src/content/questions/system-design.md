---
title: "System Design для Frontend"
description: "Проектирование архитектуры frontend приложений"
category: "Architecture"
difficulty: "hard"
tags: ["system-design", "architecture", "scalability", "performance"]
order: 25
---

## Проектирование крупных приложений

### Архитектурные паттерны

#### Feature-Sliced Design

```
src/
├── app/                    # Инициализация приложения
│   ├── providers/
│   ├── router/
│   └── store/
├── pages/                  # Страницы приложения
│   ├── home/
│   ├── profile/
│   └── settings/
├── widgets/                # Крупные блоки UI
│   ├── header/
│   ├── sidebar/
│   └── footer/
├── features/               # Бизнес-функции
│   ├── auth/
│   ├── cart/
│   └── search/
├── entities/               # Бизнес-сущности
│   ├── user/
│   ├── product/
│   └── order/
└── shared/                 # Переиспользуемый код
    ├── ui/
    ├── lib/
    └── api/
```

#### Модульная архитектура

```typescript
// modules/user/index.ts
export { UserService } from './services/UserService'
export { useUser } from './composables/useUser'
export { UserCard, UserProfile } from './components'
export type { User, UserRole } from './types'

// Использование
import { UserService, useUser } from '@/modules/user'
```

### Масштабирование состояния

```typescript
// stores/modules/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  state: () => ({
    currentUser: null as User | null,
    users: new Map<number, User>(),
    loading: false,
    error: null as Error | null
  }),
  
  getters: {
    getUserById: (state) => (id: number) => state.users.get(id),
    isAuthenticated: (state) => !!state.currentUser
  },
  
  actions: {
    async fetchUser(id: number) {
      if (this.users.has(id)) return this.users.get(id)
      
      this.loading = true
      try {
        const user = await api.getUser(id)
        this.users.set(id, user)
        return user
      } catch (error) {
        this.error = error as Error
        throw error
      } finally {
        this.loading = false
      }
    },
    
    // Нормализация данных
    setUsers(users: User[]) {
      users.forEach(user => this.users.set(user.id, user))
    }
  }
})
```

## Оптимизация производительности

### Code Splitting

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    component: () => import('@/pages/Home.vue')
  },
  {
    path: '/admin',
    component: () => import(
      /* webpackChunkName: "admin" */
      /* webpackPrefetch: true */
      '@/pages/Admin.vue'
    )
  }
]
```

### Виртуализация списков

```vue
<script setup>
import { ref, computed } from 'vue'

const items = ref(Array.from({ length: 10000 }, (_, i) => ({ id: i })))
const containerHeight = 600
const itemHeight = 50
const visibleCount = Math.ceil(containerHeight / itemHeight)

const scrollTop = ref(0)

const visibleItems = computed(() => {
  const startIndex = Math.floor(scrollTop.value / itemHeight)
  const endIndex = startIndex + visibleCount
  return items.value.slice(startIndex, endIndex + 1)
})

const totalHeight = computed(() => items.value.length * itemHeight)
const offsetY = computed(() => Math.floor(scrollTop.value / itemHeight) * itemHeight)

function handleScroll(e: Event) {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}
</script>

<template>
  <div 
    class="virtual-list" 
    :style="{ height: `${containerHeight}px` }"
    @scroll="handleScroll"
  >
    <div :style="{ height: `${totalHeight}px`, position: 'relative' }">
      <div :style="{ transform: `translateY(${offsetY}px)` }">
        <div 
          v-for="item in visibleItems" 
          :key="item.id"
          :style="{ height: `${itemHeight}px` }"
        >
          Item {{ item.id }}
        </div>
      </div>
    </div>
  </div>
</template>
```

### Мемоизация и кэширование

```typescript
// utils/cache.ts
class Cache<T> {
  private cache = new Map<string, { data: T; timestamp: number }>()
  private ttl: number
  
  constructor(ttl = 5 * 60 * 1000) { // 5 минут
    this.ttl = ttl
  }
  
  get(key: string): T | null {
    const item = this.cache.get(key)
    if (!item) return null
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key)
      return null
    }
    
    return item.data
  }
  
  set(key: string, data: T): void {
    this.cache.set(key, { data, timestamp: Date.now() })
  }
  
  clear(): void {
    this.cache.clear()
  }
}

// Использование
const apiCache = new Cache<any>(10 * 60 * 1000)

async function fetchWithCache(url: string) {
  const cached = apiCache.get(url)
  if (cached) return cached
  
  const data = await fetch(url).then(r => r.json())
  apiCache.set(url, data)
  return data
}
```

## Работа с большими данными

### Pagination

```typescript
interface PaginationParams {
  page: number
  pageSize: number
  total?: number
}

export function usePagination<T>(
  fetchFn: (page: number, pageSize: number) => Promise<{ data: T[]; total: number }>
) {
  const items = ref<T[]>([])
  const page = ref(1)
  const pageSize = ref(20)
  const total = ref(0)
  const loading = ref(false)
  
  const totalPages = computed(() => Math.ceil(total.value / pageSize.value))
  const hasNext = computed(() => page.value < totalPages.value)
  const hasPrev = computed(() => page.value > 1)
  
  async function fetch() {
    loading.value = true
    try {
      const result = await fetchFn(page.value, pageSize.value)
      items.value = result.data
      total.value = result.total
    } finally {
      loading.value = false
    }
  }
  
  function nextPage() {
    if (hasNext.value) {
      page.value++
      fetch()
    }
  }
  
  function prevPage() {
    if (hasPrev.value) {
      page.value--
      fetch()
    }
  }
  
  return {
    items,
    page,
    pageSize,
    total,
    totalPages,
    loading,
    hasNext,
    hasPrev,
    fetch,
    nextPage,
    prevPage
  }
}
```

### Infinite Scroll

```typescript
export function useInfiniteScroll<T>(
  fetchFn: (page: number) => Promise<T[]>
) {
  const items = ref<T[]>([])
  const page = ref(1)
  const loading = ref(false)
  const hasMore = ref(true)
  
  async function loadMore() {
    if (loading.value || !hasMore.value) return
    
    loading.value = true
    try {
      const newItems = await fetchFn(page.value)
      if (newItems.length === 0) {
        hasMore.value = false
      } else {
        items.value.push(...newItems)
        page.value++
      }
    } finally {
      loading.value = false
    }
  }
  
  // Автоматическая загрузка при скролле
  function setupObserver(element: HTMLElement) {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          loadMore()
        }
      },
      { threshold: 0.1 }
    )
    
    observer.observe(element)
    return () => observer.disconnect()
  }
  
  return {
    items,
    loading,
    hasMore,
    loadMore,
    setupObserver
  }
}
```

## Обработка ошибок

### Централизованная обработка

```typescript
// services/errorHandler.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode?: number
  ) {
    super(message)
    this.name = 'AppError'
  }
}

export class ErrorHandler {
  private handlers = new Map<string, (error: AppError) => void>()
  
  register(code: string, handler: (error: AppError) => void) {
    this.handlers.set(code, handler)
  }
  
  handle(error: Error) {
    if (error instanceof AppError) {
      const handler = this.handlers.get(error.code)
      if (handler) {
        handler(error)
        return
      }
    }
    
    // Дефолтная обработка
    console.error('Unhandled error:', error)
    this.showNotification('Произошла ошибка', 'error')
  }
  
  private showNotification(message: string, type: 'error' | 'warning') {
    // Показать уведомление пользователю
  }
}

// Использование
const errorHandler = new ErrorHandler()

errorHandler.register('AUTH_FAILED', (error) => {
  router.push('/login')
})

errorHandler.register('NETWORK_ERROR', (error) => {
  // Показать оффлайн индикатор
})
```

### Error Boundary для Vue

```vue
<script setup>
import { onErrorCaptured, ref } from 'vue'

const error = ref<Error | null>(null)

onErrorCaptured((err, instance, info) => {
  error.value = err
  console.error('Error captured:', err, info)
  
  // Отправить в систему мониторинга
  sendToSentry(err, { component: instance?.$options.name, info })
  
  return false // Предотвратить всплытие
})

function reset() {
  error.value = null
}
</script>

<template>
  <div v-if="error" class="error-boundary">
    <h2>Что-то пошло не так</h2>
    <p>{{ error.message }}</p>
    <button @click="reset">Попробовать снова</button>
  </div>
  <slot v-else />
</template>
```

## Реальные задачи с собеседований

### 1. Спроектировать систему уведомлений

```typescript
// Требования:
// - Разные типы уведомлений (success, error, warning, info)
// - Автоматическое закрытие через N секунд
// - Возможность закрыть вручную
// - Максимум 5 уведомлений одновременно
// - Анимация появления/исчезновения

interface Notification {
  id: string
  type: 'success' | 'error' | 'warning' | 'info'
  message: string
  duration?: number
}

export const useNotifications = defineStore('notifications', () => {
  const notifications = ref<Notification[]>([])
  const maxNotifications = 5
  
  function add(notification: Omit<Notification, 'id'>) {
    const id = crypto.randomUUID()
    const newNotification = { id, ...notification }
    
    // Удалить старые если превышен лимит
    if (notifications.value.length >= maxNotifications) {
      notifications.value.shift()
    }
    
    notifications.value.push(newNotification)
    
    // Автоматическое удаление
    const duration = notification.duration ?? 5000
    if (duration > 0) {
      setTimeout(() => remove(id), duration)
    }
    
    return id
  }
  
  function remove(id: string) {
    const index = notifications.value.findIndex(n => n.id === id)
    if (index !== -1) {
      notifications.value.splice(index, 1)
    }
  }
  
  function clear() {
    notifications.value = []
  }
  
  return {
    notifications: readonly(notifications),
    add,
    remove,
    clear
  }
})
```

### 2. Реализовать debounce поиск

```vue
<script setup>
import { ref, watch } from 'vue'
import { useDebounceFn } from '@vueuse/core'

const searchQuery = ref('')
const results = ref([])
const loading = ref(false)

const debouncedSearch = useDebounceFn(async (query: string) => {
  if (!query.trim()) {
    results.value = []
    return
  }
  
  loading.value = true
  try {
    const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`)
    results.value = await response.json()
  } finally {
    loading.value = false
  }
}, 300)

watch(searchQuery, (newQuery) => {
  debouncedSearch(newQuery)
})
</script>

<template>
  <div>
    <input v-model="searchQuery" placeholder="Поиск..." />
    <div v-if="loading">Загрузка...</div>
    <ul v-else>
      <li v-for="result in results" :key="result.id">
        {{ result.title }}
      </li>
    </ul>
  </div>
</template>
```

### 3. Оптимизировать рендеринг большого списка

```vue
<script setup>
import { ref, computed } from 'vue'

// 10000 элементов
const allItems = ref(
  Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    title: `Item ${i}`,
    description: `Description for item ${i}`
  }))
)

// Виртуализация
const ITEM_HEIGHT = 60
const VISIBLE_COUNT = 20
const scrollTop = ref(0)

const startIndex = computed(() => 
  Math.floor(scrollTop.value / ITEM_HEIGHT)
)

const endIndex = computed(() => 
  Math.min(startIndex.value + VISIBLE_COUNT, allItems.value.length)
)

const visibleItems = computed(() => 
  allItems.value.slice(startIndex.value, endIndex.value)
)

const totalHeight = computed(() => 
  allItems.value.length * ITEM_HEIGHT
)

const offsetY = computed(() => 
  startIndex.value * ITEM_HEIGHT
)

function handleScroll(e: Event) {
  scrollTop.value = (e.target as HTMLElement).scrollTop
}
</script>

<template>
  <div 
    class="list-container" 
    @scroll="handleScroll"
    style="height: 600px; overflow-y: auto;"
  >
    <div :style="{ height: `${totalHeight}px`, position: 'relative' }">
      <div :style="{ transform: `translateY(${offsetY}px)` }">
        <div 
          v-for="item in visibleItems" 
          :key="item.id"
          :style="{ height: `${ITEM_HEIGHT}px` }"
          class="list-item"
        >
          <h3>{{ item.title }}</h3>
          <p>{{ item.description }}</p>
        </div>
      </div>
    </div>
  </div>
</template>
```

## Вопросы для собеседования

### 1. Как бы вы спроектировали Dashboard с real-time обновлениями?

**Ответ:**
- WebSocket соединение для real-time данных
- Оптимистичные обновления UI
- Кэширование данных в Pinia store
- Виртуализация для больших списков
- Debounce для частых обновлений
- Error handling и reconnection logic

### 2. Как оптимизировать приложение с 100+ компонентами?

**Ответ:**
- Code splitting по роутам
- Lazy loading компонентов
- Tree shaking неиспользуемого кода
- Compression (gzip/brotli)
- CDN для статики
- Service Worker для кэширования
- Анализ бандла (webpack-bundle-analyzer)

### 3. Как обрабатывать состояние в большом приложении?

**Ответ:**
- Модульная структура stores
- Нормализация данных (избегать дублирования)
- Computed properties для производных данных
- Persist критичные данные в localStorage
- Оптимистичные обновления
- Централизованная обработка ошибок

### 4. Как реализовать offline-first приложение?

**Ответ:**
- Service Worker для кэширования
- IndexedDB для хранения данных
- Sync API для фоновой синхронизации
- Очередь запросов при offline
- Индикатор состояния сети
- Conflict resolution при синхронизации

### 5. Как масштабировать frontend команду?

**Ответ:**
- Модульная архитектура (Feature-Sliced Design)
- Shared UI библиотека компонентов
- Code style guide и ESLint правила
- CI/CD pipeline
- Code review процесс
- Документация и Storybook
- Монорепозиторий (Nx, Turborepo)
