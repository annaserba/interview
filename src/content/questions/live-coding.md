---
title: "Live Coding Tasks"
description: "Практические задачи для лайв-кодинга: исправление багов, рефакторинг, оптимизация"
category: "Лайв-кодинг"
difficulty: "medium"
tags: ["live-coding", "debugging", "refactoring", "optimization"]
order: 16
---

## Что такое лайв-кодинг на собеседовании?

На собеседовании вас могут попросить:
- **Исправить баг** в существующем коде
- **Отрефакторить** плохой код
- **Оптимизировать** медленный код
- **Добавить функциональность** к существующему коду
- **Написать код с нуля** для решения задачи

## Задача 1: Исправить баг с замыканиями

### ❌ Проблемный код

```javascript
// Создаем кнопки с обработчиками
function createButtons() {
  const buttons = [];
  
  for (var i = 0; i < 5; i++) {
    const button = document.createElement('button');
    button.textContent = `Button ${i}`;
    
    button.onclick = function() {
      alert(`Clicked button ${i}`);
    };
    
    buttons.push(button);
  }
  
  return buttons;
}

// Проблема: все кнопки показывают "Clicked button 5"
```

### ✅ Исправленный код

```javascript
// Решение 1: Используем let вместо var
function createButtons() {
  const buttons = [];
  
  for (let i = 0; i < 5; i++) {  // let создает новую область видимости
    const button = document.createElement('button');
    button.textContent = `Button ${i}`;
    
    button.onclick = function() {
      alert(`Clicked button ${i}`);
    };
    
    buttons.push(button);
  }
  
  return buttons;
}

// Решение 2: IIFE (Immediately Invoked Function Expression)
function createButtons() {
  const buttons = [];
  
  for (var i = 0; i < 5; i++) {
    const button = document.createElement('button');
    button.textContent = `Button ${i}`;
    
    (function(index) {
      button.onclick = function() {
        alert(`Clicked button ${index}`);
      };
    })(i);
    
    buttons.push(button);
  }
  
  return buttons;
}

// Решение 3: Используем dataset
function createButtons() {
  const buttons = [];
  
  for (var i = 0; i < 5; i++) {
    const button = document.createElement('button');
    button.textContent = `Button ${i}`;
    button.dataset.index = i;
    
    button.onclick = function() {
      alert(`Clicked button ${this.dataset.index}`);
    };
    
    buttons.push(button);
  }
  
  return buttons;
}
```

## Задача 2: Исправить асинхронный код

### ❌ Проблемный код

```javascript
// Загружаем данные пользователя и его посты
function loadUserData(userId) {
  let user;
  let posts;
  
  fetch(`/api/users/${userId}`)
    .then(response => response.json())
    .then(data => {
      user = data;
    });
  
  fetch(`/api/users/${userId}/posts`)
    .then(response => response.json())
    .then(data => {
      posts = data;
    });
  
  // Проблема: user и posts будут undefined
  return { user, posts };
}
```

### ✅ Исправленный код

```javascript
// Решение 1: async/await
async function loadUserData(userId) {
  const userResponse = await fetch(`/api/users/${userId}`);
  const user = await userResponse.json();
  
  const postsResponse = await fetch(`/api/users/${userId}/posts`);
  const posts = await postsResponse.json();
  
  return { user, posts };
}

// Решение 2: Promise.all (параллельная загрузка)
async function loadUserData(userId) {
  const [userResponse, postsResponse] = await Promise.all([
    fetch(`/api/users/${userId}`),
    fetch(`/api/users/${userId}/posts`)
  ]);
  
  const [user, posts] = await Promise.all([
    userResponse.json(),
    postsResponse.json()
  ]);
  
  return { user, posts };
}

// Решение 3: Promise chaining
function loadUserData(userId) {
  return fetch(`/api/users/${userId}`)
    .then(response => response.json())
    .then(user => {
      return fetch(`/api/users/${userId}/posts`)
        .then(response => response.json())
        .then(posts => ({ user, posts }));
    });
}
```

## Задача 3: Рефакторинг плохого кода

### ❌ Плохой код

```javascript
function processUser(user) {
  if (user) {
    if (user.name) {
      if (user.age) {
        if (user.age >= 18) {
          if (user.email) {
            if (user.email.includes('@')) {
              return {
                name: user.name.toUpperCase(),
                age: user.age,
                email: user.email.toLowerCase(),
                status: 'active'
              };
            } else {
              return null;
            }
          } else {
            return null;
          }
        } else {
          return null;
        }
      } else {
        return null;
      }
    } else {
      return null;
    }
  } else {
    return null;
  }
}
```

### ✅ Отрефакторенный код

```javascript
function processUser(user) {
  // Early returns для валидации
  if (!user?.name || !user?.age || !user?.email) {
    return null;
  }
  
  if (user.age < 18) {
    return null;
  }
  
  if (!user.email.includes('@')) {
    return null;
  }
  
  return {
    name: user.name.toUpperCase(),
    age: user.age,
    email: user.email.toLowerCase(),
    status: 'active'
  };
}

// Еще лучше: с валидацией и обработкой ошибок
function validateUser(user) {
  const errors = [];
  
  if (!user?.name) errors.push('Name is required');
  if (!user?.age) errors.push('Age is required');
  if (user?.age < 18) errors.push('User must be 18 or older');
  if (!user?.email) errors.push('Email is required');
  if (user?.email && !user.email.includes('@')) errors.push('Invalid email');
  
  return errors;
}

function processUser(user) {
  const errors = validateUser(user);
  
  if (errors.length > 0) {
    console.error('Validation errors:', errors);
    return null;
  }
  
  return {
    name: user.name.toUpperCase(),
    age: user.age,
    email: user.email.toLowerCase(),
    status: 'active'
  };
}
```

## Задача 4: Оптимизация медленного кода

### ❌ Медленный код

```javascript
// Поиск дубликатов в массиве (O(n²))
function findDuplicates(arr) {
  const duplicates = [];
  
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
        duplicates.push(arr[i]);
      }
    }
  }
  
  return duplicates;
}

// На массиве из 10000 элементов работает очень медленно
```

### ✅ Оптимизированный код

```javascript
// Решение с Set (O(n))
function findDuplicates(arr) {
  const seen = new Set();
  const duplicates = new Set();
  
  for (const item of arr) {
    if (seen.has(item)) {
      duplicates.add(item);
    } else {
      seen.add(item);
    }
  }
  
  return Array.from(duplicates);
}

// Или с Map для подсчета частоты
function findDuplicates(arr) {
  const frequency = new Map();
  
  for (const item of arr) {
    frequency.set(item, (frequency.get(item) || 0) + 1);
  }
  
  return Array.from(frequency.entries())
    .filter(([_, count]) => count > 1)
    .map(([item]) => item);
}
```

## Задача 5: Исправить memory leak

### ❌ Код с утечкой памяти

```javascript
class DataFetcher {
  constructor() {
    this.data = [];
    this.intervalId = null;
  }
  
  start() {
    this.intervalId = setInterval(() => {
      fetch('/api/data')
        .then(response => response.json())
        .then(data => {
          // Проблема: массив растет бесконечно
          this.data.push(data);
          this.render();
        });
    }, 1000);
  }
  
  render() {
    document.getElementById('output').innerHTML = 
      this.data.map(item => `<div>${item}</div>`).join('');
  }
}

// Проблемы:
// 1. Массив data растет без ограничений
// 2. setInterval не очищается
// 3. Нет обработки ошибок
```

### ✅ Исправленный код

```javascript
class DataFetcher {
  constructor(maxItems = 100) {
    this.data = [];
    this.intervalId = null;
    this.maxItems = maxItems;
    this.abortController = null;
  }
  
  start() {
    this.intervalId = setInterval(async () => {
      try {
        // Отменяем предыдущий запрос, если он еще выполняется
        if (this.abortController) {
          this.abortController.abort();
        }
        
        this.abortController = new AbortController();
        
        const response = await fetch('/api/data', {
          signal: this.abortController.signal
        });
        const data = await response.json();
        
        // Ограничиваем размер массива
        this.data.push(data);
        if (this.data.length > this.maxItems) {
          this.data.shift(); // Удаляем старые данные
        }
        
        this.render();
      } catch (error) {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      }
    }, 1000);
  }
  
  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
    
    if (this.abortController) {
      this.abortController.abort();
      this.abortController = null;
    }
  }
  
  render() {
    const output = document.getElementById('output');
    if (output) {
      output.innerHTML = this.data
        .map(item => `<div>${item}</div>`)
        .join('');
    }
  }
  
  destroy() {
    this.stop();
    this.data = [];
  }
}

// Использование
const fetcher = new DataFetcher(50);
fetcher.start();

// Не забываем очистить при размонтировании
window.addEventListener('beforeunload', () => {
  fetcher.destroy();
});
```

## Задача 6: Исправить баг с this

### ❌ Проблемный код

```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    this.count++;
    console.log(this.count);
  }
}

const counter = new Counter();
const button = document.getElementById('btn');

// Проблема: this будет undefined или window
button.addEventListener('click', counter.increment);
```

### ✅ Исправленный код

```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    this.count++;
    console.log(this.count);
  }
}

const counter = new Counter();
const button = document.getElementById('btn');

// Решение 1: bind
button.addEventListener('click', counter.increment.bind(counter));

// Решение 2: arrow function
button.addEventListener('click', () => counter.increment());

// Решение 3: arrow method в классе
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment = () => {
    this.count++;
    console.log(this.count);
  }
}

const counter = new Counter();
button.addEventListener('click', counter.increment);
```

## Задача 7: Добавить функциональность

### Задание

Есть простой список задач. Добавьте:
1. Фильтрацию (все/активные/завершенные)
2. Счетчик активных задач
3. Кнопку "Очистить завершенные"

### Исходный код

```javascript
class TodoList {
  constructor() {
    this.todos = [];
  }
  
  addTodo(text) {
    this.todos.push({
      id: Date.now(),
      text,
      completed: false
    });
  }
  
  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }
  
  render() {
    const list = document.getElementById('todo-list');
    list.innerHTML = this.todos
      .map(todo => `
        <li>
          <input 
            type="checkbox" 
            ${todo.completed ? 'checked' : ''}
            onchange="todoList.toggleTodo(${todo.id})"
          >
          <span style="${todo.completed ? 'text-decoration: line-through' : ''}">
            ${todo.text}
          </span>
        </li>
      `)
      .join('');
  }
}
```

### ✅ Решение с новой функциональностью

```javascript
class TodoList {
  constructor() {
    this.todos = [];
    this.filter = 'all'; // 'all', 'active', 'completed'
  }
  
  addTodo(text) {
    if (!text.trim()) return;
    
    this.todos.push({
      id: Date.now(),
      text: text.trim(),
      completed: false
    });
    this.render();
  }
  
  toggleTodo(id) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
      this.render();
    }
  }
  
  deleteTodo(id) {
    this.todos = this.todos.filter(t => t.id !== id);
    this.render();
  }
  
  setFilter(filter) {
    this.filter = filter;
    this.render();
  }
  
  clearCompleted() {
    this.todos = this.todos.filter(t => !t.completed);
    this.render();
  }
  
  getFilteredTodos() {
    switch (this.filter) {
      case 'active':
        return this.todos.filter(t => !t.completed);
      case 'completed':
        return this.todos.filter(t => t.completed);
      default:
        return this.todos;
    }
  }
  
  getActiveCount() {
    return this.todos.filter(t => !t.completed).length;
  }
  
  render() {
    const list = document.getElementById('todo-list');
    const counter = document.getElementById('active-count');
    const filteredTodos = this.getFilteredTodos();
    
    // Рендер списка
    list.innerHTML = filteredTodos
      .map(todo => `
        <li class="todo-item ${todo.completed ? 'completed' : ''}">
          <input 
            type="checkbox" 
            ${todo.completed ? 'checked' : ''}
            onchange="todoList.toggleTodo(${todo.id})"
          >
          <span class="todo-text">
            ${this.escapeHtml(todo.text)}
          </span>
          <button onclick="todoList.deleteTodo(${todo.id})">
            ✕
          </button>
        </li>
      `)
      .join('');
    
    // Обновляем счетчик
    const activeCount = this.getActiveCount();
    counter.textContent = `${activeCount} ${activeCount === 1 ? 'задача' : 'задач'}`;
    
    // Обновляем кнопки фильтров
    this.updateFilterButtons();
  }
  
  updateFilterButtons() {
    const filters = ['all', 'active', 'completed'];
    filters.forEach(filter => {
      const button = document.getElementById(`filter-${filter}`);
      if (button) {
        button.classList.toggle('active', this.filter === filter);
      }
    });
  }
  
  escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }
}

// HTML
const html = `
  <div class="todo-app">
    <input 
      type="text" 
      id="todo-input" 
      placeholder="Что нужно сделать?"
      onkeypress="if(event.key==='Enter') {
        todoList.addTodo(this.value);
        this.value = '';
      }"
    >
    
    <ul id="todo-list"></ul>
    
    <div class="todo-footer">
      <span id="active-count">0 задач</span>
      
      <div class="filters">
        <button id="filter-all" onclick="todoList.setFilter('all')">
          Все
        </button>
        <button id="filter-active" onclick="todoList.setFilter('active')">
          Активные
        </button>
        <button id="filter-completed" onclick="todoList.setFilter('completed')">
          Завершенные
        </button>
      </div>
      
      <button onclick="todoList.clearCompleted()">
        Очистить завершенные
      </button>
    </div>
  </div>
`;

const todoList = new TodoList();
```

## Задача 8: Исправить race condition

### ❌ Проблемный код

```javascript
let currentSearchId = 0;

async function search(query) {
  const searchId = ++currentSearchId;
  
  const results = await fetch(`/api/search?q=${query}`)
    .then(r => r.json());
  
  // Проблема: если быстро вводить текст,
  // старые результаты могут прийти позже новых
  displayResults(results);
}

// Пользователь вводит: "ja" -> "jav" -> "java"
// Запросы: "ja" (медленный), "jav" (быстрый), "java" (средний)
// Результаты могут отобразиться: "jav", "java", "ja" (неправильный порядок)
```

### ✅ Исправленный код

```javascript
// Решение 1: Проверка ID запроса
let currentSearchId = 0;

async function search(query) {
  const searchId = ++currentSearchId;
  
  const results = await fetch(`/api/search?q=${query}`)
    .then(r => r.json());
  
  // Показываем только если это последний запрос
  if (searchId === currentSearchId) {
    displayResults(results);
  }
}

// Решение 2: AbortController
let currentController = null;

async function search(query) {
  // Отменяем предыдущий запрос
  if (currentController) {
    currentController.abort();
  }
  
  currentController = new AbortController();
  
  try {
    const results = await fetch(`/api/search?q=${query}`, {
      signal: currentController.signal
    }).then(r => r.json());
    
    displayResults(results);
  } catch (error) {
    if (error.name !== 'AbortError') {
      console.error('Search error:', error);
    }
  }
}

// Решение 3: Debounce + AbortController
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

let currentController = null;

async function searchImmediate(query) {
  if (currentController) {
    currentController.abort();
  }
  
  currentController = new AbortController();
  
  try {
    const results = await fetch(`/api/search?q=${query}`, {
      signal: currentController.signal
    }).then(r => r.json());
    
    displayResults(results);
  } catch (error) {
    if (error.name !== 'AbortError') {
      console.error('Search error:', error);
    }
  }
}

const search = debounce(searchImmediate, 300);
```

## Советы для лайв-кодинга

### Перед началом

1. **Прочитайте задание внимательно** — уточните непонятные моменты
2. **Обсудите подход** — расскажите, как планируете решать
3. **Спросите про edge cases** — что делать с null, пустыми значениями и т.д.
4. **Уточните требования** — производительность, поддержка браузеров

### Во время кодинга

1. **Думайте вслух** — объясняйте, что делаете и почему
2. **Пишите читаемый код** — осмысленные имена, форматирование
3. **Начните с простого** — сначала работающее решение, потом оптимизация
4. **Тестируйте по ходу** — проверяйте промежуточные результаты
5. **Не бойтесь ошибок** — важно, как вы их исправляете

### После решения

1. **Протестируйте edge cases** — null, undefined, пустые массивы
2. **Обсудите оптимизацию** — можно ли улучшить производительность
3. **Спросите фидбек** — что можно было сделать лучше

### Что делать, если застряли

1. **Не паникуйте** — это нормально
2. **Проговорите проблему** — часто помогает найти решение
3. **Попросите подсказку** — это не провал, а сотрудничество
4. **Упростите задачу** — решите частный случай, потом обобщите

## Частые ошибки

### ❌ Плохо

```javascript
// Нет обработки ошибок
async function getData() {
  const data = await fetch('/api/data').then(r => r.json());
  return data;
}

// Мутация входных данных
function sortUsers(users) {
  return users.sort((a, b) => a.name.localeCompare(b.name));
}

// Магические числа
setTimeout(() => doSomething(), 300);

// Неинформативные имена
function f(x) {
  return x.map(i => i * 2);
}
```

### ✅ Хорошо

```javascript
// С обработкой ошибок
async function getData() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch data:', error);
    throw error;
  }
}

// Без мутации
function sortUsers(users) {
  return [...users].sort((a, b) => a.name.localeCompare(b.name));
}

// С константами
const DEBOUNCE_DELAY_MS = 300;
setTimeout(() => doSomething(), DEBOUNCE_DELAY_MS);

// Понятные имена
function doubleNumbers(numbers) {
  return numbers.map(number => number * 2);
}
```

## Чек-лист перед отправкой решения

- [ ] Код работает для основного случая
- [ ] Обработаны edge cases (null, undefined, пустые значения)
- [ ] Нет утечек памяти (очищены таймеры, listeners)
- [ ] Есть обработка ошибок
- [ ] Код читаемый (понятные имена, форматирование)
- [ ] Нет лишнего кода (закомментированного, неиспользуемого)
- [ ] Оптимальная сложность алгоритма
- [ ] Код можно легко тестировать
