---
title: "DOM и события"
description: "Работа с DOM, фазы событий, делегирование, preventDefault и stopPropagation"
category: "Браузерные API"
difficulty: "medium"
tags: ["dom", "events", "delegation", "bubbling"]
order: 7
---

## Работа с DOM

### Поиск элементов

```javascript
// Современные методы
document.querySelector('.class');
document.querySelectorAll('.class');
document.getElementById('id');
document.getElementsByClassName('class');
document.getElementsByTagName('div');

// Поиск внутри элемента
const container = document.querySelector('.container');
const items = container.querySelectorAll('.item');
```

### Создание и изменение элементов

```javascript
// Создание
const div = document.createElement('div');
div.className = 'item';
div.textContent = 'Новый элемент';
div.innerHTML = '<span>HTML контент</span>';

// Добавление в DOM
parent.appendChild(div);
parent.append(div); // Может добавить несколько элементов
parent.prepend(div); // В начало
element.before(div); // Перед элементом
element.after(div); // После элемента

// Удаление
element.remove();
parent.removeChild(element);

// Замена
element.replaceWith(newElement);
```

### Атрибуты и классы

```javascript
// Атрибуты
element.getAttribute('data-id');
element.setAttribute('data-id', '123');
element.removeAttribute('data-id');
element.hasAttribute('data-id');

// Классы
element.classList.add('active');
element.classList.remove('active');
element.classList.toggle('active');
element.classList.contains('active');

// Стили
element.style.color = 'red';
element.style.backgroundColor = 'blue';
```

## Фазы событий

События в DOM проходят три фазы:

1. **Capturing (погружение)** — от `window` к целевому элементу
2. **Target (цель)** — на целевом элементе
3. **Bubbling (всплытие)** — от целевого элемента к `window`

```
window
  ↓ capturing
document
  ↓
html
  ↓
body
  ↓
div
  ↓
button ← target (клик здесь)
  ↑
div
  ↑ bubbling
body
  ↑
html
  ↑
document
  ↑
window
```

### Пример

```javascript
const outer = document.querySelector('.outer');
const inner = document.querySelector('.inner');

// По умолчанию — bubbling (третий параметр false)
outer.addEventListener('click', () => {
  console.log('Outer clicked');
});

inner.addEventListener('click', () => {
  console.log('Inner clicked');
});

// Клик на inner выведет:
// Inner clicked
// Outer clicked

// Capturing фаза (третий параметр true)
outer.addEventListener('click', () => {
  console.log('Outer capturing');
}, true);

// Клик на inner выведет:
// Outer capturing (capturing фаза)
// Inner clicked (target фаза)
// Outer clicked (bubbling фаза)
```

## preventDefault() и stopPropagation()

### preventDefault()

Отменяет действие по умолчанию:

```javascript
// Отменить переход по ссылке
link.addEventListener('click', (e) => {
  e.preventDefault();
  console.log('Ссылка не сработает');
});

// Отменить отправку формы
form.addEventListener('submit', (e) => {
  e.preventDefault();
  // Валидация и отправка через fetch
});

// Отменить контекстное меню
document.addEventListener('contextmenu', (e) => {
  e.preventDefault();
});
```

### stopPropagation()

Останавливает всплытие события:

```javascript
inner.addEventListener('click', (e) => {
  e.stopPropagation(); // Событие не дойдет до outer
  console.log('Inner clicked');
});

outer.addEventListener('click', () => {
  console.log('Outer clicked'); // Не выполнится
});
```

### stopImmediatePropagation()

Останавливает все обработчики, включая на текущем элементе:

```javascript
element.addEventListener('click', (e) => {
  console.log('Обработчик 1');
  e.stopImmediatePropagation();
});

element.addEventListener('click', () => {
  console.log('Обработчик 2'); // Не выполнится
});
```

## Делегирование событий

Вместо добавления обработчика на каждый элемент, добавляем один на родителя.

### Без делегирования (плохо)

```javascript
const buttons = document.querySelectorAll('.button');
buttons.forEach(button => {
  button.addEventListener('click', handleClick);
});
// Проблема: новые кнопки не будут иметь обработчик
```

### С делегированием (хорошо)

```javascript
const container = document.querySelector('.container');

container.addEventListener('click', (e) => {
  // Проверяем, что клик был на кнопке
  if (e.target.matches('.button')) {
    handleClick(e);
  }
  
  // Или с closest для вложенных элементов
  const button = e.target.closest('.button');
  if (button) {
    handleClick(button);
  }
});
```

### Практический пример: список с удалением

```javascript
const list = document.querySelector('.todo-list');

list.addEventListener('click', (e) => {
  // Удаление элемента
  if (e.target.matches('.delete-btn')) {
    const item = e.target.closest('.todo-item');
    item.remove();
  }
  
  // Переключение выполнено/не выполнено
  if (e.target.matches('.checkbox')) {
    const item = e.target.closest('.todo-item');
    item.classList.toggle('completed');
  }
});

// Добавление нового элемента
function addTodo(text) {
  const item = document.createElement('li');
  item.className = 'todo-item';
  item.innerHTML = `
    <input type="checkbox" class="checkbox">
    <span>${text}</span>
    <button class="delete-btn">Удалить</button>
  `;
  list.appendChild(item);
  // Обработчики уже работают благодаря делегированию!
}
```

## Типичные задачи

### 1. Модальное окно

```javascript
const modal = document.querySelector('.modal');
const openBtn = document.querySelector('.open-modal');
const closeBtn = document.querySelector('.close-modal');
const overlay = document.querySelector('.overlay');

function openModal() {
  modal.classList.add('active');
  overlay.classList.add('active');
  document.body.style.overflow = 'hidden';
}

function closeModal() {
  modal.classList.remove('active');
  overlay.classList.remove('active');
  document.body.style.overflow = '';
}

openBtn.addEventListener('click', openModal);
closeBtn.addEventListener('click', closeModal);

// Закрытие по клику на overlay
overlay.addEventListener('click', closeModal);

// Закрытие по Escape
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && modal.classList.contains('active')) {
    closeModal();
  }
});
```

### 2. Табы

```javascript
const tabButtons = document.querySelectorAll('.tab-button');
const tabContents = document.querySelectorAll('.tab-content');

tabButtons.forEach(button => {
  button.addEventListener('click', () => {
    // Убираем active у всех
    tabButtons.forEach(btn => btn.classList.remove('active'));
    tabContents.forEach(content => content.classList.remove('active'));
    
    // Добавляем active к текущему
    button.classList.add('active');
    const tabId = button.dataset.tab;
    document.getElementById(tabId).classList.add('active');
  });
});
```

### 3. Бесконечный скролл

```javascript
let page = 1;
let loading = false;

window.addEventListener('scroll', () => {
  // Проверяем, достигли ли низа страницы
  const { scrollTop, scrollHeight, clientHeight } = document.documentElement;
  
  if (scrollTop + clientHeight >= scrollHeight - 100 && !loading) {
    loadMore();
  }
});

async function loadMore() {
  loading = true;
  
  try {
    const response = await fetch(`/api/items?page=${page}`);
    const items = await response.json();
    
    items.forEach(item => {
      const element = createItemElement(item);
      container.appendChild(element);
    });
    
    page++;
  } catch (error) {
    console.error('Ошибка загрузки:', error);
  } finally {
    loading = false;
  }
}
```

### 4. Drag and Drop

```javascript
let draggedElement = null;

// Начало перетаскивания
document.addEventListener('dragstart', (e) => {
  if (e.target.classList.contains('draggable')) {
    draggedElement = e.target;
    e.target.style.opacity = '0.5';
  }
});

// Конец перетаскивания
document.addEventListener('dragend', (e) => {
  if (e.target.classList.contains('draggable')) {
    e.target.style.opacity = '';
  }
});

// Над зоной сброса
document.addEventListener('dragover', (e) => {
  if (e.target.classList.contains('drop-zone')) {
    e.preventDefault(); // Разрешаем сброс
  }
});

// Сброс
document.addEventListener('drop', (e) => {
  if (e.target.classList.contains('drop-zone')) {
    e.preventDefault();
    e.target.appendChild(draggedElement);
  }
});
```

## Производительность

### Рефлоу и репайнт

**Reflow (рефлоу)** — пересчет геометрии элементов (дорого):
- Изменение размеров, позиции
- Добавление/удаление элементов
- Изменение шрифтов

**Repaint (репайнт)** — перерисовка без изменения геометрии (дешевле):
- Изменение цвета, фона
- Изменение видимости

### Оптимизация

```javascript
// ❌ Плохо — несколько рефлоу
element.style.width = '100px';
element.style.height = '100px';
element.style.margin = '10px';

// ✅ Хорошо — один рефлоу
element.style.cssText = 'width: 100px; height: 100px; margin: 10px;';

// ✅ Или через класс
element.classList.add('styled');

// ❌ Плохо — много операций с DOM
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  container.appendChild(div); // Рефлоу на каждой итерации
}

// ✅ Хорошо — DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  fragment.appendChild(div);
}
container.appendChild(fragment); // Один рефлоу
```

## Практические советы

1. **Используйте делегирование** для динамических элементов
2. **Минимизируйте DOM операции** — используйте DocumentFragment
3. **Кэшируйте элементы** — не ищите их каждый раз
4. **Используйте `closest()`** для поиска родителя
5. **Помните про фазы событий** — capturing редко нужен
6. **Всегда очищайте обработчики** при удалении элементов
