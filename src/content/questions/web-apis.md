---
title: "Web APIs"
description: "Современные Web APIs: Fetch, Storage, Geolocation, Notifications"
category: "JavaScript"
difficulty: "medium"
tags: ["web-api", "fetch", "storage", "geolocation"]
order: 24
---

## Fetch API

### Базовые запросы

```javascript
// GET запрос
fetch('https://api.example.com/users')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error))

// Async/await
async function getUsers() {
  try {
    const response = await fetch('https://api.example.com/users')
    const data = await response.json()
    return data
  } catch (error) {
    console.error('Error:', error)
  }
}
```

### POST запрос

```javascript
async function createUser(userData) {
  const response = await fetch('https://api.example.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer token'
    },
    body: JSON.stringify(userData)
  })
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`)
  }
  
  return await response.json()
}
```

### Обработка ошибок

```javascript
async function fetchWithErrorHandling(url) {
  try {
    const response = await fetch(url)
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }
    
    const data = await response.json()
    return data
  } catch (error) {
    if (error instanceof TypeError) {
      console.error('Network error:', error)
    } else {
      console.error('Error:', error)
    }
    throw error
  }
}
```

## LocalStorage и SessionStorage

```javascript
// LocalStorage (сохраняется после закрытия браузера)
localStorage.setItem('user', JSON.stringify({ id: 1, name: 'John' }))
const user = JSON.parse(localStorage.getItem('user'))
localStorage.removeItem('user')
localStorage.clear()

// SessionStorage (удаляется после закрытия вкладки)
sessionStorage.setItem('token', 'abc123')
const token = sessionStorage.getItem('token')

// Проверка доступности
if (typeof Storage !== 'undefined') {
  // Storage доступен
}

// Обработка события storage (синхронизация между вкладками)
window.addEventListener('storage', (e) => {
  console.log('Key:', e.key)
  console.log('Old value:', e.oldValue)
  console.log('New value:', e.newValue)
})
```

## IndexedDB

```javascript
// Открытие базы данных
const request = indexedDB.open('MyDatabase', 1)

request.onupgradeneeded = (event) => {
  const db = event.target.result
  const objectStore = db.createObjectStore('users', { keyPath: 'id' })
  objectStore.createIndex('email', 'email', { unique: true })
}

request.onsuccess = (event) => {
  const db = event.target.result
  
  // Добавление данных
  const transaction = db.transaction(['users'], 'readwrite')
  const objectStore = transaction.objectStore('users')
  objectStore.add({ id: 1, name: 'John', email: 'john@example.com' })
  
  // Чтение данных
  const getRequest = objectStore.get(1)
  getRequest.onsuccess = () => {
    console.log(getRequest.result)
  }
}
```

## Geolocation API

```javascript
if ('geolocation' in navigator) {
  navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords
      console.log(`Lat: ${latitude}, Lon: ${longitude}`)
    },
    (error) => {
      console.error('Error:', error.message)
    },
    {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0
    }
  )
  
  // Отслеживание позиции
  const watchId = navigator.geolocation.watchPosition((position) => {
    console.log('Position updated:', position.coords)
  })
  
  // Остановить отслеживание
  navigator.geolocation.clearWatch(watchId)
}
```

## Notifications API

```javascript
// Запрос разрешения
async function requestNotificationPermission() {
  const permission = await Notification.requestPermission()
  return permission === 'granted'
}

// Показать уведомление
function showNotification(title, options) {
  if (Notification.permission === 'granted') {
    new Notification(title, {
      body: 'Notification body',
      icon: '/icon.png',
      badge: '/badge.png',
      tag: 'unique-tag',
      requireInteraction: false
    })
  }
}

// Обработка кликов
const notification = new Notification('Title')
notification.onclick = () => {
  window.focus()
  notification.close()
}
```

## Intersection Observer

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element is visible')
      entry.target.classList.add('visible')
    }
  })
}, {
  threshold: 0.5,
  rootMargin: '0px'
})

// Наблюдать за элементом
const element = document.querySelector('.lazy-load')
observer.observe(element)

// Прекратить наблюдение
observer.unobserve(element)
observer.disconnect()
```

## Web Workers

```javascript
// main.js
const worker = new Worker('worker.js')

worker.postMessage({ type: 'calculate', data: [1, 2, 3] })

worker.onmessage = (event) => {
  console.log('Result:', event.data)
}

worker.onerror = (error) => {
  console.error('Worker error:', error)
}

// Завершить worker
worker.terminate()

// worker.js
self.onmessage = (event) => {
  const { type, data } = event.data
  
  if (type === 'calculate') {
    const result = data.reduce((sum, num) => sum + num, 0)
    self.postMessage(result)
  }
}
```

## Clipboard API

```javascript
// Копировать текст
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text)
    console.log('Copied to clipboard')
  } catch (error) {
    console.error('Failed to copy:', error)
  }
}

// Прочитать из буфера
async function readFromClipboard() {
  try {
    const text = await navigator.clipboard.readText()
    return text
  } catch (error) {
    console.error('Failed to read:', error)
  }
}
```

## Вопросы для собеседования

### 1. В чём разница между localStorage и sessionStorage?

**Ответ:**
- **localStorage**: данные сохраняются постоянно
- **sessionStorage**: данные удаляются после закрытия вкладки
- Оба имеют лимит ~5-10MB

### 2. Что такое Intersection Observer?

**Ответ:**
API для отслеживания видимости элементов в viewport. Используется для:
- Ленивой загрузки изображений
- Бесконечного скролла
- Аналитики видимости

### 3. Зачем нужны Web Workers?

**Ответ:**
Для выполнения тяжёлых вычислений в отдельном потоке, не блокируя основной поток и UI.

### 4. Как обработать ошибки в Fetch?

**Ответ:**
```javascript
try {
  const response = await fetch(url)
  if (!response.ok) throw new Error('HTTP error')
  const data = await response.json()
} catch (error) {
  // Обработка ошибки
}
```

### 5. В чём разница между cookies и localStorage?

**Ответ:**
- **Cookies**: отправляются с каждым запросом, ~4KB, есть expiry
- **localStorage**: только на клиенте, ~5-10MB, без expiry
