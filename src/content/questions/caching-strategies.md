---
title: "Стратегии кэширования"
description: "Кэширование данных на клиенте и сервере для оптимизации производительности"
category: "Performance"
difficulty: "hard"
tags: ["caching", "performance", "optimization", "storage"]
order: 32
---

## Что такое кэширование?

Кэширование — это сохранение копии данных для быстрого доступа в будущем. Уменьшает нагрузку на сервер и ускоряет загрузку.

### Преимущества
- ⚡ Быстрая загрузка данных
- 📉 Снижение нагрузки на сервер
- 💰 Экономия трафика
- 🌐 Работа offline

### Недостатки
- 🔄 Проблема устаревших данных
- 💾 Использование памяти
- 🔧 Сложность инвалидации

## Уровни кэширования

```
┌─────────────────────────────────────┐
│   Browser Cache (Memory/Disk)      │
├─────────────────────────────────────┤
│   Service Worker Cache              │
├─────────────────────────────────────┤
│   CDN Cache                         │
├─────────────────────────────────────┤
│   Reverse Proxy (Nginx, Varnish)   │
├─────────────────────────────────────┤
│   Application Cache (Redis)         │
├─────────────────────────────────────┤
│   Database Query Cache              │
└─────────────────────────────────────┘
```

## HTTP кэширование

### Cache-Control заголовки

```javascript
// Сервер устанавливает заголовки
app.get('/api/data', (req, res) => {
  // Публичный кэш на 1 час
  res.set('Cache-Control', 'public, max-age=3600')
  
  // Приватный кэш (только браузер)
  res.set('Cache-Control', 'private, max-age=3600')
  
  // Не кэшировать
  res.set('Cache-Control', 'no-store')
  
  // Всегда проверять с сервером
  res.set('Cache-Control', 'no-cache')
  
  // Кэш с ревалидацией
  res.set('Cache-Control', 'max-age=3600, must-revalidate')
  
  res.json({ data: 'some data' })
})
```

### Директивы Cache-Control

| Директива | Описание |
|-----------|----------|
| `public` | Можно кэшировать везде (CDN, прокси) |
| `private` | Только в браузере пользователя |
| `no-cache` | Проверять с сервером перед использованием |
| `no-store` | Не кэшировать вообще |
| `max-age=N` | Время жизни в секундах |
| `s-maxage=N` | Время жизни для shared кэша (CDN) |
| `must-revalidate` | Проверять после истечения |
| `immutable` | Никогда не изменится (для версионированных файлов) |

### ETag (Entity Tag)

```javascript
// Сервер генерирует ETag
const crypto = require('crypto')

app.get('/api/data', (req, res) => {
  const data = { id: 1, name: 'John' }
  const etag = crypto
    .createHash('md5')
    .update(JSON.stringify(data))
    .digest('hex')
  
  // Проверить If-None-Match от клиента
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end() // Not Modified
  }
  
  res.set('ETag', etag)
  res.set('Cache-Control', 'no-cache')
  res.json(data)
})

// Клиент
async function fetchWithETag(url, etag) {
  const headers = {}
  if (etag) {
    headers['If-None-Match'] = etag
  }
  
  const response = await fetch(url, { headers })
  
  if (response.status === 304) {
    console.log('Using cached version')
    return getCachedData(url)
  }
  
  const data = await response.json()
  const newETag = response.headers.get('ETag')
  
  cacheData(url, data, newETag)
  return data
}
```

### Last-Modified

```javascript
// Сервер
app.get('/api/data', (req, res) => {
  const data = getDataFromDB()
  const lastModified = new Date(data.updatedAt)
  
  // Проверить If-Modified-Since
  const ifModifiedSince = req.headers['if-modified-since']
  if (ifModifiedSince && new Date(ifModifiedSince) >= lastModified) {
    return res.status(304).end()
  }
  
  res.set('Last-Modified', lastModified.toUTCString())
  res.set('Cache-Control', 'no-cache')
  res.json(data)
})

// Клиент
async function fetchWithLastModified(url, lastModified) {
  const headers = {}
  if (lastModified) {
    headers['If-Modified-Since'] = lastModified
  }
  
  const response = await fetch(url, { headers })
  
  if (response.status === 304) {
    return getCachedData(url)
  }
  
  const data = await response.json()
  const newLastModified = response.headers.get('Last-Modified')
  
  cacheData(url, data, newLastModified)
  return data
}
```

## In-Memory кэш

### Простой кэш с TTL

```javascript
class MemoryCache {
  constructor() {
    this.cache = new Map()
  }
  
  set(key, value, ttl = 60000) {
    const expiresAt = Date.now() + ttl
    this.cache.set(key, { value, expiresAt })
  }
  
  get(key) {
    const item = this.cache.get(key)
    
    if (!item) return null
    
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key)
      return null
    }
    
    return item.value
  }
  
  has(key) {
    return this.get(key) !== null
  }
  
  delete(key) {
    this.cache.delete(key)
  }
  
  clear() {
    this.cache.clear()
  }
  
  // Автоматическая очистка устаревших записей
  startCleanup(interval = 60000) {
    this.cleanupInterval = setInterval(() => {
      const now = Date.now()
      for (const [key, item] of this.cache.entries()) {
        if (now > item.expiresAt) {
          this.cache.delete(key)
        }
      }
    }, interval)
  }
  
  stopCleanup() {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval)
    }
  }
}

// Использование
const cache = new MemoryCache()
cache.startCleanup()

cache.set('user:1', { id: 1, name: 'John' }, 5000) // 5 секунд

const user = cache.get('user:1')
console.log(user) // { id: 1, name: 'John' }

setTimeout(() => {
  console.log(cache.get('user:1')) // null (истёк TTL)
}, 6000)
```

### LRU Cache (Least Recently Used)

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity
    this.cache = new Map()
  }
  
  get(key) {
    if (!this.cache.has(key)) return null
    
    // Переместить в конец (самый свежий)
    const value = this.cache.get(key)
    this.cache.delete(key)
    this.cache.set(key, value)
    
    return value
  }
  
  set(key, value) {
    // Удалить если уже есть
    if (this.cache.has(key)) {
      this.cache.delete(key)
    }
    
    // Добавить в конец
    this.cache.set(key, value)
    
    // Удалить самый старый если превышен лимит
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value
      this.cache.delete(firstKey)
    }
  }
  
  has(key) {
    return this.cache.has(key)
  }
  
  delete(key) {
    this.cache.delete(key)
  }
  
  clear() {
    this.cache.clear()
  }
  
  get size() {
    return this.cache.size
  }
}

// Использование
const lru = new LRUCache(3)

lru.set('a', 1)
lru.set('b', 2)
lru.set('c', 3)
console.log(lru.size) // 3

lru.set('d', 4) // 'a' будет удалено
console.log(lru.get('a')) // null
console.log(lru.get('b')) // 2
```

## LocalStorage кэш

```javascript
class LocalStorageCache {
  constructor(prefix = 'cache:') {
    this.prefix = prefix
  }
  
  set(key, value, ttl = null) {
    const item = {
      value,
      timestamp: Date.now(),
      ttl
    }
    
    try {
      localStorage.setItem(
        this.prefix + key,
        JSON.stringify(item)
      )
    } catch (error) {
      // Quota exceeded - очистить старые записи
      this.cleanup()
      try {
        localStorage.setItem(
          this.prefix + key,
          JSON.stringify(item)
        )
      } catch (e) {
        console.error('LocalStorage full:', e)
      }
    }
  }
  
  get(key) {
    const itemStr = localStorage.getItem(this.prefix + key)
    
    if (!itemStr) return null
    
    try {
      const item = JSON.parse(itemStr)
      
      // Проверить TTL
      if (item.ttl && Date.now() - item.timestamp > item.ttl) {
        this.delete(key)
        return null
      }
      
      return item.value
    } catch (error) {
      this.delete(key)
      return null
    }
  }
  
  delete(key) {
    localStorage.removeItem(this.prefix + key)
  }
  
  clear() {
    const keys = Object.keys(localStorage)
    keys.forEach(key => {
      if (key.startsWith(this.prefix)) {
        localStorage.removeItem(key)
      }
    })
  }
  
  cleanup() {
    const keys = Object.keys(localStorage)
    const now = Date.now()
    
    keys.forEach(key => {
      if (!key.startsWith(this.prefix)) return
      
      try {
        const item = JSON.parse(localStorage.getItem(key))
        if (item.ttl && now - item.timestamp > item.ttl) {
          localStorage.removeItem(key)
        }
      } catch (e) {
        localStorage.removeItem(key)
      }
    })
  }
}

// Использование
const cache = new LocalStorageCache()

cache.set('user', { id: 1, name: 'John' }, 60000) // 1 минута
const user = cache.get('user')
```

## IndexedDB кэш

```javascript
class IndexedDBCache {
  constructor(dbName = 'app-cache', storeName = 'cache') {
    this.dbName = dbName
    this.storeName = storeName
    this.db = null
  }
  
  async init() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1)
      
      request.onerror = () => reject(request.error)
      request.onsuccess = () => {
        this.db = request.result
        resolve()
      }
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result
        if (!db.objectStoreNames.contains(this.storeName)) {
          const store = db.createObjectStore(this.storeName, { keyPath: 'key' })
          store.createIndex('timestamp', 'timestamp')
          store.createIndex('expiresAt', 'expiresAt')
        }
      }
    })
  }
  
  async set(key, value, ttl = null) {
    if (!this.db) await this.init()
    
    const timestamp = Date.now()
    const expiresAt = ttl ? timestamp + ttl : null
    
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite')
      const store = transaction.objectStore(this.storeName)
      const request = store.put({
        key,
        value,
        timestamp,
        expiresAt
      })
      
      request.onsuccess = () => resolve()
      request.onerror = () => reject(request.error)
    })
  }
  
  async get(key) {
    if (!this.db) await this.init()
    
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readonly')
      const store = transaction.objectStore(this.storeName)
      const request = store.get(key)
      
      request.onsuccess = () => {
        const item = request.result
        
        if (!item) {
          resolve(null)
          return
        }
        
        // Проверить TTL
        if (item.expiresAt && Date.now() > item.expiresAt) {
          this.delete(key)
          resolve(null)
          return
        }
        
        resolve(item.value)
      }
      
      request.onerror = () => reject(request.error)
    })
  }
  
  async delete(key) {
    if (!this.db) await this.init()
    
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite')
      const store = transaction.objectStore(this.storeName)
      const request = store.delete(key)
      
      request.onsuccess = () => resolve()
      request.onerror = () => reject(request.error)
    })
  }
  
  async clear() {
    if (!this.db) await this.init()
    
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite')
      const store = transaction.objectStore(this.storeName)
      const request = store.clear()
      
      request.onsuccess = () => resolve()
      request.onerror = () => reject(request.error)
    })
  }
  
  async cleanup() {
    if (!this.db) await this.init()
    
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction([this.storeName], 'readwrite')
      const store = transaction.objectStore(this.storeName)
      const index = store.index('expiresAt')
      const now = Date.now()
      
      const request = index.openCursor()
      
      request.onsuccess = (event) => {
        const cursor = event.target.result
        if (cursor) {
          if (cursor.value.expiresAt && cursor.value.expiresAt < now) {
            cursor.delete()
          }
          cursor.continue()
        } else {
          resolve()
        }
      }
      
      request.onerror = () => reject(request.error)
    })
  }
}

// Использование
const cache = new IndexedDBCache()

await cache.set('user:1', { id: 1, name: 'John' }, 60000)
const user = await cache.get('user:1')

// Периодическая очистка
setInterval(() => cache.cleanup(), 60000)
```

## Service Worker кэш

```javascript
// service-worker.js
const CACHE_NAME = 'app-cache-v1'
const CACHE_URLS = [
  '/',
  '/styles.css',
  '/script.js',
  '/logo.png'
]

// Установка - кэшировать статические файлы
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(CACHE_URLS)
    })
  )
})

// Активация - удалить старые кэши
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_NAME) {
            return caches.delete(cacheName)
          }
        })
      )
    })
  )
})

// Fetch - стратегии кэширования
self.addEventListener('fetch', (event) => {
  const { request } = event
  const url = new URL(request.url)
  
  // Стратегия для API запросов
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request))
  }
  // Стратегия для статических файлов
  else {
    event.respondWith(cacheFirst(request))
  }
})

// Cache First - сначала кэш, потом сеть
async function cacheFirst(request) {
  const cache = await caches.open(CACHE_NAME)
  const cached = await cache.match(request)
  
  if (cached) {
    return cached
  }
  
  try {
    const response = await fetch(request)
    cache.put(request, response.clone())
    return response
  } catch (error) {
    return new Response('Offline', { status: 503 })
  }
}

// Network First - сначала сеть, потом кэш
async function networkFirst(request) {
  const cache = await caches.open(CACHE_NAME)
  
  try {
    const response = await fetch(request)
    cache.put(request, response.clone())
    return response
  } catch (error) {
    const cached = await cache.match(request)
    return cached || new Response('Offline', { status: 503 })
  }
}

// Stale While Revalidate - кэш + обновление в фоне
async function staleWhileRevalidate(request) {
  const cache = await caches.open(CACHE_NAME)
  const cached = await cache.match(request)
  
  const fetchPromise = fetch(request).then((response) => {
    cache.put(request, response.clone())
    return response
  })
  
  return cached || fetchPromise
}
```

## Стратегии кэширования

### 1. Cache First (Cache Falling Back to Network)

```javascript
async function cacheFirst(url) {
  const cached = await cache.get(url)
  if (cached) return cached
  
  const response = await fetch(url)
  cache.set(url, response)
  return response
}
```

**Когда использовать:**
- Статические ресурсы (CSS, JS, изображения)
- Редко меняющиеся данные
- Offline-first приложения

### 2. Network First (Network Falling Back to Cache)

```javascript
async function networkFirst(url) {
  try {
    const response = await fetch(url)
    cache.set(url, response)
    return response
  } catch (error) {
    const cached = await cache.get(url)
    if (cached) return cached
    throw error
  }
}
```

**Когда использовать:**
- Часто меняющиеся данные
- API запросы
- Когда важна актуальность

### 3. Stale While Revalidate

```javascript
async function staleWhileRevalidate(url) {
  const cached = await cache.get(url)
  
  // Обновить в фоне
  fetch(url).then(response => {
    cache.set(url, response)
  })
  
  return cached || fetch(url)
}
```

**Когда использовать:**
- Баланс между скоростью и актуальностью
- Новостные ленты
- Социальные сети

### 4. Network Only

```javascript
async function networkOnly(url) {
  return fetch(url)
}
```

**Когда использовать:**
- Критичные данные
- Платёжные операции
- Аналитика

### 5. Cache Only

```javascript
async function cacheOnly(url) {
  return cache.get(url)
}
```

**Когда использовать:**
- Полностью offline приложения
- Предзагруженные данные

## Инвалидация кэша

### Time-based (TTL)

```javascript
class CacheWithTTL {
  set(key, value, ttl) {
    const expiresAt = Date.now() + ttl
    this.cache.set(key, { value, expiresAt })
  }
  
  get(key) {
    const item = this.cache.get(key)
    if (!item) return null
    
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key)
      return null
    }
    
    return item.value
  }
}
```

### Event-based

```javascript
// Инвалидация при изменении данных
async function updateUser(userId, data) {
  await api.updateUser(userId, data)
  
  // Инвалидировать связанные кэши
  cache.delete(`user:${userId}`)
  cache.delete(`user:${userId}:posts`)
  cache.delete('users:list')
}
```

### Tag-based

```javascript
class TaggedCache {
  constructor() {
    this.cache = new Map()
    this.tags = new Map()
  }
  
  set(key, value, tags = []) {
    this.cache.set(key, value)
    
    tags.forEach(tag => {
      if (!this.tags.has(tag)) {
        this.tags.set(tag, new Set())
      }
      this.tags.get(tag).add(key)
    })
  }
  
  invalidateTag(tag) {
    const keys = this.tags.get(tag)
    if (keys) {
      keys.forEach(key => this.cache.delete(key))
      this.tags.delete(tag)
    }
  }
}

// Использование
const cache = new TaggedCache()

cache.set('user:1', userData, ['user', 'user:1'])
cache.set('user:1:posts', posts, ['user:1', 'posts'])

// Инвалидировать все кэши пользователя
cache.invalidateTag('user:1')
```

## Вопросы для собеседования

### 1. Какие стратегии кэширования вы знаете?

**Ответ:**
- **Cache First**: сначала кэш, потом сеть (статика)
- **Network First**: сначала сеть, потом кэш (API)
- **Stale While Revalidate**: кэш + обновление в фоне
- **Network Only**: только сеть (критичные данные)
- **Cache Only**: только кэш (offline)

### 2. В чём разница между ETag и Last-Modified?

**Ответ:**
- **ETag**: хэш содержимого, точнее
- **Last-Modified**: дата изменения, менее точно (секунды)

ETag лучше для часто меняющихся файлов.

### 3. Что такое LRU кэш?

**Ответ:**
Least Recently Used — удаляет самые давно использованные элементы при переполнении. Реализуется через Map (сохраняет порядок вставки).

### 4. Как работает Service Worker кэш?

**Ответ:**
Service Worker перехватывает fetch запросы и может:
- Вернуть из кэша
- Запросить с сервера
- Комбинировать стратегии

Работает даже offline.

### 5. Когда использовать LocalStorage vs IndexedDB?

**Ответ:**
- **LocalStorage**: простые данные, <5MB, синхронный
- **IndexedDB**: большие объёмы, структурированные данные, асинхронный

IndexedDB для кэширования API ответов.

### 6. Что такое Cache Busting?

**Ответ:**
Техника принудительного обновления кэша через изменение URL:
```
script.js?v=1.2.3
script.abc123.js (hash в имени)
```

### 7. Как реализовать инвалидацию кэша?

**Ответ:**
- **TTL**: время жизни
- **Event-based**: при изменении данных
- **Tag-based**: группировка по тегам
- **Manual**: явное удаление

### 8. В чём проблема кэширования?

**Ответ:**
- Устаревшие данные
- Использование памяти
- Сложность инвалидации
- Cache stampede (одновременные запросы)

Решение: правильный TTL, стратегии, мониторинг.
