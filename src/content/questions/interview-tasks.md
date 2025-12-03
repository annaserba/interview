---
title: "Практические задачи с собеседований"
description: "Реальные coding задачи с разбором решений"
category: "Лайв-кодинг"
difficulty: "hard"
tags: ["coding", "interview", "algorithms", "practice"]
order: 27
---

## Задачи на массивы и объекты

### 1. Flatten вложенного массива

```javascript
// Задача: [1, [2, [3, 4], 5], 6] → [1, 2, 3, 4, 5, 6]

// Решение 1: Рекурсия
function flatten(arr) {
  const result = []
  
  for (const item of arr) {
    if (Array.isArray(item)) {
      result.push(...flatten(item))
    } else {
      result.push(item)
    }
  }
  
  return result
}

// Решение 2: Reduce
function flatten(arr) {
  return arr.reduce((acc, item) => {
    return acc.concat(Array.isArray(item) ? flatten(item) : item)
  }, [])
}

// Решение 3: flat() (встроенный метод)
const flattened = arr.flat(Infinity)

console.log(flatten([1, [2, [3, 4], 5], 6])) // [1, 2, 3, 4, 5, 6]
```

### 2. Deep clone объекта

```javascript
// Задача: Создать глубокую копию объекта с вложенными объектами и массивами

function deepClone(obj, hash = new WeakMap()) {
  // Примитивы и null
  if (obj === null || typeof obj !== 'object') {
    return obj
  }
  
  // Циклические ссылки
  if (hash.has(obj)) {
    return hash.get(obj)
  }
  
  // Date
  if (obj instanceof Date) {
    return new Date(obj)
  }
  
  // RegExp
  if (obj instanceof RegExp) {
    return new RegExp(obj)
  }
  
  // Массив или объект
  const clone = Array.isArray(obj) ? [] : {}
  hash.set(obj, clone)
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key], hash)
    }
  }
  
  return clone
}

// Тест
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'Moscow',
    coords: { lat: 55, lng: 37 }
  },
  hobbies: ['reading', 'coding']
}

const copy = deepClone(original)
copy.address.city = 'SPB'
console.log(original.address.city) // Moscow (не изменился)
```

### 3. Group by ключу

```javascript
// Задача: Сгруппировать массив объектов по ключу

function groupBy(arr, key) {
  return arr.reduce((acc, item) => {
    const group = item[key]
    if (!acc[group]) {
      acc[group] = []
    }
    acc[group].push(item)
    return acc
  }, {})
}

// Или с функцией
function groupBy(arr, fn) {
  return arr.reduce((acc, item) => {
    const group = typeof fn === 'function' ? fn(item) : item[fn]
    if (!acc[group]) {
      acc[group] = []
    }
    acc[group].push(item)
    return acc
  }, {})
}

const users = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 },
  { name: 'Bob', age: 30 }
]

console.log(groupBy(users, 'age'))
// { 25: [{...}], 30: [{...}, {...}] }

console.log(groupBy(users, user => user.age >= 30 ? 'senior' : 'junior'))
// { senior: [{...}, {...}], junior: [{...}] }
```

### 4. Найти пересечение массивов

```javascript
// Задача: Найти общие элементы в нескольких массивах

function intersection(...arrays) {
  if (arrays.length === 0) return []
  if (arrays.length === 1) return arrays[0]
  
  const [first, ...rest] = arrays
  const set = new Set(first)
  
  return [...set].filter(item => 
    rest.every(arr => arr.includes(item))
  )
}

console.log(intersection([1, 2, 3], [2, 3, 4], [2, 3, 5]))
// [2, 3]
```

## Задачи на строки

### 5. Проверка на палиндром

```javascript
// Задача: Проверить, является ли строка палиндромом (игнорируя пробелы и регистр)

function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '')
  return cleaned === cleaned.split('').reverse().join('')
}

// Оптимизированное решение (без создания новой строки)
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '')
  let left = 0
  let right = cleaned.length - 1
  
  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false
    }
    left++
    right--
  }
  
  return true
}

console.log(isPalindrome('A man, a plan, a canal: Panama')) // true
console.log(isPalindrome('race a car')) // false
```

### 6. Анаграммы

```javascript
// Задача: Проверить, являются ли две строки анаграммами

function areAnagrams(str1, str2) {
  const normalize = str => 
    str.toLowerCase().replace(/[^a-z]/g, '').split('').sort().join('')
  
  return normalize(str1) === normalize(str2)
}

// Оптимизированное решение (O(n) вместо O(n log n))
function areAnagrams(str1, str2) {
  const clean1 = str1.toLowerCase().replace(/[^a-z]/g, '')
  const clean2 = str2.toLowerCase().replace(/[^a-z]/g, '')
  
  if (clean1.length !== clean2.length) return false
  
  const charCount = {}
  
  for (const char of clean1) {
    charCount[char] = (charCount[char] || 0) + 1
  }
  
  for (const char of clean2) {
    if (!charCount[char]) return false
    charCount[char]--
  }
  
  return true
}

console.log(areAnagrams('listen', 'silent')) // true
console.log(areAnagrams('hello', 'world')) // false
```

### 7. Самая длинная подстрока без повторов

```javascript
// Задача: Найти длину самой длинной подстроки без повторяющихся символов

function lengthOfLongestSubstring(s) {
  const seen = new Map()
  let maxLength = 0
  let start = 0
  
  for (let end = 0; end < s.length; end++) {
    const char = s[end]
    
    if (seen.has(char) && seen.get(char) >= start) {
      start = seen.get(char) + 1
    }
    
    seen.set(char, end)
    maxLength = Math.max(maxLength, end - start + 1)
  }
  
  return maxLength
}

console.log(lengthOfLongestSubstring('abcabcbb')) // 3 (abc)
console.log(lengthOfLongestSubstring('bbbbb'))    // 1 (b)
console.log(lengthOfLongestSubstring('pwwkew'))   // 3 (wke)
```

## Задачи на асинхронность

### 8. Promise.all с лимитом конкурентности

```javascript
// Задача: Выполнить промисы с ограничением количества одновременных запросов

async function promiseAllWithLimit(promises, limit) {
  const results = []
  const executing = []
  
  for (const [index, promise] of promises.entries()) {
    const p = Promise.resolve(promise).then(result => {
      results[index] = result
    })
    
    executing.push(p)
    
    if (executing.length >= limit) {
      await Promise.race(executing)
      executing.splice(executing.findIndex(p => p === await p), 1)
    }
  }
  
  await Promise.all(executing)
  return results
}

// Использование
const urls = Array.from({ length: 10 }, (_, i) => 
  `https://api.example.com/item/${i}`
)

const promises = urls.map(url => fetch(url).then(r => r.json()))
const results = await promiseAllWithLimit(promises, 3) // Максимум 3 одновременно
```

### 9. Retry с экспоненциальной задержкой

```javascript
// Задача: Повторять запрос при ошибке с увеличивающейся задержкой

async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn()
    } catch (error) {
      if (i === maxRetries - 1) throw error
      
      const delay = baseDelay * Math.pow(2, i)
      console.log(`Retry ${i + 1} after ${delay}ms`)
      await new Promise(resolve => setTimeout(resolve, delay))
    }
  }
}

// Использование
const fetchData = () => fetch('https://api.example.com/data')
  .then(r => r.json())

try {
  const data = await retryWithBackoff(fetchData, 3, 1000)
  console.log(data)
} catch (error) {
  console.error('Failed after retries:', error)
}
```

### 10. Debounce Promise

```javascript
// Задача: Debounce для асинхронных функций

function debouncePromise(fn, delay) {
  let timeoutId
  let pendingPromise = null
  
  return function(...args) {
    clearTimeout(timeoutId)
    
    if (!pendingPromise) {
      pendingPromise = new Promise((resolve, reject) => {
        timeoutId = setTimeout(async () => {
          try {
            const result = await fn.apply(this, args)
            resolve(result)
          } catch (error) {
            reject(error)
          } finally {
            pendingPromise = null
          }
        }, delay)
      })
    }
    
    return pendingPromise
  }
}

// Использование
const searchAPI = debouncePromise(async (query) => {
  const response = await fetch(`/api/search?q=${query}`)
  return response.json()
}, 300)

// Вызовы в течение 300ms будут игнорироваться
searchAPI('test1')
searchAPI('test2')
const results = await searchAPI('test3') // Выполнится только этот
```

## Задачи на DOM и события

### 11. Event Delegation

```javascript
// Задача: Реализовать делегирование событий для динамического списка

class TodoList {
  constructor(containerSelector) {
    this.container = document.querySelector(containerSelector)
    this.todos = []
    this.setupEventListeners()
  }
  
  setupEventListeners() {
    this.container.addEventListener('click', (e) => {
      const target = e.target
      
      // Удаление
      if (target.classList.contains('delete-btn')) {
        const id = target.closest('.todo-item').dataset.id
        this.deleteTodo(id)
      }
      
      // Переключение completed
      if (target.classList.contains('checkbox')) {
        const id = target.closest('.todo-item').dataset.id
        this.toggleTodo(id)
      }
    })
  }
  
  addTodo(text) {
    const todo = {
      id: Date.now().toString(),
      text,
      completed: false
    }
    this.todos.push(todo)
    this.render()
  }
  
  deleteTodo(id) {
    this.todos = this.todos.filter(todo => todo.id !== id)
    this.render()
  }
  
  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id)
    if (todo) {
      todo.completed = !todo.completed
      this.render()
    }
  }
  
  render() {
    this.container.innerHTML = this.todos.map(todo => `
      <div class="todo-item" data-id="${todo.id}">
        <input type="checkbox" class="checkbox" ${todo.completed ? 'checked' : ''}>
        <span class="${todo.completed ? 'completed' : ''}">${todo.text}</span>
        <button class="delete-btn">Delete</button>
      </div>
    `).join('')
  }
}
```

### 12. Infinite Scroll

```javascript
// Задача: Реализовать бесконечную прокрутку

class InfiniteScroll {
  constructor(containerSelector, fetchFn) {
    this.container = document.querySelector(containerSelector)
    this.fetchFn = fetchFn
    this.page = 1
    this.loading = false
    this.hasMore = true
    
    this.setupObserver()
  }
  
  setupObserver() {
    const sentinel = document.createElement('div')
    sentinel.className = 'sentinel'
    this.container.appendChild(sentinel)
    
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && !this.loading && this.hasMore) {
          this.loadMore()
        }
      },
      { threshold: 0.1 }
    )
    
    observer.observe(sentinel)
  }
  
  async loadMore() {
    this.loading = true
    this.showLoader()
    
    try {
      const items = await this.fetchFn(this.page)
      
      if (items.length === 0) {
        this.hasMore = false
      } else {
        this.renderItems(items)
        this.page++
      }
    } catch (error) {
      console.error('Failed to load items:', error)
    } finally {
      this.loading = false
      this.hideLoader()
    }
  }
  
  renderItems(items) {
    const html = items.map(item => `
      <div class="item">${item.title}</div>
    `).join('')
    
    const sentinel = this.container.querySelector('.sentinel')
    sentinel.insertAdjacentHTML('beforebegin', html)
  }
  
  showLoader() {
    const loader = document.createElement('div')
    loader.className = 'loader'
    loader.textContent = 'Loading...'
    this.container.appendChild(loader)
  }
  
  hideLoader() {
    const loader = this.container.querySelector('.loader')
    if (loader) loader.remove()
  }
}

// Использование
const scroll = new InfiniteScroll('#content', async (page) => {
  const response = await fetch(`/api/items?page=${page}`)
  return response.json()
})
```

## Задачи на алгоритмы

### 13. LRU Cache

```javascript
// Задача: Реализовать LRU (Least Recently Used) кэш

class LRUCache {
  constructor(capacity) {
    this.capacity = capacity
    this.cache = new Map()
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1
    
    // Переместить в конец (самый свежий)
    const value = this.cache.get(key)
    this.cache.delete(key)
    this.cache.set(key, value)
    
    return value
  }
  
  put(key, value) {
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
}

// Тест
const cache = new LRUCache(2)
cache.put(1, 1)
cache.put(2, 2)
console.log(cache.get(1))    // 1
cache.put(3, 3)              // Удалит ключ 2
console.log(cache.get(2))    // -1 (не найден)
```

### 14. Throttle с trailing call

```javascript
// Задача: Реализовать throttle с вызовом в конце

function throttle(fn, delay) {
  let lastCall = 0
  let timeoutId = null
  
  return function(...args) {
    const now = Date.now()
    const timeSinceLastCall = now - lastCall
    
    // Очистить предыдущий trailing call
    if (timeoutId) {
      clearTimeout(timeoutId)
      timeoutId = null
    }
    
    if (timeSinceLastCall >= delay) {
      // Вызвать сразу
      lastCall = now
      fn.apply(this, args)
    } else {
      // Запланировать trailing call
      timeoutId = setTimeout(() => {
        lastCall = Date.now()
        fn.apply(this, args)
        timeoutId = null
      }, delay - timeSinceLastCall)
    }
  }
}
```

## Вопросы для обсуждения

### 1. Как бы вы оптимизировали рендеринг списка из 10000 элементов?

**Ответ:**
- Виртуализация (рендерить только видимые элементы)
- Пагинация или infinite scroll
- Debounce для фильтрации/поиска
- Мемоизация компонентов
- Web Workers для тяжёлых вычислений

### 2. Объясните разницу между debounce и throttle

**Ответ:**
- **Debounce**: откладывает выполнение до прекращения вызовов
- **Throttle**: гарантирует выполнение не чаще чем раз в N мс

Debounce для поиска, throttle для scroll/resize.

### 3. Как реализовать глубокое сравнение объектов?

**Ответ:**
```javascript
function deepEqual(obj1, obj2) {
  if (obj1 === obj2) return true
  
  if (typeof obj1 !== 'object' || typeof obj2 !== 'object' ||
      obj1 === null || obj2 === null) {
    return false
  }
  
  const keys1 = Object.keys(obj1)
  const keys2 = Object.keys(obj2)
  
  if (keys1.length !== keys2.length) return false
  
  return keys1.every(key => deepEqual(obj1[key], obj2[key]))
}
```
