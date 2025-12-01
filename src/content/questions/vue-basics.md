---
title: "Vue.js: Базовые знания"
description: "Основные концепции и возможности Vue.js для собеседования"
category: "Vue.js"
difficulty: "easy"
tags: ["vue", "basics", "reactivity", "components", "directives"]
order: 16
---

## Что такое Vue.js?

Vue.js — это прогрессивный JavaScript-фреймворк для создания пользовательских интерфейсов. Он разработан для постепенного внедрения: можно использовать только базовые возможности или построить полноценное SPA-приложение.

### Основные особенности

- **Реактивность** — автоматическое обновление DOM при изменении данных
- **Компонентный подход** — переиспользуемые изолированные блоки UI
- **Декларативный рендеринг** — описываем что хотим видеть, а не как это сделать
- **Простота изучения** — понятный синтаксис и хорошая документация

## Создание Vue приложения

### Options API (классический подход)

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Счётчик: {{ count }}</p>
    <button @click="increment">+1</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      title: 'Мой счётчик',
      count: 0
    }
  },
  methods: {
    increment() {
      this.count++
    }
  },
  computed: {
    doubleCount() {
      return this.count * 2
    }
  },
  mounted() {
    console.log('Компонент смонтирован')
  }
}
</script>
```

### Composition API (современный подход)

```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p>Счётчик: {{ count }}</p>
    <button @click="increment">+1</button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const title = ref('Мой счётчик')
const count = ref(0)

const doubleCount = computed(() => count.value * 2)

function increment() {
  count.value++
}

onMounted(() => {
  console.log('Компонент смонтирован')
})
</script>
```

## Реактивность

### ref() — реактивные примитивы

```javascript
import { ref } from 'vue'

// Создание реактивной переменной
const count = ref(0)
const message = ref('Привет')

// Чтение и изменение через .value
console.log(count.value) // 0
count.value++
console.log(count.value) // 1

// В template .value не нужен
// <p>{{ count }}</p>
```

### reactive() — реактивные объекты

```javascript
import { reactive } from 'vue'

// Создание реактивного объекта
const state = reactive({
  count: 0,
  user: {
    name: 'Анна',
    age: 25
  }
})

// Прямое обращение к свойствам
console.log(state.count) // 0
state.count++
state.user.name = 'Борис'
```

### Разница между ref и reactive

```javascript
// ref — для примитивов и объектов
const count = ref(0)
const user = ref({ name: 'Анна' })

count.value++ // Нужен .value
user.value.name = 'Борис' // Нужен .value для объекта

// reactive — только для объектов
const state = reactive({
  count: 0,
  user: { name: 'Анна' }
})

state.count++ // .value не нужен
state.user.name = 'Борис'

// ❌ Нельзя переназначить reactive
state = reactive({ count: 10 }) // Потеряет реактивность!

// ✅ Можно переназначить ref
user.value = { name: 'Новый пользователь' }
```

## Computed свойства

Вычисляемые свойства кэшируются и пересчитываются только при изменении зависимостей.

```vue
<template>
  <div>
    <input v-model="firstName" placeholder="Имя">
    <input v-model="lastName" placeholder="Фамилия">
    <p>Полное имя: {{ fullName }}</p>
    <p>Инициалы: {{ initials }}</p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const firstName = ref('Анна')
const lastName = ref('Иванова')

// Computed только для чтения
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

const initials = computed(() => {
  return `${firstName.value[0]}.${lastName.value[0]}.`
})

// Computed с getter и setter
const fullNameEditable = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(value) {
    const parts = value.split(' ')
    firstName.value = parts[0]
    lastName.value = parts[1]
  }
})
</script>
```

### Computed vs Methods

```vue
<template>
  <div>
    <!-- Computed — кэшируется -->
    <p>{{ expensiveComputed }}</p>
    <p>{{ expensiveComputed }}</p> <!-- Не пересчитывается -->
    
    <!-- Method — вызывается каждый раз -->
    <p>{{ expensiveMethod() }}</p>
    <p>{{ expensiveMethod() }}</p> <!-- Вызывается снова -->
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const items = ref([1, 2, 3, 4, 5])

// Вызывается только при изменении items
const expensiveComputed = computed(() => {
  console.log('Computed вызван')
  return items.value.reduce((sum, n) => sum + n, 0)
})

// Вызывается при каждом рендере
function expensiveMethod() {
  console.log('Method вызван')
  return items.value.reduce((sum, n) => sum + n, 0)
}
</script>
```

## Watchers (наблюдатели)

Следят за изменениями данных и выполняют побочные эффекты.

```vue
<script setup>
import { ref, watch, watchEffect } from 'vue'

const count = ref(0)
const user = ref({ name: 'Анна', age: 25 })

// Простой watch
watch(count, (newValue, oldValue) => {
  console.log(`Счётчик изменился: ${oldValue} → ${newValue}`)
})

// Watch с опциями
watch(
  count,
  (newValue) => {
    console.log('Новое значение:', newValue)
  },
  {
    immediate: true, // Вызвать сразу
    deep: true // Глубокое наблюдение
  }
)

// Watch для объектов (нужен deep)
watch(
  user,
  (newUser) => {
    console.log('Пользователь изменился:', newUser)
  },
  { deep: true }
)

// Watch конкретного свойства
watch(
  () => user.value.name,
  (newName) => {
    console.log('Имя изменилось:', newName)
  }
)

// Watch нескольких источников
watch(
  [count, () => user.value.name],
  ([newCount, newName], [oldCount, oldName]) => {
    console.log('Изменились:', newCount, newName)
  }
)

// watchEffect — автоматически отслеживает зависимости
watchEffect(() => {
  console.log(`Счётчик: ${count.value}, Имя: ${user.value.name}`)
})
</script>
```

## Директивы

### v-bind — привязка атрибутов

```vue
<template>
  <!-- Полная форма -->
  <img v-bind:src="imageSrc" v-bind:alt="imageAlt">
  
  <!-- Сокращённая форма -->
  <img :src="imageSrc" :alt="imageAlt">
  
  <!-- Динамический атрибут -->
  <button :[attributeName]="value">Кнопка</button>
  
  <!-- Привязка класса -->
  <div :class="{ active: isActive, error: hasError }">Текст</div>
  <div :class="[classA, classB]">Текст</div>
  <div :class="[isActive ? 'active' : '', 'base']">Текст</div>
  
  <!-- Привязка стилей -->
  <div :style="{ color: textColor, fontSize: fontSize + 'px' }">Текст</div>
  <div :style="[styleObject1, styleObject2]">Текст</div>
</template>

<script setup>
import { ref } from 'vue'

const imageSrc = ref('/logo.png')
const imageAlt = ref('Логотип')
const attributeName = ref('disabled')
const value = ref(true)

const isActive = ref(true)
const hasError = ref(false)
const classA = ref('class-a')
const classB = ref('class-b')

const textColor = ref('red')
const fontSize = ref(16)
const styleObject1 = ref({ color: 'blue' })
const styleObject2 = ref({ fontSize: '20px' })
</script>
```

### v-on — обработка событий

```vue
<template>
  <!-- Полная форма -->
  <button v-on:click="handleClick">Кликни</button>
  
  <!-- Сокращённая форма -->
  <button @click="handleClick">Кликни</button>
  
  <!-- Инлайн обработчик -->
  <button @click="count++">+1</button>
  
  <!-- С параметрами -->
  <button @click="greet('Анна')">Поздороваться</button>
  
  <!-- Доступ к event -->
  <button @click="handleEvent($event)">Event</button>
  <button @click="(e) => handleEvent(e)">Event</button>
  
  <!-- Модификаторы событий -->
  <form @submit.prevent="onSubmit">
    <button type="submit">Отправить</button>
  </form>
  
  <div @click.self="handleSelf">
    <button @click.stop="handleButton">Кнопка</button>
  </div>
  
  <input @keyup.enter="handleEnter">
  <input @keyup.ctrl.enter="handleCtrlEnter">
  
  <!-- Динамическое событие -->
  <button @[eventName]="handleDynamic">Динамика</button>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(0)
const eventName = ref('click')

function handleClick() {
  console.log('Клик!')
}

function greet(name) {
  console.log(`Привет, ${name}!`)
}

function handleEvent(event) {
  console.log('Event:', event)
}

function onSubmit() {
  console.log('Форма отправлена')
}

function handleSelf() {
  console.log('Клик по самому div')
}

function handleButton() {
  console.log('Клик по кнопке')
}

function handleEnter() {
  console.log('Enter нажат')
}

function handleCtrlEnter() {
  console.log('Ctrl+Enter нажат')
}

function handleDynamic() {
  console.log('Динамическое событие')
}
</script>
```

### v-model — двусторонняя привязка

```vue
<template>
  <!-- Текстовый input -->
  <input v-model="message" placeholder="Введите текст">
  <p>{{ message }}</p>
  
  <!-- Многострочный текст -->
  <textarea v-model="description"></textarea>
  
  <!-- Checkbox -->
  <input type="checkbox" v-model="checked" id="checkbox">
  <label for="checkbox">{{ checked }}</label>
  
  <!-- Множественные checkbox -->
  <input type="checkbox" value="Яблоко" v-model="fruits">
  <input type="checkbox" value="Банан" v-model="fruits">
  <input type="checkbox" value="Апельсин" v-model="fruits">
  <p>Выбрано: {{ fruits }}</p>
  
  <!-- Radio -->
  <input type="radio" value="Да" v-model="answer">
  <input type="radio" value="Нет" v-model="answer">
  <p>Ответ: {{ answer }}</p>
  
  <!-- Select -->
  <select v-model="selected">
    <option disabled value="">Выберите</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  
  <!-- Модификаторы -->
  <input v-model.lazy="lazyMessage"> <!-- Обновление при blur -->
  <input v-model.number="age" type="number"> <!-- Приведение к числу -->
  <input v-model.trim="username"> <!-- Удаление пробелов -->
</template>

<script setup>
import { ref } from 'vue'

const message = ref('')
const description = ref('')
const checked = ref(false)
const fruits = ref([])
const answer = ref('')
const selected = ref('')
const lazyMessage = ref('')
const age = ref(0)
const username = ref('')
</script>
```

### v-if, v-else, v-show

```vue
<template>
  <!-- v-if — условный рендеринг (удаляет из DOM) -->
  <div v-if="type === 'A'">
    Тип A
  </div>
  <div v-else-if="type === 'B'">
    Тип B
  </div>
  <div v-else>
    Другой тип
  </div>
  
  <!-- v-show — переключение видимости (display: none) -->
  <div v-show="isVisible">
    Видимый элемент
  </div>
  
  <!-- template для группировки -->
  <template v-if="showGroup">
    <h1>Заголовок</h1>
    <p>Параграф</p>
  </template>
</template>

<script setup>
import { ref } from 'vue'

const type = ref('A')
const isVisible = ref(true)
const showGroup = ref(true)
</script>
```

**Когда использовать:**
- `v-if` — когда условие редко меняется (высокая стоимость переключения)
- `v-show` — когда условие часто меняется (низкая стоимость переключения)

### v-for — списки

```vue
<template>
  <!-- Массив -->
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
  
  <!-- С индексом -->
  <ul>
    <li v-for="(item, index) in items" :key="item.id">
      {{ index }}: {{ item.name }}
    </li>
  </ul>
  
  <!-- Объект -->
  <ul>
    <li v-for="(value, key) in user" :key="key">
      {{ key }}: {{ value }}
    </li>
  </ul>
  
  <!-- С индексом объекта -->
  <ul>
    <li v-for="(value, key, index) in user" :key="key">
      {{ index }}. {{ key }}: {{ value }}
    </li>
  </ul>
  
  <!-- Диапазон -->
  <span v-for="n in 10" :key="n">{{ n }} </span>
  
  <!-- v-for с v-if (не рекомендуется вместе) -->
  <!-- ❌ Плохо -->
  <li v-for="item in items" v-if="item.isActive" :key="item.id">
    {{ item.name }}
  </li>
  
  <!-- ✅ Хорошо — используйте computed -->
  <li v-for="item in activeItems" :key="item.id">
    {{ item.name }}
  </li>
</template>

<script setup>
import { ref, computed } from 'vue'

const items = ref([
  { id: 1, name: 'Яблоко', isActive: true },
  { id: 2, name: 'Банан', isActive: false },
  { id: 3, name: 'Апельсин', isActive: true }
])

const user = ref({
  name: 'Анна',
  age: 25,
  city: 'Москва'
})

const activeItems = computed(() => {
  return items.value.filter(item => item.isActive)
})
</script>
```

## Lifecycle Hooks (хуки жизненного цикла)

```vue
<script setup>
import { 
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  ref 
} from 'vue'

const count = ref(0)

// До монтирования компонента
onBeforeMount(() => {
  console.log('onBeforeMount: компонент скоро будет смонтирован')
})

// После монтирования (DOM доступен)
onMounted(() => {
  console.log('onMounted: компонент смонтирован')
  // Здесь можно работать с DOM, делать API запросы
})

// До обновления DOM
onBeforeUpdate(() => {
  console.log('onBeforeUpdate: компонент скоро обновится')
})

// После обновления DOM
onUpdated(() => {
  console.log('onUpdated: компонент обновлён')
})

// До размонтирования
onBeforeUnmount(() => {
  console.log('onBeforeUnmount: компонент скоро будет размонтирован')
  // Очистка таймеров, подписок и т.д.
})

// После размонтирования
onUnmounted(() => {
  console.log('onUnmounted: компонент размонтирован')
})
</script>
```

### Порядок вызова хуков

```
1. setup()
2. onBeforeMount()
3. onMounted()
   ↓ (при изменении данных)
4. onBeforeUpdate()
5. onUpdated()
   ↓ (при размонтировании)
6. onBeforeUnmount()
7. onUnmounted()
```

## Компоненты

### Создание и использование компонента

```vue
<!-- Button.vue -->
<template>
  <button :class="variant" @click="handleClick">
    <slot></slot>
  </button>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'

// Props
const props = defineProps({
  variant: {
    type: String,
    default: 'primary',
    validator: (value) => ['primary', 'secondary', 'danger'].includes(value)
  }
})

// Events
const emit = defineEmits(['click'])

function handleClick(event) {
  emit('click', event)
}
</script>

<style scoped>
button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primary {
  background: blue;
  color: white;
}

.secondary {
  background: gray;
  color: white;
}

.danger {
  background: red;
  color: white;
}
</style>
```

```vue
<!-- App.vue -->
<template>
  <div>
    <Button variant="primary" @click="handlePrimaryClick">
      Первичная кнопка
    </Button>
    
    <Button variant="danger" @click="handleDangerClick">
      Опасная кнопка
    </Button>
  </div>
</template>

<script setup>
import Button from './Button.vue'

function handlePrimaryClick() {
  console.log('Primary clicked')
}

function handleDangerClick() {
  console.log('Danger clicked')
}
</script>
```

### Props

```vue
<script setup>
import { defineProps } from 'vue'

// Простое объявление
const props = defineProps(['title', 'count'])

// С типами
const props = defineProps({
  title: String,
  count: Number,
  isActive: Boolean,
  tags: Array,
  user: Object
})

// Подробное объявление
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  count: {
    type: Number,
    default: 0
  },
  tags: {
    type: Array,
    default: () => [] // Функция для объектов/массивов
  },
  status: {
    type: String,
    default: 'pending',
    validator: (value) => {
      return ['pending', 'active', 'completed'].includes(value)
    }
  }
})

// TypeScript
interface Props {
  title: string
  count?: number
  tags?: string[]
}

const props = defineProps<Props>()

// TypeScript с дефолтными значениями
const props = withDefaults(defineProps<Props>(), {
  count: 0,
  tags: () => []
})
</script>
```

### Events (эмиты)

```vue
<template>
  <button @click="increment">+1</button>
</template>

<script setup>
import { defineEmits } from 'vue'

// Простое объявление
const emit = defineEmits(['update', 'delete'])

// С валидацией
const emit = defineEmits({
  update: (value) => {
    if (typeof value === 'number') {
      return true
    }
    console.warn('update event должен передавать число')
    return false
  },
  delete: null // Без валидации
})

function increment() {
  emit('update', 1)
}

// TypeScript
const emit = defineEmits<{
  update: [value: number]
  delete: [id: string]
}>()
</script>
```

### Slots (слоты)

**Что такое слоты?**

Слоты (slots) — это механизм в Vue.js, который позволяет родительскому компоненту передавать контент (HTML, текст, другие компоненты) в дочерний компонент. Это делает компоненты более гибкими и переиспользуемыми.

**Для чего используются слоты:**

1. **Создание гибких компонентов** — один компонент может отображать разный контент
2. **Композиция компонентов** — вложение одних компонентов в другие
3. **Переиспользование разметки** — общая структура с разным содержимым
4. **Избежание дублирования кода** — не нужно создавать множество похожих компонентов

#### Дефолтный слот

Самый простой вид слота — передача контента внутрь компонента:

```vue
<!-- Button.vue -->
<template>
  <button class="btn">
    <slot></slot> <!-- Сюда попадёт контент из родителя -->
  </button>
</template>
```

```vue
<!-- Использование -->
<template>
  <Button>Нажми меня</Button> <!-- Текст попадёт в <slot> -->
  <Button>
    <span>🔥</span> С иконкой
  </Button>
</template>
```

#### Именованные слоты

Когда нужно несколько мест для контента, используются именованные слоты:

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <!-- Слот для заголовка -->
    <header v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </header>
    
    <!-- Дефолтный слот для основного контента -->
    <main class="card-body">
      <slot></slot>
    </main>
    
    <!-- Слот для футера -->
    <footer v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<script setup>
import { useSlots } from 'vue'

const slots = useSlots()
// Проверяем, передан ли слот
console.log('Has header?', !!slots.header)
console.log('Has footer?', !!slots.footer)
</script>

<style scoped>
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
}

.card-header {
  padding: 16px;
  border-bottom: 1px solid #ddd;
  background: #f5f5f5;
}

.card-body {
  padding: 16px;
}

.card-footer {
  padding: 16px;
  border-top: 1px solid #ddd;
  background: #f5f5f5;
}
</style>
```

```vue
<!-- Использование -->
<template>
  <!-- Полная карточка со всеми слотами -->
  <Card>
    <template #header>
      <h1>Заголовок карточки</h1>
    </template>
    
    <p>Основной контент карточки</p>
    <p>Может быть несколько параграфов</p>
    
    <template #footer>
      <button>Сохранить</button>
      <button>Отмена</button>
    </template>
  </Card>
  
  <!-- Карточка только с контентом (без header и footer) -->
  <Card>
    <p>Простая карточка без заголовка и футера</p>
  </Card>
</template>
```

#### Дефолтный контент слота

Можно задать контент по умолчанию, который отобразится, если родитель ничего не передал:

```vue
<!-- Alert.vue -->
<template>
  <div class="alert">
    <slot>
      <!-- Этот текст покажется, если слот пустой -->
      Внимание! Важное сообщение.
    </slot>
  </div>
</template>
```

```vue
<!-- Использование -->
<template>
  <!-- Покажет дефолтный текст -->
  <Alert />
  
  <!-- Покажет переданный текст -->
  <Alert>Ошибка сохранения данных!</Alert>
</template>
```

### Scoped Slots (слоты с данными)

**Что такое Scoped Slots?**

Scoped slots позволяют дочернему компоненту передавать данные обратно в родительский компонент через слот. Это полезно, когда дочерний компонент управляет данными, но родитель решает, как их отображать.

**Для чего используются:**

1. **Кастомизация отображения** — компонент предоставляет данные, родитель решает как их показать
2. **Переиспользуемые списки** — один компонент списка с разными вариантами отображения элементов
3. **Разделение логики и представления** — логика в дочернем, UI в родительском

#### Пример: список с кастомным отображением

```vue
<!-- List.vue -->
<template>
  <ul class="list">
    <li v-for="(item, index) in items" :key="item.id" class="list-item">
      <!-- Передаём данные через слот -->
      <slot :item="item" :index="index"></slot>
    </li>
  </ul>
</template>

<script setup>
import { defineProps } from 'vue'

defineProps({
  items: {
    type: Array,
    required: true
  }
})
</script>
```

```vue
<!-- Использование -->
<template>
  <div>
    <!-- Простое отображение -->
    <List :items="users">
      <template #default="{ item, index }">
        <strong>{{ index + 1 }}.</strong> {{ item.name }}
      </template>
    </List>
    
    <!-- Расширенное отображение -->
    <List :items="users">
      <template #default="{ item }">
        <div class="user-card">
          <img :src="item.avatar" :alt="item.name">
          <div>
            <h3>{{ item.name }}</h3>
            <p>{{ item.email }}</p>
          </div>
        </div>
      </template>
    </List>
    
    <!-- С кнопками действий -->
    <List :items="users">
      <template #default="{ item, index }">
        <span>{{ item.name }}</span>
        <button @click="editUser(item)">Редактировать</button>
        <button @click="deleteUser(index)">Удалить</button>
      </template>
    </List>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import List from './List.vue'

const users = ref([
  { id: 1, name: 'Анна', email: 'anna@mail.ru', avatar: '/anna.jpg' },
  { id: 2, name: 'Борис', email: 'boris@mail.ru', avatar: '/boris.jpg' }
])

function editUser(user) {
  console.log('Редактирование:', user)
}

function deleteUser(index) {
  users.value.splice(index, 1)
}
</script>
```

#### Пример: таблица с сортировкой

```vue
<!-- DataTable.vue -->
<template>
  <table>
    <thead>
      <tr>
        <th v-for="column in columns" :key="column.key">
          {{ column.label }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="row in sortedData" :key="row.id">
        <!-- Передаём строку данных в слот -->
        <slot :row="row"></slot>
      </tr>
    </tbody>
  </table>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  data: Array,
  columns: Array
})

const sortedData = computed(() => {
  // Логика сортировки
  return props.data
})
</script>
```

```vue
<!-- Использование -->
<template>
  <DataTable :data="products" :columns="columns">
    <template #default="{ row }">
      <td>{{ row.name }}</td>
      <td>{{ row.price }} ₽</td>
      <td>
        <span :class="row.inStock ? 'in-stock' : 'out-of-stock'">
          {{ row.inStock ? 'В наличии' : 'Нет в наличии' }}
        </span>
      </td>
    </template>
  </DataTable>
</template>
```

#### Множественные именованные scoped slots

```vue
<!-- ProductCard.vue -->
<template>
  <div class="product-card">
    <div class="product-image">
      <slot name="image" :product="product"></slot>
    </div>
    
    <div class="product-info">
      <slot name="title" :product="product">
        <h3>{{ product.name }}</h3>
      </slot>
      
      <slot name="price" :product="product">
        <p>{{ product.price }} ₽</p>
      </slot>
    </div>
    
    <div class="product-actions">
      <slot name="actions" :product="product" :addToCart="addToCart"></slot>
    </div>
  </div>
</template>

<script setup>
const props = defineProps(['product'])

function addToCart() {
  console.log('Добавлено в корзину:', props.product)
}
</script>
```

```vue
<!-- Использование -->
<template>
  <ProductCard :product="product">
    <template #image="{ product }">
      <img :src="product.image" :alt="product.name">
    </template>
    
    <template #title="{ product }">
      <h2 class="custom-title">🔥 {{ product.name }}</h2>
    </template>
    
    <template #actions="{ product, addToCart }">
      <button @click="addToCart">Купить</button>
      <button @click="addToFavorites(product)">❤️</button>
    </template>
  </ProductCard>
</template>
```

## Provide / Inject

Передача данных через несколько уровней компонентов без props drilling.

```vue
<!-- App.vue (родитель) -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
const user = ref({ name: 'Анна', role: 'admin' })

// Предоставляем данные
provide('theme', theme)
provide('user', user)

// Предоставляем методы
provide('updateTheme', (newTheme) => {
  theme.value = newTheme
})
</script>
```

```vue
<!-- ChildComponent.vue (любой потомок) -->
<template>
  <div :class="theme">
    <p>Пользователь: {{ user.name }}</p>
    <button @click="updateTheme('light')">Светлая тема</button>
  </div>
</template>

<script setup>
import { inject } from 'vue'

// Получаем данные
const theme = inject('theme')
const user = inject('user')
const updateTheme = inject('updateTheme')

// С дефолтным значением
const settings = inject('settings', { lang: 'ru' })
</script>
```

## Template Refs

Доступ к DOM элементам и компонентам.

```vue
<template>
  <input ref="inputRef" type="text">
  <button @click="focusInput">Фокус на input</button>
  
  <ChildComponent ref="childRef" />
  <button @click="callChildMethod">Вызвать метод дочернего</button>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import ChildComponent from './ChildComponent.vue'

const inputRef = ref(null)
const childRef = ref(null)

function focusInput() {
  inputRef.value.focus()
}

function callChildMethod() {
  childRef.value.someMethod()
}

onMounted(() => {
  console.log('Input element:', inputRef.value)
})
</script>
```

### Refs в v-for

```vue
<template>
  <ul>
    <li v-for="item in items" :key="item.id" :ref="setItemRef">
      {{ item.name }}
    </li>
  </ul>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const items = ref([
  { id: 1, name: 'Item 1' },
  { id: 2, name: 'Item 2' }
])

const itemRefs = ref([])

function setItemRef(el) {
  if (el) {
    itemRefs.value.push(el)
  }
}

onMounted(() => {
  console.log('All items:', itemRefs.value)
})
</script>
```

## Composables (переиспользуемая логика)

```javascript
// useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  const doubleCount = computed(() => count.value * 2)
  
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
    doubleCount,
    increment,
    decrement,
    reset
  }
}
```

```vue
<!-- Использование -->
<template>
  <div>
    <p>Счётчик: {{ count }}</p>
    <p>Удвоенный: {{ doubleCount }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
    <button @click="reset">Сброс</button>
  </div>
</template>

<script setup>
import { useCounter } from './useCounter'

const { count, doubleCount, increment, decrement, reset } = useCounter(10)
</script>
```

### Пример: useFetch

```javascript
// useFetch.js
import { ref } from 'vue'

export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const loading = ref(false)
  
  async function fetch() {
    loading.value = true
    error.value = null
    
    try {
      const response = await window.fetch(url)
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      data.value = await response.json()
    } catch (e) {
      error.value = e
    } finally {
      loading.value = false
    }
  }
  
  return { data, error, loading, fetch }
}
```

## Основные вопросы на собеседовании

### 1. В чём разница между Options API и Composition API?

**Options API:**
- Классический подход
- Логика разделена по опциям (data, methods, computed)
- Проще для начинающих
- Хуже переиспользование логики

**Composition API:**
- Современный подход
- Логика группируется по функциональности
- Лучше TypeScript поддержка
- Легче переиспользовать через composables

### 2. Когда использовать ref, а когда reactive?

**ref:**
- Для примитивов (string, number, boolean)
- Когда нужно переназначение
- Для массивов и объектов, которые могут быть заменены

**reactive:**
- Только для объектов
- Когда не нужно переназначение
- Для сложных вложенных структур

### 3. В чём разница между computed и watch?

**computed:**
- Для вычисления значений на основе других данных
- Кэширование результата
- Синхронные операции
- Должен возвращать значение

**watch:**
- Для побочных эффектов (API запросы, логирование)
- Нет кэширования
- Может быть асинхронным
- Не возвращает значение

### 4. Что такое Virtual DOM?

Virtual DOM — это JavaScript-представление реального DOM. Vue создаёт виртуальное дерево, сравнивает его с предыдущей версией (diffing) и обновляет только изменённые части реального DOM (patching).

**Преимущества:**
- Быстрее прямых манипуляций с DOM
- Батчинг обновлений
- Кроссплатформенность

### 5. Что такое реактивность в Vue?

Реактивность — это автоматическое отслеживание зависимостей и обновление UI при изменении данных. Vue использует Proxy (Vue 3) для перехвата операций чтения/записи и отслеживания зависимостей.

## Чек-лист базовых знаний Vue.js

- [ ] Понимание реактивности (ref, reactive)
- [ ] Работа с computed и watch
- [ ] Знание всех основных директив (v-if, v-for, v-model, v-bind, v-on)
- [ ] Lifecycle hooks и их порядок
- [ ] Создание компонентов (props, events, slots)
- [ ] Работа с формами и v-model
- [ ] Provide/Inject для передачи данных
- [ ] Template refs для доступа к DOM
- [ ] Создание composables для переиспользования логики
- [ ] Разница между Options API и Composition API
