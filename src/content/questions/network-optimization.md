---
title: "Оптимизация сетевых запросов"
description: "Стратегии оптимизации загрузки данных и работы с API"
category: "Performance"
difficulty: "hard"
tags: ["network", "optimization", "caching", "performance"]
order: 29
---

## Стратегии кэширования

### HTTP Cache Headers

```javascript
// Настройка кэширования на сервере
app.get('/api/static-data', (req, res) => {
  res.set({
    'Cache-Control': 'public, max-age=31536000', // 1 год
    'ETag': generateETag(data),
    'Last-Modified': new Date(data.updatedAt).toUTCString()
  })
  res.json(data)
})

// Для часто меняющихся данных
app.get('/api/dynamic-data', (req, res) => {
  res.set({
    'Cache-Control': 'private, max-age=300', // 5 минут
    'ETag': generateETag(data)
  })
  res.json(data)
})

// Без кэширования
app.get('/api/sensitive-data', (req, res) => {
  res.set({
    'Cache-Control': 'no-store, no-cache, must-revalidate',
    'Pragma': 'no-cache'
  })
  res.json(data)
})
```

### In-Memory Cache

```typescript
class ApiCache {
  private cache = new Map<string, {
    data: any
    timestamp: number
    etag?: string
  }>()
  
  constructor(private ttl = 5 * 60 * 1000) {} // 5 минут
  
  get(key: string) {
    const item = this.cache.get(key)
    if (!item) return null
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key)
      return null
    }
    
    return item.data
  }
  
  set(key: string, data: any, etag?: string) {
    this.cache.set(key, {
      data,
      timestamp: Date.now(),
      etag
    })
  }
  
  getETag(key: string) {
    return this.cache.get(key)?.etag
  }
  
  clear() {
    this.cache.clear()
  }
  
  delete(key: string) {
    this.cache.delete(key)
  }
}

// Использование
const apiCache = new ApiCache(10 * 60 * 1000) // 10 минут

async function fetchWithCache(url: string) {
  // Проверить кэш
  const cached = apiCache.get(url)
  if (cached) {
    console.log('Cache hit:', url)
    return cached
  }
  
  // Запрос с ETag
  const etag = apiCache.getETag(url)
  const headers: HeadersInit = {}
  if (etag) {
    headers['If-None-Match'] = etag
  }
  
  const response = await fetch(url, { headers })
  
  // 304 Not Modified
  if (response.status === 304) {
    const cached = apiCache.get(url)
    return cached
  }
  
  const data = await response.json()
  const newETag = response.headers.get('ETag')
  
  apiCache.set(url, data, newETag || undefined)
  return data
}
```

### IndexedDB для долговременного хранения

```typescript
class IndexedDBCache {
  private db: IDBDatabase | null = null
  private dbName = 'api-cache'
  private storeName = 'responses'
  
  async init() {
    return new Promise<void>((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1)
      
      request.onerror = () => reject(request.error)
      request.onsuccess = () => {
        this.db = request.result
        resolve()
      }
      
      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result
        if (!db.objectStoreNames.contains(this.storeName)) {
          const store = db.createObjectStore(this.storeName, { keyPath: 'url' })
          store.createIndex('timestamp', 'timestamp')
        }
      }
    })
  }
  
  async get(url: string) {
    if (!this.db) await this.init()
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readonly')
      const store = transaction.objectStore(this.storeName)
      const request = store.get(url)
      
      request.onsuccess = () => {
        const result = request.result
        if (result && Date.now() - result.timestamp < result.ttl) {
          resolve(result.data)
        } else {
          resolve(null)
        }
      }
      request.onerror = () => reject(request.error)
    })
  }
  
  async set(url: string, data: any, ttl = 24 * 60 * 60 * 1000) {
    if (!this.db) await this.init()
    
    return new Promise<void>((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readwrite')
      const store = transaction.objectStore(this.storeName)
      const request = store.put({
        url,
        data,
        timestamp: Date.now(),
        ttl
      })
      
      request.onsuccess = () => resolve()
      request.onerror = () => reject(request.error)
    })
  }
  
  async clear() {
    if (!this.db) await this.init()
    
    return new Promise<void>((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readwrite')
      const store = transaction.objectStore(this.storeName)
      const request = store.clear()
      
      request.onsuccess = () => resolve()
      request.onerror = () => reject(request.error)
    })
  }
}
```

## Request Batching

### Объединение запросов

```typescript
class RequestBatcher {
  private queue: Array<{
    id: string
    resolve: (data: any) => void
    reject: (error: any) => void
  }> = []
  
  private timeoutId: number | null = null
  private batchDelay = 50 // ms
  
  constructor(
    private batchFn: (ids: string[]) => Promise<Record<string, any>>
  ) {}
  
  request(id: string): Promise<any> {
    return new Promise((resolve, reject) => {
      this.queue.push({ id, resolve, reject })
      
      if (this.timeoutId) {
        clearTimeout(this.timeoutId)
      }
      
      this.timeoutId = setTimeout(() => {
        this.flush()
      }, this.batchDelay) as unknown as number
    })
  }
  
  private async flush() {
    if (this.queue.length === 0) return
    
    const batch = this.queue.splice(0)
    const ids = batch.map(item => item.id)
    
    try {
      const results = await this.batchFn(ids)
      
      batch.forEach(item => {
        if (results[item.id]) {
          item.resolve(results[item.id])
        } else {
          item.reject(new Error(`No data for id: ${item.id}`))
        }
      })
    } catch (error) {
      batch.forEach(item => item.reject(error))
    }
  }
}

// Использование
const userBatcher = new RequestBatcher(async (ids) => {
  const response = await fetch('/api/users/batch', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ ids })
  })
  return response.json()
})

// Вместо 3 запросов будет 1
const user1 = await userBatcher.request('1')
const user2 = await userBatcher.request('2')
const user3 = await userBatcher.request('3')
```

## Request Deduplication

### Устранение дублирующихся запросов

```typescript
class RequestDeduplicator {
  private pending = new Map<string, Promise<any>>()
  
  async fetch(url: string, options?: RequestInit): Promise<any> {
    const key = this.getCacheKey(url, options)
    
    // Если запрос уже выполняется, вернуть тот же Promise
    if (this.pending.has(key)) {
      console.log('Deduplicating request:', url)
      return this.pending.get(key)!
    }
    
    // Создать новый запрос
    const promise = fetch(url, options)
      .then(r => r.json())
      .finally(() => {
        // Удалить из pending после завершения
        this.pending.delete(key)
      })
    
    this.pending.set(key, promise)
    return promise
  }
  
  private getCacheKey(url: string, options?: RequestInit): string {
    return `${options?.method || 'GET'}:${url}`
  }
}

// Использование
const deduplicator = new RequestDeduplicator()

// Только один запрос будет выполнен
const [data1, data2, data3] = await Promise.all([
  deduplicator.fetch('/api/users'),
  deduplicator.fetch('/api/users'),
  deduplicator.fetch('/api/users')
])
```

## Prefetching и Preloading

### Link Prefetch

```vue
<script setup>
import { onMounted } from 'vue'

function prefetchRoute(path: string) {
  const link = document.createElement('link')
  link.rel = 'prefetch'
  link.href = path
  document.head.appendChild(link)
}

onMounted(() => {
  // Предзагрузка вероятных маршрутов
  prefetchRoute('/about')
  prefetchRoute('/contact')
})
</script>
```

### Intersection Observer для ленивой загрузки

```vue
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const imageRef = ref<HTMLImageElement>()
const isLoaded = ref(false)

let observer: IntersectionObserver | null = null

onMounted(() => {
  observer = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && !isLoaded.value) {
          const img = entry.target as HTMLImageElement
          img.src = img.dataset.src!
          isLoaded.value = true
          observer?.unobserve(img)
        }
      })
    },
    {
      rootMargin: '50px' // Загружать за 50px до появления
    }
  )
  
  if (imageRef.value) {
    observer.observe(imageRef.value)
  }
})

onUnmounted(() => {
  observer?.disconnect()
})
</script>

<template>
  <img
    ref="imageRef"
    :data-src="imageSrc"
    alt="Lazy loaded image"
  />
</template>
```

### Predictive Prefetching

```typescript
class PredictivePrefetcher {
  private history: string[] = []
  private predictions = new Map<string, string[]>()
  private prefetched = new Set<string>()
  
  recordNavigation(from: string, to: string) {
    this.history.push(to)
    
    // Обновить предсказания
    if (!this.predictions.has(from)) {
      this.predictions.set(from, [])
    }
    this.predictions.get(from)!.push(to)
  }
  
  prefetchNext(currentRoute: string) {
    const predictions = this.predictions.get(currentRoute)
    if (!predictions) return
    
    // Найти самый частый следующий маршрут
    const frequency = new Map<string, number>()
    predictions.forEach(route => {
      frequency.set(route, (frequency.get(route) || 0) + 1)
    })
    
    const sorted = [...frequency.entries()]
      .sort((a, b) => b[1] - a[1])
    
    // Предзагрузить топ-3
    sorted.slice(0, 3).forEach(([route]) => {
      if (!this.prefetched.has(route)) {
        this.prefetch(route)
        this.prefetched.add(route)
      }
    })
  }
  
  private prefetch(route: string) {
    const link = document.createElement('link')
    link.rel = 'prefetch'
    link.href = route
    document.head.appendChild(link)
  }
}
```

## GraphQL оптимизации

### Query Batching

```typescript
import { ApolloClient, InMemoryCache, HttpLink } from '@apollo/client'
import { BatchHttpLink } from '@apollo/client/link/batch-http'

const client = new ApolloClient({
  link: new BatchHttpLink({
    uri: '/graphql',
    batchMax: 10, // Максимум запросов в батче
    batchInterval: 20 // Интервал батчинга в ms
  }),
  cache: new InMemoryCache()
})
```

### Persisted Queries

```typescript
import { createPersistedQueryLink } from '@apollo/client/link/persisted-queries'
import { sha256 } from 'crypto-hash'

const link = createPersistedQueryLink({
  sha256,
  useGETForHashedQueries: true
}).concat(httpLink)

// Первый запрос: отправляет hash
// Если сервер не знает hash, отправляет полный query
// Последующие запросы: только hash
```

## Service Worker для кэширования

```javascript
// service-worker.js
const CACHE_NAME = 'api-cache-v1'
const API_CACHE_DURATION = 5 * 60 * 1000 // 5 минут

self.addEventListener('fetch', (event) => {
  const { request } = event
  
  // Кэшировать только GET запросы к API
  if (request.method !== 'GET' || !request.url.includes('/api/')) {
    return
  }
  
  event.respondWith(
    caches.open(CACHE_NAME).then(async (cache) => {
      const cached = await cache.match(request)
      
      if (cached) {
        const cachedTime = new Date(cached.headers.get('sw-cache-time') || 0)
        const now = Date.now()
        
        // Если кэш свежий, вернуть его
        if (now - cachedTime.getTime() < API_CACHE_DURATION) {
          console.log('Serving from cache:', request.url)
          return cached
        }
      }
      
      // Запросить с сервера
      try {
        const response = await fetch(request)
        
        // Сохранить в кэш
        if (response.ok) {
          const clonedResponse = response.clone()
          const headers = new Headers(clonedResponse.headers)
          headers.set('sw-cache-time', new Date().toISOString())
          
          const cachedResponse = new Response(
            await clonedResponse.blob(),
            {
              status: clonedResponse.status,
              statusText: clonedResponse.statusText,
              headers
            }
          )
          
          cache.put(request, cachedResponse)
        }
        
        return response
      } catch (error) {
        // Если offline, вернуть старый кэш
        if (cached) {
          console.log('Offline: serving stale cache:', request.url)
          return cached
        }
        throw error
      }
    })
  )
})
```

## Вопросы для собеседования

### 1. Объясните разницу между Cache-Control и ETag

**Ответ:**
- **Cache-Control**: определяет как долго кэшировать (max-age)
- **ETag**: уникальный идентификатор версии ресурса для условных запросов

Cache-Control предотвращает запросы, ETag делает их условными (304).

### 2. Что такое request batching и когда его использовать?

**Ответ:**
Объединение нескольких запросов в один. Используется когда:
- Много мелких запросов к одному API
- GraphQL запросы
- Загрузка связанных данных

Уменьшает overhead HTTP запросов.

### 3. Как реализовать offline-first стратегию?

**Ответ:**
- Service Worker для кэширования
- IndexedDB для данных
- Background Sync для отложенных запросов
- Очередь запросов при offline
- Индикатор состояния сети

### 4. Что такое prefetching vs preloading?

**Ответ:**
- **Prefetch**: загрузка ресурсов для будущей навигации (низкий приоритет)
- **Preload**: загрузка критичных ресурсов текущей страницы (высокий приоритет)

### 5. Как оптимизировать GraphQL запросы?

**Ответ:**
- Query batching
- Persisted queries
- DataLoader для N+1 проблемы
- Pagination
- Field-level caching
- Automatic persisted queries (APQ)
