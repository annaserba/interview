---
title: "Pinia (State Management)"
description: "Управление состоянием в Vue.js с помощью Pinia"
category: "Vue.js"
difficulty: "medium"
tags: ["vue", "pinia", "state-management", "store"]
order: 20
---

## Что такое Pinia?

Pinia — официальная библиотека управления состоянием для Vue.js. Это преемник Vuex с более простым API и лучшей поддержкой TypeScript.

### Основные преимущества

- **Простой API**: меньше boilerplate кода
- **TypeScript**: полная поддержка типизации
- **Devtools**: интеграция с Vue DevTools
- **Модульность**: каждый store независим
- **Composition API**: нативная поддержка

## Установка и настройка

```bash
npm install pinia
```

```javascript
// main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const pinia = createPinia()
const app = createApp(App)

app.use(pinia)
app.mount('#app')
```

## Создание Store

### Options Store (похож на Options API)

```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  // State
  state: () => ({
    count: 0,
    name: 'Counter'
  }),
  
  // Getters (вычисляемые свойства)
  getters: {
    doubleCount: (state) => state.count * 2,
    
    // Геттер с параметром
    countPlusN: (state) => {
      return (n) => state.count + n
    },
    
    // Доступ к другим геттерам
    doubleCountPlusOne() {
      return this.doubleCount + 1
    }
  },
  
  // Actions (методы)
  actions: {
    increment() {
      this.count++
    },
    
    decrement() {
      this.count--
    },
    
    async fetchCount() {
      const response = await fetch('/api/count')
      const data = await response.json()
      this.count = data.count
    },
    
    // Можно вызывать другие actions
    reset() {
      this.count = 0
      this.name = 'Counter'
    }
  }
})
```

### Setup Store (похож на Composition API)

```javascript
// stores/counter.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // State
  const count = ref(0)
  const name = ref('Counter')
  
  // Getters
  const doubleCount = computed(() => count.value * 2)
  const doubleCountPlusOne = computed(() => doubleCount.value + 1)
  
  // Actions
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  async function fetchCount() {
    const response = await fetch('/api/count')
    const data = await response.json()
    count.value = data.count
  }
  
  function reset() {
    count.value = 0
    name.value = 'Counter'
  }
  
  return {
    count,
    name,
    doubleCount,
    doubleCountPlusOne,
    increment,
    decrement,
    fetchCount,
    reset
  }
})
```

## Использование Store в компонентах

### Composition API

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'
import { storeToRefs } from 'pinia'

const store = useCounterStore()

// Деструктуризация с сохранением реактивности
const { count, doubleCount } = storeToRefs(store)

// Actions можно деструктурировать напрямую
const { increment, decrement } = store
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double: {{ doubleCount }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
    <button @click="store.reset()">Reset</button>
  </div>
</template>
```

### Options API

```vue
<script>
import { useCounterStore } from '@/stores/counter'
import { mapState, mapActions } from 'pinia'

export default {
  computed: {
    ...mapState(useCounterStore, ['count', 'doubleCount']),
    // Или с переименованием
    ...mapState(useCounterStore, {
      myCount: 'count',
      myDouble: 'doubleCount'
    })
  },
  
  methods: {
    ...mapActions(useCounterStore, ['increment', 'decrement', 'reset'])
  }
}
</script>
```

## Работа с несколькими Store

```javascript
// stores/user.js
export const useUserStore = defineStore('user', {
  state: () => ({
    user: null,
    isAuthenticated: false
  }),
  
  actions: {
    async login(credentials) {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify(credentials)
      })
      this.user = await response.json()
      this.isAuthenticated = true
    },
    
    logout() {
      this.user = null
      this.isAuthenticated = false
    }
  }
})

// stores/cart.js
import { useUserStore } from './user'

export const useCartStore = defineStore('cart', {
  state: () => ({
    items: []
  }),
  
  actions: {
    async checkout() {
      const userStore = useUserStore()
      
      if (!userStore.isAuthenticated) {
        throw new Error('User not authenticated')
      }
      
      // Оформление заказа
      await fetch('/api/checkout', {
        method: 'POST',
        body: JSON.stringify({
          userId: userStore.user.id,
          items: this.items
        })
      })
      
      this.items = []
    }
  }
})
```

## Плагины

### Персистентность (сохранение в localStorage)

```javascript
// plugins/persistence.js
export function persistencePlugin({ store }) {
  // Загрузка из localStorage при инициализации
  const saved = localStorage.getItem(store.$id)
  if (saved) {
    store.$patch(JSON.parse(saved))
  }
  
  // Сохранение при изменении
  store.$subscribe((mutation, state) => {
    localStorage.setItem(store.$id, JSON.stringify(state))
  })
}

// main.js
import { createPinia } from 'pinia'
import { persistencePlugin } from './plugins/persistence'

const pinia = createPinia()
pinia.use(persistencePlugin)
```

### Логирование

```javascript
// plugins/logger.js
export function loggerPlugin({ store }) {
  store.$onAction(({ name, args, after, onError }) => {
    console.log(`Action "${name}" called with args:`, args)
    
    after((result) => {
      console.log(`Action "${name}" returned:`, result)
    })
    
    onError((error) => {
      console.error(`Action "${name}" failed:`, error)
    })
  })
}
```

## Расширенные возможности

### $patch для множественных изменений

```javascript
const store = useCounterStore()

// Объект
store.$patch({
  count: store.count + 1,
  name: 'New Counter'
})

// Функция (лучше для сложной логики)
store.$patch((state) => {
  state.count++
  state.items.push({ id: 1, name: 'Item' })
})
```

### $reset для сброса состояния

```javascript
const store = useCounterStore()
store.$reset() // Возвращает к начальному state
```

### $subscribe для отслеживания изменений

```javascript
const store = useCounterStore()

store.$subscribe((mutation, state) => {
  console.log('State changed:', state)
  
  // Сохранить в localStorage
  localStorage.setItem('counter', JSON.stringify(state))
})

// С detached: true подписка переживёт unmount компонента
store.$subscribe(callback, { detached: true })
```

### $onAction для отслеживания actions

```javascript
const unsubscribe = store.$onAction(({
  name,      // имя action
  args,      // аргументы
  after,     // хук после выполнения
  onError    // хук при ошибке
}) => {
  console.log(`Action ${name} started`)
  
  after((result) => {
    console.log(`Action ${name} finished with result:`, result)
  })
  
  onError((error) => {
    console.error(`Action ${name} failed:`, error)
  })
})

// Отписаться
unsubscribe()
```

## TypeScript

```typescript
// stores/user.ts
import { defineStore } from 'pinia'

interface User {
  id: number
  name: string
  email: string
}

interface UserState {
  user: User | null
  isAuthenticated: boolean
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    user: null,
    isAuthenticated: false
  }),
  
  getters: {
    userName: (state): string => {
      return state.user?.name ?? 'Guest'
    }
  },
  
  actions: {
    async login(email: string, password: string): Promise<void> {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      })
      this.user = await response.json()
      this.isAuthenticated = true
    }
  }
})
```

### Setup Store с TypeScript

```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
}

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => user.value !== null)
  
  async function login(email: string, password: string): Promise<void> {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    })
    user.value = await response.json()
  }
  
  return {
    user,
    isAuthenticated,
    login
  }
})
```

## Тестирование

```javascript
import { setActivePinia, createPinia } from 'pinia'
import { useCounterStore } from '@/stores/counter'

describe('Counter Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('increments counter', () => {
    const store = useCounterStore()
    expect(store.count).toBe(0)
    
    store.increment()
    expect(store.count).toBe(1)
  })
  
  it('computes double count', () => {
    const store = useCounterStore()
    store.count = 5
    expect(store.doubleCount).toBe(10)
  })
  
  it('resets state', () => {
    const store = useCounterStore()
    store.count = 10
    store.$reset()
    expect(store.count).toBe(0)
  })
})
```

## Вопросы для собеседования

### 1. В чём разница между Pinia и Vuex?

**Ответ:**
- **Проще API**: нет mutations, только actions
- **TypeScript**: лучшая поддержка типизации
- **Модульность**: не нужны модули, каждый store независим
- **Размер**: меньше размер бандла
- **DevTools**: лучшая интеграция

### 2. Когда использовать Options Store vs Setup Store?

**Ответ:**
- **Options Store**: проще для начинающих, похож на Options API
- **Setup Store**: больше гибкости, лучше для сложной логики, можно использовать composables

### 3. Зачем нужен storeToRefs?

**Ответ:**
Для деструктуризации state и getters с сохранением реактивности:
```javascript
const { count } = storeToRefs(store) // реактивно
const { count } = store // НЕ реактивно
```

### 4. Как организовать большое приложение с Pinia?

**Ответ:**
- Создавать отдельный store для каждой функциональной области
- Использовать композицию stores (один store использует другой)
- Выносить общую логику в composables
- Использовать плагины для кросс-cutting concerns

### 5. Как тестировать Pinia stores?

**Ответ:**
- Создавать новый Pinia instance для каждого теста
- Использовать `setActivePinia(createPinia())`
- Тестировать actions, getters и state изолированно
- Мокать API запросы
