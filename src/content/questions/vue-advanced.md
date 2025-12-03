---
title: "Vue.js: Продвинутые концепции"
description: "Продвинутые паттерны и техники работы с Vue.js"
category: "Vue.js"
difficulty: "hard"
tags: ["vue", "advanced", "composables", "provide-inject", "teleport", "suspense"]
order: 18
---

## Composables (Композируемые функции)

Composables — это функции, которые используют Composition API для инкапсуляции и переиспользования логики с состоянием.

### Пример: useCounter

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  const doubled = computed(() => count.value * 2)
  
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  function reset() {
    count.value = initialValue
  }
  
  return {
    count,
    doubled,
    increment,
    decrement,
    reset
  }
}
```

### Использование

```vue
<script setup>
import { useCounter } from './composables/useCounter'

const { count, doubled, increment, decrement, reset } = useCounter(10)
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Doubled: {{ doubled }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
    <button @click="reset">Reset</button>
  </div>
</template>
```

## Provide / Inject

Механизм для передачи данных от родительского компонента к любому потомку без prop drilling.

### Родительский компонент

```vue
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
const updateTheme = (newTheme) => {
  theme.value = newTheme
}

// Предоставляем данные и методы
provide('theme', theme)
provide('updateTheme', updateTheme)
</script>
```

### Дочерний компонент (на любом уровне вложенности)

```vue
<script setup>
import { inject } from 'vue'

const theme = inject('theme')
const updateTheme = inject('updateTheme')
</script>

<template>
  <div :class="theme">
    <button @click="updateTheme('light')">Light</button>
    <button @click="updateTheme('dark')">Dark</button>
  </div>
</template>
```

### Provide с реактивностью

```javascript
// app-level provide
import { createApp, reactive } from 'vue'

const app = createApp({})

const store = reactive({
  user: null,
  isAuthenticated: false
})

app.provide('store', store)
```

## Teleport

Позволяет рендерить содержимое компонента в другом месте DOM-дерева.

### Модальное окно

```vue
<template>
  <button @click="showModal = true">Открыть модал</button>
  
  <Teleport to="body">
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h2>Модальное окно</h2>
        <p>Контент модального окна</p>
        <button @click="showModal = false">Закрыть</button>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
import { ref } from 'vue'

const showModal = ref(false)
</script>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
}
</style>
```

## Suspense (Экспериментальная функция)

Позволяет обрабатывать асинхронные зависимости в дереве компонентов.

### Асинхронный компонент

```vue
<!-- AsyncComponent.vue -->
<script setup>
const data = await fetch('/api/data').then(r => r.json())
</script>

<template>
  <div>{{ data }}</div>
</template>
```

### Использование с Suspense

```vue
<template>
  <Suspense>
    <!-- Основной контент -->
    <template #default>
      <AsyncComponent />
    </template>
    
    <!-- Fallback во время загрузки -->
    <template #fallback>
      <div>Загрузка...</div>
    </template>
  </Suspense>
</template>
```

### Обработка ошибок

```vue
<script setup>
import { onErrorCaptured, ref } from 'vue'

const error = ref(null)

onErrorCaptured((err) => {
  error.value = err
  return false // предотвращает дальнейшее всплытие
})
</script>

<template>
  <div v-if="error">
    Ошибка: {{ error.message }}
  </div>
  <Suspense v-else>
    <AsyncComponent />
    <template #fallback>
      <div>Загрузка...</div>
    </template>
  </Suspense>
</template>
```

## Custom Directives (Пользовательские директивы)

Создание собственных директив для работы с DOM.

### Директива v-focus

```javascript
// directives/focus.js
export const vFocus = {
  mounted(el) {
    el.focus()
  }
}
```

### Директива с аргументами

```javascript
// directives/color.js
export const vColor = {
  mounted(el, binding) {
    el.style.color = binding.value
  },
  updated(el, binding) {
    el.style.color = binding.value
  }
}
```

### Использование

```vue
<script setup>
import { vFocus } from './directives/focus'
import { vColor } from './directives/color'
import { ref } from 'vue'

const color = ref('red')
</script>

<template>
  <input v-focus v-color="color" />
  <button @click="color = 'blue'">Изменить цвет</button>
</template>
```

### Директива с модификаторами

```javascript
export const vScroll = {
  mounted(el, binding) {
    const handler = () => {
      if (binding.modifiers.lazy) {
        // Ленивая обработка
        setTimeout(() => binding.value(), 100)
      } else {
        binding.value()
      }
    }
    
    el.addEventListener('scroll', handler)
    el._scrollHandler = handler
  },
  unmounted(el) {
    el.removeEventListener('scroll', el._scrollHandler)
  }
}
```

## Render Functions & JSX

Программное создание виртуального DOM.

### h() функция

```javascript
import { h, ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    
    return () => h('div', [
      h('h1', 'Counter'),
      h('p', `Count: ${count.value}`),
      h('button', {
        onClick: () => count.value++
      }, '+1')
    ])
  }
}
```

### JSX

```jsx
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    
    return () => (
      <div>
        <h1>Counter</h1>
        <p>Count: {count.value}</p>
        <button onClick={() => count.value++}>+1</button>
      </div>
    )
  }
}
```

## Plugins

Создание плагинов для расширения функциональности Vue.

### Простой плагин

```javascript
// plugins/i18n.js
export default {
  install(app, options) {
    const translations = options.translations || {}
    
    app.config.globalProperties.$t = (key) => {
      return translations[key] || key
    }
    
    app.provide('i18n', {
      t: (key) => translations[key] || key
    })
  }
}
```

### Использование плагина

```javascript
// main.js
import { createApp } from 'vue'
import i18nPlugin from './plugins/i18n'
import App from './App.vue'

const app = createApp(App)

app.use(i18nPlugin, {
  translations: {
    hello: 'Привет',
    goodbye: 'До свидания'
  }
})

app.mount('#app')
```

## Transition Hooks

Программная работа с анимациями.

```vue
<script setup>
import { ref } from 'vue'

const show = ref(true)

function onBeforeEnter(el) {
  el.style.opacity = 0
  el.style.transform = 'translateY(-20px)'
}

function onEnter(el, done) {
  el.offsetHeight // trigger reflow
  el.style.transition = 'all 0.3s'
  el.style.opacity = 1
  el.style.transform = 'translateY(0)'
  
  el.addEventListener('transitionend', done)
}

function onLeave(el, done) {
  el.style.transition = 'all 0.3s'
  el.style.opacity = 0
  el.style.transform = 'translateY(-20px)'
  
  el.addEventListener('transitionend', done)
}
</script>

<template>
  <button @click="show = !show">Toggle</button>
  
  <Transition
    @before-enter="onBeforeEnter"
    @enter="onEnter"
    @leave="onLeave"
    :css="false"
  >
    <div v-if="show">Анимированный контент</div>
  </Transition>
</template>
```

## Вопросы для собеседования

### 1. Что такое composables и чем они отличаются от mixins?

**Ответ:**
Composables — это функции, использующие Composition API для переиспользования логики. Отличия от mixins:

- **Явный источник**: видно откуда приходят свойства
- **Нет конфликтов имён**: можно переименовывать при импорте
- **Лучшая типизация**: TypeScript понимает типы
- **Гибкость**: можно вызывать условно или в цикле

### 2. Когда использовать provide/inject вместо props?

**Ответ:**
- Глубокая вложенность компонентов (избежать prop drilling)
- Плагины и библиотеки (глобальные сервисы)
- Темы и конфигурация приложения
- Не использовать для обычной передачи данных между родителем и ребёнком

### 3. Для чего нужен Teleport?

**Ответ:**
Teleport позволяет рендерить контент компонента в другом месте DOM:
- Модальные окна
- Всплывающие подсказки (tooltips)
- Уведомления (notifications)
- Любой контент, который должен быть вне основного дерева компонентов

### 4. Когда создавать custom directive?

**Ответ:**
Когда нужна прямая работа с DOM:
- Фокус на элементе
- Обработка событий низкого уровня
- Интеграция со сторонними библиотеками
- Анимации и эффекты
- Не использовать для логики, которую можно реализовать через composables

### 5. В чём разница между render functions и templates?

**Ответ:**
**Templates:**
- Декларативный синтаксис
- Проще читать и писать
- Оптимизация компилятором
- Подходит для большинства случаев

**Render functions:**
- Программный подход
- Полная гибкость JavaScript
- Динамическая структура компонента
- Используется для библиотек и сложных случаев
