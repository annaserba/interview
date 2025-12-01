---
title: "Vue.js: Исправление кода"
description: "Практические задачи по отладке и исправлению кода на Vue.js"
category: "Vue.js"
difficulty: "medium"
tags: ["vue", "debugging", "reactivity", "composition-api"]
order: 17
---

## Задача 1: Исправить реактивность

### ❌ Проблемный код

```vue
<template>
  <div>
    <h2>Пользователи</h2>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.name }} - {{ user.age }} лет
        <button @click="updateAge(user)">+1 год</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const users = ref([
  { id: 1, name: 'Анна', age: 25 },
  { id: 2, name: 'Борис', age: 30 }
]);

// Проблема: возраст не обновляется в UI
function updateAge(user) {
  user.age++;
}
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <h2>Пользователи</h2>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.name }} - {{ user.age }} лет
        <button @click="updateAge(user.id)">+1 год</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const users = ref([
  { id: 1, name: 'Анна', age: 25 },
  { id: 2, name: 'Борис', age: 30 }
]);

// Решение 1: Находим пользователя в реактивном массиве
function updateAge(userId) {
  const user = users.value.find(u => u.id === userId);
  if (user) {
    user.age++;
  }
}

// Решение 2: Создаем новый массив (иммутабельный подход)
function updateAge(userId) {
  users.value = users.value.map(user => 
    user.id === userId 
      ? { ...user, age: user.age + 1 }
      : user
  );
}

// Решение 3: Используем индекс
function updateAge(index) {
  users.value[index].age++;
}
</script>
```

## Задача 2: Исправить computed свойство

### ❌ Проблемный код

```vue
<template>
  <div>
    <input v-model="searchQuery" placeholder="Поиск...">
    <ul>
      <li v-for="item in filteredItems" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const searchQuery = ref('');
const items = ref([
  { id: 1, name: 'Яблоко' },
  { id: 2, name: 'Банан' },
  { id: 3, name: 'Апельсин' }
]);

// Проблема: computed с побочными эффектами
const filteredItems = computed(() => {
  console.log('Фильтрация...'); // Побочный эффект - логирование
  const query = searchQuery.value.toLowerCase();
  
  // Проблема: мутация исходного массива
  return items.value.filter(item => {
    item.highlighted = item.name.toLowerCase().includes(query);
    return item.highlighted;
  });
});
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <input v-model="searchQuery" placeholder="Поиск...">
    <ul>
      <li 
        v-for="item in filteredItems" 
        :key="item.id"
        :class="{ highlighted: isHighlighted(item) }"
      >
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const searchQuery = ref('');
const items = ref([
  { id: 1, name: 'Яблоко' },
  { id: 2, name: 'Банан' },
  { id: 3, name: 'Апельсин' }
]);

// Решение: computed без побочных эффектов
const filteredItems = computed(() => {
  const query = searchQuery.value.toLowerCase();
  
  if (!query) return items.value;
  
  return items.value.filter(item => 
    item.name.toLowerCase().includes(query)
  );
});

// Отдельная функция для проверки подсветки
function isHighlighted(item) {
  const query = searchQuery.value.toLowerCase();
  return item.name.toLowerCase().includes(query);
}

// Или создаем отдельный computed для подсветки
const highlightedIds = computed(() => {
  const query = searchQuery.value.toLowerCase();
  return new Set(
    items.value
      .filter(item => item.name.toLowerCase().includes(query))
      .map(item => item.id)
  );
});
</script>
```

## Задача 3: Исправить watch

### ❌ Проблемный код

```vue
<template>
  <div>
    <input v-model="user.name" placeholder="Имя">
    <input v-model="user.email" placeholder="Email">
    <p>Изменений: {{ changeCount }}</p>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const user = ref({
  name: '',
  email: ''
});

const changeCount = ref(0);

// Проблема: watch не срабатывает при изменении вложенных свойств
watch(user, () => {
  changeCount.value++;
});
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <input v-model="user.name" placeholder="Имя">
    <input v-model="user.email" placeholder="Email">
    <p>Изменений: {{ changeCount }}</p>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const user = ref({
  name: '',
  email: ''
});

const changeCount = ref(0);

// Решение 1: Добавляем deep: true
watch(user, () => {
  changeCount.value++;
}, { deep: true });

// Решение 2: Следим за конкретными свойствами
watch(
  () => [user.value.name, user.value.email],
  () => {
    changeCount.value++;
  }
);

// Решение 3: Используем reactive вместо ref
import { reactive, watch } from 'vue';

const user = reactive({
  name: '',
  email: ''
});

watch(
  () => ({ ...user }), // Создаем копию для сравнения
  () => {
    changeCount.value++;
  }
);
</script>
```

## Задача 4: Исправить v-model на компоненте

### ❌ Проблемный код

```vue
<!-- ParentComponent.vue -->
<template>
  <CustomInput v-model="message" />
  <p>Сообщение: {{ message }}</p>
</template>

<script setup>
import { ref } from 'vue';
import CustomInput from './CustomInput.vue';

const message = ref('');
</script>

<!-- CustomInput.vue -->
<template>
  <input 
    :value="modelValue" 
    @input="modelValue = $event.target.value"
  >
</template>

<script setup>
// Проблема: нельзя напрямую изменять props
defineProps(['modelValue']);
</script>
```

### ✅ Исправленный код

```vue
<!-- CustomInput.vue - Решение 1: emit -->
<template>
  <input 
    :value="modelValue" 
    @input="$emit('update:modelValue', $event.target.value)"
  >
</template>

<script setup>
defineProps(['modelValue']);
defineEmits(['update:modelValue']);
</script>

<!-- CustomInput.vue - Решение 2: computed с get/set -->
<template>
  <input v-model="value">
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps(['modelValue']);
const emit = defineEmits(['update:modelValue']);

const value = computed({
  get() {
    return props.modelValue;
  },
  set(newValue) {
    emit('update:modelValue', newValue);
  }
});
</script>

<!-- CustomInput.vue - Решение 3: useVModel (VueUse) -->
<template>
  <input v-model="value">
</template>

<script setup>
import { useVModel } from '@vueuse/core';

const props = defineProps(['modelValue']);
const emit = defineEmits(['update:modelValue']);

const value = useVModel(props, 'modelValue', emit);
</script>
```

## Задача 5: Исправить lifecycle hooks

### ❌ Проблемный код

```vue
<template>
  <div ref="chartContainer"></div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Chart from 'chart.js/auto';

const chartContainer = ref(null);
let chart = null;

// Проблема: chart не очищается при размонтировании
onMounted(() => {
  chart = new Chart(chartContainer.value, {
    type: 'bar',
    data: {
      labels: ['Январь', 'Февраль', 'Март'],
      datasets: [{
        label: 'Продажи',
        data: [12, 19, 3]
      }]
    }
  });
});
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div ref="chartContainer"></div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import Chart from 'chart.js/auto';

const chartContainer = ref(null);
let chart = null;

onMounted(() => {
  if (chartContainer.value) {
    chart = new Chart(chartContainer.value, {
      type: 'bar',
      data: {
        labels: ['Январь', 'Февраль', 'Март'],
        datasets: [{
          label: 'Продажи',
          data: [12, 19, 3]
        }]
      }
    });
  }
});

// Решение: очищаем ресурсы
onBeforeUnmount(() => {
  if (chart) {
    chart.destroy();
    chart = null;
  }
});
</script>
```

## Задача 6: Исправить async компонент

### ❌ Проблемный код

```vue
<template>
  <div>
    <h2>Пользователь</h2>
    <p>Имя: {{ user.name }}</p>
    <p>Email: {{ user.email }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const user = ref({});

// Проблема: нет обработки загрузки и ошибок
onMounted(async () => {
  const response = await fetch('/api/user');
  user.value = await response.json();
});
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <div v-if="loading" class="loading">
      Загрузка...
    </div>
    
    <div v-else-if="error" class="error">
      Ошибка: {{ error.message }}
      <button @click="loadUser">Повторить</button>
    </div>
    
    <div v-else-if="user">
      <h2>Пользователь</h2>
      <p>Имя: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const user = ref(null);
const loading = ref(false);
const error = ref(null);

async function loadUser() {
  loading.value = true;
  error.value = null;
  
  try {
    const response = await fetch('/api/user');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    user.value = await response.json();
  } catch (e) {
    error.value = e;
    console.error('Failed to load user:', e);
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadUser();
});
</script>

<style scoped>
.loading {
  padding: 20px;
  text-align: center;
  color: #666;
}

.error {
  padding: 20px;
  background: #fee;
  border: 1px solid #fcc;
  border-radius: 4px;
  color: #c00;
}
</style>
```

## Задача 7: Исправить provide/inject

### ❌ Проблемный код

```vue
<!-- App.vue -->
<template>
  <UserProfile />
</template>

<script setup>
import { provide } from 'vue';
import UserProfile from './UserProfile.vue';

// Проблема: provide не реактивен
const user = {
  name: 'Анна',
  role: 'admin'
};

provide('user', user);

// Изменение не отразится в дочерних компонентах
setTimeout(() => {
  user.name = 'Борис';
}, 2000);
</script>

<!-- UserProfile.vue -->
<template>
  <div>{{ user.name }}</div>
</template>

<script setup>
import { inject } from 'vue';

const user = inject('user');
</script>
```

### ✅ Исправленный код

```vue
<!-- App.vue - Решение 1: ref/reactive -->
<template>
  <UserProfile />
</template>

<script setup>
import { provide, ref } from 'vue';
import UserProfile from './UserProfile.vue';

const user = ref({
  name: 'Анна',
  role: 'admin'
});

provide('user', user);

// Теперь изменение отразится
setTimeout(() => {
  user.value.name = 'Борис';
}, 2000);
</script>

<!-- UserProfile.vue -->
<template>
  <div>{{ user.name }}</div>
</template>

<script setup>
import { inject } from 'vue';

const user = inject('user');
</script>

<!-- App.vue - Решение 2: readonly для защиты -->
<template>
  <UserProfile />
</template>

<script setup>
import { provide, ref, readonly } from 'vue';
import UserProfile from './UserProfile.vue';

const user = ref({
  name: 'Анна',
  role: 'admin'
});

// Предоставляем readonly версию
provide('user', readonly(user));

// Также предоставляем методы для изменения
provide('updateUser', (updates) => {
  user.value = { ...user.value, ...updates };
});
</script>

<!-- UserProfile.vue -->
<template>
  <div>
    {{ user.name }}
    <button @click="updateUser({ name: 'Борис' })">
      Изменить имя
    </button>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const user = inject('user');
const updateUser = inject('updateUser');
</script>
```

## Задача 8: Исправить key в v-for

### ❌ Проблемный код

```vue
<template>
  <div>
    <button @click="shuffleItems">Перемешать</button>
    <ul>
      <li v-for="(item, index) in items" :key="index">
        <input v-model="item.value" :placeholder="item.name">
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const items = ref([
  { id: 1, name: 'Первый', value: '' },
  { id: 2, name: 'Второй', value: '' },
  { id: 3, name: 'Третий', value: '' }
]);

// Проблема: при перемешивании значения input остаются на месте
function shuffleItems() {
  items.value = items.value.sort(() => Math.random() - 0.5);
}
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <button @click="shuffleItems">Перемешать</button>
    <ul>
      <!-- Решение: используем уникальный id вместо index -->
      <li v-for="item in items" :key="item.id">
        <input v-model="item.value" :placeholder="item.name">
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const items = ref([
  { id: 1, name: 'Первый', value: '' },
  { id: 2, name: 'Второй', value: '' },
  { id: 3, name: 'Третий', value: '' }
]);

function shuffleItems() {
  items.value = [...items.value].sort(() => Math.random() - 0.5);
}
</script>
```

## Задача 9: Исправить teleport

### ❌ Проблемный код

```vue
<template>
  <div>
    <button @click="showModal = true">Открыть модалку</button>
    
    <!-- Проблема: модалка рендерится внутри компонента,
         может быть перекрыта z-index родителя -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <h2>Модальное окно</h2>
        <button @click="showModal = false">Закрыть</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const showModal = ref(false);
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
</style>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <button @click="showModal = true">Открыть модалку</button>
    
    <!-- Решение: используем teleport для рендера в body -->
    <Teleport to="body">
      <div v-if="showModal" class="modal" @click.self="showModal = false">
        <div class="modal-content">
          <h2>Модальное окно</h2>
          <p>Нажмите ESC или кликните вне окна для закрытия</p>
          <button @click="showModal = false">Закрыть</button>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const showModal = ref(false);

// Закрытие по ESC
watch(showModal, (isOpen) => {
  if (isOpen) {
    const handleEscape = (e) => {
      if (e.key === 'Escape') {
        showModal.value = false;
      }
    };
    
    document.addEventListener('keydown', handleEscape);
    
    // Очистка при закрытии
    return () => {
      document.removeEventListener('keydown', handleEscape);
    };
  }
});

// Блокировка скролла body при открытой модалке
watch(showModal, (isOpen) => {
  if (isOpen) {
    document.body.style.overflow = 'hidden';
  } else {
    document.body.style.overflow = '';
  }
});
</script>

<style>
/* Не scoped, так как модалка в body */
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
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
}
</style>
```

## Задача 10: Исправить производительность

### ❌ Проблемный код

```vue
<template>
  <div>
    <input v-model="searchQuery" placeholder="Поиск...">
    
    <!-- Проблема: фильтрация в template вызывается при каждом рендере -->
    <ul>
      <li v-for="item in items.filter(i => 
        i.name.toLowerCase().includes(searchQuery.toLowerCase())
      )" :key="item.id">
        {{ item.name }} - {{ formatPrice(item.price) }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const searchQuery = ref('');
const items = ref([
  { id: 1, name: 'Товар 1', price: 1000 },
  { id: 2, name: 'Товар 2', price: 2000 },
  // ... 1000 товаров
]);

// Проблема: функция вызывается для каждого элемента при каждом рендере
function formatPrice(price) {
  return new Intl.NumberFormat('ru-RU', {
    style: 'currency',
    currency: 'RUB'
  }).format(price);
}
</script>
```

### ✅ Исправленный код

```vue
<template>
  <div>
    <input v-model="searchQuery" placeholder="Поиск...">
    
    <!-- Решение: используем computed -->
    <ul>
      <li v-for="item in filteredItems" :key="item.id">
        {{ item.name }} - {{ item.formattedPrice }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const searchQuery = ref('');
const items = ref([
  { id: 1, name: 'Товар 1', price: 1000 },
  { id: 2, name: 'Товар 2', price: 2000 },
  // ... 1000 товаров
]);

// Создаем formatter один раз
const priceFormatter = new Intl.NumberFormat('ru-RU', {
  style: 'currency',
  currency: 'RUB'
});

// Computed для фильтрации и форматирования
const filteredItems = computed(() => {
  const query = searchQuery.value.toLowerCase();
  
  return items.value
    .filter(item => item.name.toLowerCase().includes(query))
    .map(item => ({
      ...item,
      formattedPrice: priceFormatter.format(item.price)
    }));
});

// Или с виртуальным скроллингом для больших списков
import { useVirtualList } from '@vueuse/core';

const { list, containerProps, wrapperProps } = useVirtualList(
  filteredItems,
  {
    itemHeight: 50,
  }
);
</script>
```

## Частые ошибки в Vue.js

### 1. Прямая мутация props

```vue
<!-- ❌ Плохо -->
<script setup>
const props = defineProps(['value']);

function update() {
  props.value = 'new value'; // Ошибка!
}
</script>

<!-- ✅ Хорошо -->
<script setup>
const props = defineProps(['value']);
const emit = defineEmits(['update:value']);

function update() {
  emit('update:value', 'new value');
}
</script>
```

### 2. Забыли .value для ref

```vue
<!-- ❌ Плохо -->
<script setup>
import { ref } from 'vue';

const count = ref(0);

function increment() {
  count++; // Ошибка! Нужно count.value++
}
</script>

<!-- ✅ Хорошо -->
<script setup>
import { ref } from 'vue';

const count = ref(0);

function increment() {
  count.value++;
}
</script>
```

### 3. Неправильная работа с массивами

```vue
<!-- ❌ Плохо -->
<script setup>
import { ref } from 'vue';

const items = ref([1, 2, 3]);

function addItem() {
  items.push(4); // Ошибка! Нужно items.value.push(4)
}
</script>

<!-- ✅ Хорошо -->
<script setup>
import { ref } from 'vue';

const items = ref([1, 2, 3]);

function addItem() {
  items.value.push(4);
}
</script>
```

## Советы по отладке Vue.js

### 1. Используйте Vue DevTools

- Инспектируйте компоненты и их состояние
- Отслеживайте события
- Проверяйте производительность

### 2. Добавьте логирование в watch

```javascript
watch(
  () => someValue.value,
  (newVal, oldVal) => {
    console.log('Changed from', oldVal, 'to', newVal);
  }
);
```

### 3. Проверяйте реактивность

```javascript
import { isRef, isReactive, toRaw } from 'vue';

console.log('Is ref?', isRef(myValue));
console.log('Is reactive?', isReactive(myObject));
console.log('Raw value:', toRaw(myReactive));
```

### 4. Используйте onErrorCaptured

```vue
<script setup>
import { onErrorCaptured } from 'vue';

onErrorCaptured((err, instance, info) => {
  console.error('Error captured:', err);
  console.log('Component:', instance);
  console.log('Error info:', info);
  
  // Возвращаем false, чтобы остановить распространение
  return false;
});
</script>
```

## Чек-лист для Vue.js кода

- [ ] Все ref используются с `.value` в `<script>`
- [ ] Props не мутируются напрямую
- [ ] Используется правильный `key` в `v-for` (не index)
- [ ] Computed свойства без побочных эффектов
- [ ] Watch с `deep: true` для объектов (если нужно)
- [ ] Очистка ресурсов в `onBeforeUnmount`
- [ ] Обработка loading и error состояний
- [ ] Teleport для модальных окон
- [ ] Оптимизация с computed вместо методов в template
- [ ] Vue DevTools для отладки
