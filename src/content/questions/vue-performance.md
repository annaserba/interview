---
title: "Vue.js: Оптимизация производительности"
description: "Продвинутые техники оптимизации Vue приложений"
category: "Vue.js"
difficulty: "hard"
tags: ["vue", "performance", "optimization", "best-practices"]
order: 28
---

## Оптимизация рендеринга

### v-once для статического контента

```vue
<template>
  <!-- Рендерится только один раз -->
  <div v-once>
    <h1>{{ title }}</h1>
    <p>{{ description }}</p>
  </div>
</template>
```

### v-memo для условного обновления

```vue
<template>
  <!-- Обновляется только если изменились зависимости -->
  <div v-memo="[user.id, user.name]">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <p>{{ user.phone }}</p>
  </div>
  
  <!-- Для списков -->
  <div 
    v-for="item in list" 
    :key="item.id"
    v-memo="[item.id, item.selected]"
  >
    {{ item.name }}
  </div>
</template>
```

### Computed vs Methods

```vue
<script setup>
import { ref, computed } from 'vue'

const count = ref(0)

// ✅ Computed - кэшируется
const doubled = computed(() => {
  console.log('Computing doubled')
  return count.value * 2
})

// ❌ Method - вызывается при каждом рендере
function getDoubled() {
  console.log('Computing doubled')
  return count.value * 2
}
</script>

<template>
  <div>
    <!-- Вызовется один раз при изменении count -->
    <p>{{ doubled }}</p>
    
    <!-- Вызовется при каждом рендере компонента -->
    <p>{{ getDoubled() }}</p>
  </div>
</template>
```

### Ленивая загрузка компонентов

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

// Ленивая загрузка
const HeavyComponent = defineAsyncComponent(() =>
  import('./HeavyComponent.vue')
)

// С опциями
const HeavyComponent = defineAsyncComponent({
  loader: () => import('./HeavyComponent.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})
</script>

<template>
  <Suspense>
    <HeavyComponent />
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>
```

## Виртуализация списков

### Виртуальный скролл

```vue
<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  items: Array,
  itemHeight: { type: Number, default: 50 },
  visibleCount: { type: Number, default: 20 }
})

const scrollTop = ref(0)

const startIndex = computed(() => 
  Math.floor(scrollTop.value / props.itemHeight)
)

const endIndex = computed(() => 
  Math.min(
    startIndex.value + props.visibleCount,
    props.items.length
  )
)

const visibleItems = computed(() => 
  props.items.slice(startIndex.value, endIndex.value)
)

const totalHeight = computed(() => 
  props.items.length * props.itemHeight
)

const offsetY = computed(() => 
  startIndex.value * props.itemHeight
)

function handleScroll(e) {
  scrollTop.value = e.target.scrollTop
}
</script>

<template>
  <div 
    class="virtual-list"
    @scroll="handleScroll"
    :style="{ height: `${visibleCount * itemHeight}px`, overflow: 'auto' }"
  >
    <div :style="{ height: `${totalHeight}px`, position: 'relative' }">
      <div :style="{ transform: `translateY(${offsetY}px)` }">
        <div
          v-for="item in visibleItems"
          :key="item.id"
          :style="{ height: `${itemHeight}px` }"
        >
          <slot :item="item" />
        </div>
      </div>
    </div>
  </div>
</template>
```

### Использование vue-virtual-scroller

```bash
npm install vue-virtual-scroller
```

```vue
<script setup>
import { RecycleScroller } from 'vue-virtual-scroller'
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

const items = ref(Array.from({ length: 10000 }, (_, i) => ({
  id: i,
  title: `Item ${i}`
})))
</script>

<template>
  <RecycleScroller
    :items="items"
    :item-size="50"
    key-field="id"
    v-slot="{ item }"
  >
    <div class="item">{{ item.title }}</div>
  </RecycleScroller>
</template>
```

## Оптимизация реактивности

### shallowRef и shallowReactive

```vue
<script setup>
import { ref, shallowRef, shallowReactive } from 'vue'

// Глубокая реактивность (медленно для больших объектов)
const deepState = ref({
  user: {
    profile: {
      name: 'John',
      settings: { theme: 'dark' }
    }
  }
})

// Поверхностная реактивность (быстро)
const shallowState = shallowRef({
  user: {
    profile: {
      name: 'John',
      settings: { theme: 'dark' }
    }
  }
})

// Изменение вложенного свойства не вызовет обновление
shallowState.value.user.profile.name = 'Jane' // Не сработает

// Нужно заменить весь объект
shallowState.value = {
  ...shallowState.value,
  user: {
    ...shallowState.value.user,
    profile: {
      ...shallowState.value.user.profile,
      name: 'Jane'
    }
  }
}

// Или использовать triggerRef
import { triggerRef } from 'vue'
shallowState.value.user.profile.name = 'Jane'
triggerRef(shallowState)
</script>
```

### markRaw для неотслеживаемых данных

```vue
<script setup>
import { ref, markRaw } from 'vue'

// Большой объект, который не нужно отслеживать
const heavyData = markRaw({
  // Много данных
  items: new Array(10000).fill(null).map((_, i) => ({ id: i }))
})

// Библиотеки третьих сторон
import Chart from 'chart.js'
const chartInstance = markRaw(new Chart(ctx, config))
</script>
```

### readonly для защиты от изменений

```vue
<script setup>
import { ref, readonly } from 'vue'

const state = ref({ count: 0 })
const readonlyState = readonly(state)

// Можно читать
console.log(readonlyState.value.count)

// Нельзя изменять (warning в dev mode)
readonlyState.value.count++ // Warning
</script>
```

## Оптимизация компонентов

### KeepAlive для кэширования

```vue
<template>
  <KeepAlive :max="10">
    <component :is="currentComponent" />
  </KeepAlive>
  
  <!-- С include/exclude -->
  <KeepAlive :include="['ComponentA', 'ComponentB']">
    <component :is="currentComponent" />
  </KeepAlive>
  
  <!-- С роутером -->
  <router-view v-slot="{ Component }">
    <KeepAlive>
      <component :is="Component" />
    </KeepAlive>
  </router-view>
</template>

<script setup>
// В компоненте можно использовать хуки
import { onActivated, onDeactivated } from 'vue'

onActivated(() => {
  console.log('Component activated')
  // Обновить данные
})

onDeactivated(() => {
  console.log('Component deactivated')
  // Очистить таймеры
})
</script>
```

### Функциональные компоненты

```vue
<!-- FunctionalComponent.vue -->
<script setup>
// Нет реактивности, нет lifecycle hooks
// Просто функция рендеринга
defineProps(['title', 'items'])
</script>

<template>
  <div>
    <h2>{{ title }}</h2>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>
```

## Оптимизация Pinia Store

### Нормализация данных

```typescript
// ❌ Плохо - дублирование данных
interface State {
  posts: Array<{
    id: number
    title: string
    author: {
      id: number
      name: string
    }
  }>
}

// ✅ Хорошо - нормализованные данные
interface State {
  posts: Record<number, Post>
  users: Record<number, User>
  postIds: number[]
}

export const usePostStore = defineStore('posts', {
  state: (): State => ({
    posts: {},
    users: {},
    postIds: []
  }),
  
  getters: {
    getPostById: (state) => (id: number) => {
      const post = state.posts[id]
      if (!post) return null
      
      return {
        ...post,
        author: state.users[post.authorId]
      }
    },
    
    allPosts: (state) => {
      return state.postIds.map(id => state.posts[id])
    }
  },
  
  actions: {
    setPosts(posts: Post[]) {
      posts.forEach(post => {
        this.posts[post.id] = post
        this.users[post.authorId] = post.author
        if (!this.postIds.includes(post.id)) {
          this.postIds.push(post.id)
        }
      })
    }
  }
})
```

### Селективная подписка

```vue
<script setup>
import { storeToRefs } from 'pinia'
import { useUserStore } from '@/stores/user'

const userStore = useUserStore()

// ❌ Плохо - подписка на весь store
const { user, posts, comments, settings } = storeToRefs(userStore)

// ✅ Хорошо - только нужные данные
const { user } = storeToRefs(userStore)
</script>
```

## Оптимизация сборки

### Vite конфигурация

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  
  build: {
    // Разделение на chunks
    rollupOptions: {
      output: {
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          'ui-vendor': ['element-plus'],
          'utils': ['lodash-es', 'dayjs']
        }
      }
    },
    
    // Минимальный размер для chunk
    chunkSizeWarningLimit: 1000,
    
    // Source maps только для production
    sourcemap: false
  },
  
  // Оптимизация зависимостей
  optimizeDeps: {
    include: ['vue', 'vue-router', 'pinia']
  }
})
```

### Tree shaking

```javascript
// ❌ Плохо - импорт всей библиотеки
import _ from 'lodash'
_.debounce(fn, 300)

// ✅ Хорошо - импорт только нужной функции
import debounce from 'lodash-es/debounce'
debounce(fn, 300)

// Или используйте lodash-es
import { debounce } from 'lodash-es'
```

## Мониторинг производительности

### Performance API

```vue
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  // Измерить время рендеринга
  performance.mark('component-mounted')
  
  const measure = performance.measure(
    'component-render',
    'component-created',
    'component-mounted'
  )
  
  console.log(`Render time: ${measure.duration}ms`)
})

// Отслеживание долгих задач
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 50) {
      console.warn('Long task detected:', entry)
    }
  }
})

observer.observe({ entryTypes: ['longtask'] })
</script>
```

### Vue DevTools Performance

```vue
<script setup>
// Использовать Vue DevTools для:
// - Timeline записи
// - Component render time
// - Memory usage
// - Event tracking

// Добавить метки для профилирования
import { getCurrentInstance } from 'vue'

const instance = getCurrentInstance()
instance?.appContext.config.performance = true
</script>
```

## Лучшие практики

### 1. Избегать глубокой вложенности

```vue
<!-- ❌ Плохо -->
<template>
  <div>
    <div>
      <div>
        <div>
          <ComponentA>
            <ComponentB>
              <ComponentC />
            </ComponentB>
          </ComponentA>
        </div>
      </div>
    </div>
  </div>
</template>

<!-- ✅ Хорошо -->
<template>
  <ComponentA>
    <ComponentB>
      <ComponentC />
    </ComponentB>
  </ComponentA>
</template>
```

### 2. Использовать key правильно

```vue
<!-- ❌ Плохо - index как key -->
<div v-for="(item, index) in items" :key="index">
  {{ item.name }}
</div>

<!-- ✅ Хорошо - уникальный id -->
<div v-for="item in items" :key="item.id">
  {{ item.name }}
</div>
```

### 3. Debounce для дорогих операций

```vue
<script setup>
import { ref, watch } from 'vue'
import { useDebounceFn } from '@vueuse/core'

const searchQuery = ref('')
const results = ref([])

const debouncedSearch = useDebounceFn(async (query) => {
  const response = await fetch(`/api/search?q=${query}`)
  results.value = await response.json()
}, 300)

watch(searchQuery, (newQuery) => {
  debouncedSearch(newQuery)
})
</script>
```

## Вопросы для собеседования

### 1. Когда использовать computed vs watch?

**Ответ:**
- **Computed**: для производных данных, синхронных вычислений
- **Watch**: для побочных эффектов, асинхронных операций

### 2. Что такое v-memo и когда его использовать?

**Ответ:**
`v-memo` кэширует результат рендеринга и обновляет только при изменении зависимостей. Используется для:
- Больших списков
- Тяжёлых компонентов
- Оптимизации v-for

### 3. Как оптимизировать большой список?

**Ответ:**
- Виртуализация (vue-virtual-scroller)
- Пагинация
- Infinite scroll
- v-memo для элементов
- shallowRef для данных

### 4. В чём разница между shallowRef и ref?

**Ответ:**
- **ref**: глубокая реактивность (отслеживает все вложенные свойства)
- **shallowRef**: поверхностная реактивность (только .value)

shallowRef быстрее для больших объектов.

### 5. Как измерить производительность Vue приложения?

**Ответ:**
- Vue DevTools Performance tab
- Chrome DevTools Performance
- Lighthouse
- Performance API
- Web Vitals (LCP, FID, CLS)
