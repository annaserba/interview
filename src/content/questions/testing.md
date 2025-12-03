---
title: "Тестирование Frontend"
description: "Unit, Integration и E2E тестирование JavaScript приложений"
category: "Testing"
difficulty: "medium"
tags: ["testing", "vitest", "jest", "playwright", "vue-test-utils"]
order: 21
---

## Типы тестирования

### Unit тесты
Тестируют отдельные функции и компоненты в изоляции.

### Integration тесты
Тестируют взаимодействие между компонентами и модулями.

### E2E тесты
Тестируют приложение целиком с точки зрения пользователя.

## Vitest / Jest

### Установка Vitest

```bash
npm install -D vitest @vitest/ui
```

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './tests/setup.js'
  }
})
```

### Базовые тесты

```javascript
// utils/math.js
export function add(a, b) {
  return a + b
}

export function multiply(a, b) {
  return a * b
}

// utils/math.test.js
import { describe, it, expect } from 'vitest'
import { add, multiply } from './math'

describe('Math utils', () => {
  it('adds two numbers', () => {
    expect(add(2, 3)).toBe(5)
    expect(add(-1, 1)).toBe(0)
  })
  
  it('multiplies two numbers', () => {
    expect(multiply(2, 3)).toBe(6)
    expect(multiply(0, 5)).toBe(0)
  })
})
```

### Matchers (утверждения)

```javascript
// Равенство
expect(value).toBe(5)                    // строгое равенство ===
expect(value).toEqual({ a: 1 })          // глубокое равенство
expect(value).not.toBe(3)                // отрицание

// Истинность
expect(value).toBeTruthy()
expect(value).toBeFalsy()
expect(value).toBeNull()
expect(value).toBeUndefined()
expect(value).toBeDefined()

// Числа
expect(value).toBeGreaterThan(3)
expect(value).toBeLessThan(10)
expect(value).toBeCloseTo(0.3)           // для float

// Строки
expect(str).toMatch(/pattern/)
expect(str).toContain('substring')

// Массивы
expect(arr).toContain(item)
expect(arr).toHaveLength(3)
expect(arr).toEqual(expect.arrayContaining([1, 2]))

// Объекты
expect(obj).toHaveProperty('key')
expect(obj).toMatchObject({ a: 1 })

// Функции
expect(fn).toThrow()
expect(fn).toThrow('error message')
expect(fn).toHaveBeenCalled()
expect(fn).toHaveBeenCalledWith(arg1, arg2)
```

### Асинхронные тесты

```javascript
// Promises
it('fetches data', async () => {
  const data = await fetchData()
  expect(data).toEqual({ id: 1 })
})

// Или
it('fetches data', () => {
  return fetchData().then(data => {
    expect(data).toEqual({ id: 1 })
  })
})

// Ошибки
it('handles errors', async () => {
  await expect(fetchData()).rejects.toThrow('Network error')
})
```

### Моки (Mocks)

```javascript
import { vi } from 'vitest'

// Мок функции
const mockFn = vi.fn()
mockFn('arg1', 'arg2')

expect(mockFn).toHaveBeenCalled()
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2')
expect(mockFn).toHaveBeenCalledTimes(1)

// Возвращаемое значение
const mockFn = vi.fn().mockReturnValue(42)
expect(mockFn()).toBe(42)

// Разные значения при каждом вызове
const mockFn = vi.fn()
  .mockReturnValueOnce(1)
  .mockReturnValueOnce(2)
  .mockReturnValue(3)

// Асинхронные моки
const mockFn = vi.fn().mockResolvedValue({ data: 'test' })
const result = await mockFn()

// Мок модуля
vi.mock('./api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: 1, name: 'John' })
}))

// Частичный мок
vi.mock('./utils', async () => {
  const actual = await vi.importActual('./utils')
  return {
    ...actual,
    someFunction: vi.fn()
  }
})
```

### Spy (шпионы)

```javascript
import { vi } from 'vitest'

const obj = {
  method: () => 'original'
}

const spy = vi.spyOn(obj, 'method')
obj.method()

expect(spy).toHaveBeenCalled()

// Восстановить оригинал
spy.mockRestore()
```

### Хуки (Setup/Teardown)

```javascript
describe('Database tests', () => {
  let db
  
  // Перед всеми тестами
  beforeAll(async () => {
    db = await connectDB()
  })
  
  // После всех тестов
  afterAll(async () => {
    await db.close()
  })
  
  // Перед каждым тестом
  beforeEach(() => {
    db.clear()
  })
  
  // После каждого теста
  afterEach(() => {
    // cleanup
  })
  
  it('inserts data', () => {
    db.insert({ id: 1 })
    expect(db.find(1)).toBeDefined()
  })
})
```

## Тестирование Vue компонентов

### Установка

```bash
npm install -D @vue/test-utils
```

### Базовый тест компонента

```vue
<!-- Counter.vue -->
<template>
  <div>
    <p>Count: {{ count }}</p>
    <button @click="increment">Increment</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(0)
const increment = () => count.value++
</script>
```

```javascript
// Counter.test.js
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Counter from './Counter.vue'

describe('Counter', () => {
  it('renders count', () => {
    const wrapper = mount(Counter)
    expect(wrapper.text()).toContain('Count: 0')
  })
  
  it('increments count on button click', async () => {
    const wrapper = mount(Counter)
    const button = wrapper.find('button')
    
    await button.trigger('click')
    expect(wrapper.text()).toContain('Count: 1')
    
    await button.trigger('click')
    expect(wrapper.text()).toContain('Count: 2')
  })
})
```

### Props и Events

```vue
<!-- UserCard.vue -->
<template>
  <div class="user-card">
    <h2>{{ user.name }}</h2>
    <button @click="$emit('edit', user.id)">Edit</button>
  </div>
</template>

<script setup>
defineProps({
  user: {
    type: Object,
    required: true
  }
})

defineEmits(['edit'])
</script>
```

```javascript
// UserCard.test.js
import { mount } from '@vue/test-utils'
import UserCard from './UserCard.vue'

describe('UserCard', () => {
  const user = { id: 1, name: 'John' }
  
  it('renders user name', () => {
    const wrapper = mount(UserCard, {
      props: { user }
    })
    expect(wrapper.text()).toContain('John')
  })
  
  it('emits edit event', async () => {
    const wrapper = mount(UserCard, {
      props: { user }
    })
    
    await wrapper.find('button').trigger('click')
    
    expect(wrapper.emitted()).toHaveProperty('edit')
    expect(wrapper.emitted('edit')[0]).toEqual([1])
  })
})
```

### Slots

```javascript
it('renders slot content', () => {
  const wrapper = mount(Component, {
    slots: {
      default: '<p>Default slot</p>',
      header: '<h1>Header slot</h1>'
    }
  })
  
  expect(wrapper.html()).toContain('<p>Default slot</p>')
  expect(wrapper.html()).toContain('<h1>Header slot</h1>')
})
```

### Provide/Inject

```javascript
it('provides data to child', () => {
  const wrapper = mount(ParentComponent, {
    global: {
      provide: {
        theme: 'dark'
      }
    }
  })
  
  expect(wrapper.text()).toContain('dark')
})
```

### Тестирование с Pinia

```javascript
import { setActivePinia, createPinia } from 'pinia'
import { mount } from '@vue/test-utils'

describe('Component with store', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })
  
  it('uses store', () => {
    const wrapper = mount(Component)
    const store = useCounterStore()
    
    store.increment()
    expect(wrapper.text()).toContain('Count: 1')
  })
})
```

### Мокирование API запросов

```javascript
import { vi } from 'vitest'

// Мок fetch
global.fetch = vi.fn().mockResolvedValue({
  json: async () => ({ id: 1, name: 'John' })
})

it('fetches user data', async () => {
  const wrapper = mount(UserProfile)
  
  await wrapper.vm.$nextTick()
  await new Promise(resolve => setTimeout(resolve, 0))
  
  expect(wrapper.text()).toContain('John')
})
```

## E2E тестирование с Playwright

### Установка

```bash
npm init playwright@latest
```

### Базовый тест

```javascript
// tests/example.spec.js
import { test, expect } from '@playwright/test'

test('homepage has title', async ({ page }) => {
  await page.goto('http://localhost:3000')
  
  await expect(page).toHaveTitle(/My App/)
})

test('navigation works', async ({ page }) => {
  await page.goto('http://localhost:3000')
  
  await page.click('text=About')
  await expect(page).toHaveURL(/.*about/)
  
  await expect(page.locator('h1')).toContainText('About Us')
})
```

### Взаимодействие с элементами

```javascript
test('form submission', async ({ page }) => {
  await page.goto('http://localhost:3000/login')
  
  // Заполнение формы
  await page.fill('input[name="email"]', 'user@example.com')
  await page.fill('input[name="password"]', 'password123')
  
  // Клик по кнопке
  await page.click('button[type="submit"]')
  
  // Ожидание навигации
  await page.waitForURL('**/dashboard')
  
  // Проверка
  await expect(page.locator('.welcome')).toContainText('Welcome')
})
```

### Селекторы

```javascript
// По тексту
await page.click('text=Login')

// По role
await page.click('role=button[name="Submit"]')

// CSS селектор
await page.click('.btn-primary')

// XPath
await page.click('//button[contains(text(), "Submit")]')

// Комбинация
await page.click('form >> button.submit')
```

### Ожидания

```javascript
// Ожидание элемента
await page.waitForSelector('.loading', { state: 'hidden' })

// Ожидание навигации
await page.waitForURL('**/dashboard')

// Ожидание ответа API
await page.waitForResponse(resp => 
  resp.url().includes('/api/users') && resp.status() === 200
)

// Кастомное ожидание
await page.waitForFunction(() => 
  document.querySelectorAll('.item').length > 5
)
```

### Скриншоты и видео

```javascript
test('visual test', async ({ page }) => {
  await page.goto('http://localhost:3000')
  
  // Скриншот всей страницы
  await page.screenshot({ path: 'screenshot.png', fullPage: true })
  
  // Скриншот элемента
  await page.locator('.hero').screenshot({ path: 'hero.png' })
})

// playwright.config.js
export default {
  use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  }
}
```

## Лучшие практики

### 1. Arrange-Act-Assert (AAA)

```javascript
it('adds item to cart', () => {
  // Arrange (подготовка)
  const cart = new Cart()
  const item = { id: 1, name: 'Product' }
  
  // Act (действие)
  cart.addItem(item)
  
  // Assert (проверка)
  expect(cart.items).toHaveLength(1)
  expect(cart.items[0]).toEqual(item)
})
```

### 2. Один тест = одна проверка

```javascript
// ❌ Плохо
it('user functionality', () => {
  expect(user.name).toBe('John')
  expect(user.age).toBe(30)
  expect(user.email).toBe('john@example.com')
})

// ✅ Хорошо
it('has correct name', () => {
  expect(user.name).toBe('John')
})

it('has correct age', () => {
  expect(user.age).toBe(30)
})
```

### 3. Избегать implementation details

```javascript
// ❌ Плохо - тестируем внутреннюю реализацию
it('increments counter', () => {
  wrapper.vm.count++
  expect(wrapper.vm.count).toBe(1)
})

// ✅ Хорошо - тестируем поведение
it('increments counter on button click', async () => {
  await wrapper.find('button').trigger('click')
  expect(wrapper.text()).toContain('Count: 1')
})
```

### 4. Использовать data-testid

```vue
<template>
  <button data-testid="submit-button">Submit</button>
</template>
```

```javascript
const button = wrapper.find('[data-testid="submit-button"]')
```

## Вопросы для собеседования

### 1. В чём разница между unit и integration тестами?

**Ответ:**
- **Unit**: тестируют отдельные функции/компоненты в изоляции, быстрые, много моков
- **Integration**: тестируют взаимодействие между модулями, медленнее, меньше моков

### 2. Когда использовать mount vs shallowMount?

**Ответ:**
- **mount**: рендерит компонент со всеми дочерними компонентами
- **shallowMount**: рендерит только сам компонент (дочерние заменяются заглушками)
- Используйте `mount` для integration тестов, `shallowMount` для unit

### 3. Как тестировать асинхронный код?

**Ответ:**
- Использовать `async/await`
- `await wrapper.vm.$nextTick()` для обновления DOM
- Моки с `mockResolvedValue` / `mockRejectedValue`
- `waitFor` для ожидания изменений

### 4. Что такое test coverage и какой процент достаточен?

**Ответ:**
- Coverage показывает процент покрытого кода тестами
- 80-90% — хороший показатель
- 100% не всегда нужно и достижимо
- Важнее качество тестов, чем количество

### 5. Как организовать тесты в большом проекте?

**Ответ:**
- Структура папок повторяет структуру `src`
- Файлы тестов рядом с тестируемым кодом
- Группировка с `describe`
- Переиспользуемые утилиты в `tests/utils`
- Отдельные папки для unit/integration/e2e
