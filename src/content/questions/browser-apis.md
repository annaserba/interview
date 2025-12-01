---
title: "Браузерные API и хранение данных"
description: "LocalStorage, SessionStorage, Cookies, fetch, CORS"
category: "Браузерные API"
difficulty: "medium"
tags: ["storage", "fetch", "cors", "browser"]
order: 10
---

## Хранение данных в браузере

### LocalStorage

Хранит данные **без срока истечения** (до очистки браузера).

```javascript
// Сохранение
localStorage.setItem('username', 'Анна');
localStorage.setItem('settings', JSON.stringify({ theme: 'dark' }));

// Получение
const username = localStorage.getItem('username'); // 'Анна'
const settings = JSON.parse(localStorage.getItem('settings'));

// Удаление
localStorage.removeItem('username');

// Очистка всего
localStorage.clear();

// Проверка наличия
if (localStorage.getItem('username')) {
  // ...
}

// Размер хранилища
console.log(localStorage.length);

// Перебор ключей
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  console.log(key, localStorage.getItem(key));
}
```

### SessionStorage

Хранит данные **до закрытия вкладки**.

```javascript
// API идентичен localStorage
sessionStorage.setItem('tempData', 'value');
const data = sessionStorage.getItem('tempData');
sessionStorage.removeItem('tempData');
sessionStorage.clear();
```

### Cookies

Хранят данные с возможностью **отправки на сервер**.

```javascript
// Установка cookie
document.cookie = 'username=Анна';
document.cookie = 'theme=dark; max-age=3600'; // 1 час
document.cookie = 'token=abc123; path=/; secure; httpOnly';

// Чтение всех cookies
console.log(document.cookie); // 'username=Анна; theme=dark'

// Парсинг cookies
function getCookie(name) {
  const matches = document.cookie.match(
    new RegExp('(?:^|; )' + name + '=([^;]*)')
  );
  return matches ? decodeURIComponent(matches[1]) : undefined;
}

const username = getCookie('username');

// Удаление cookie (установка в прошлое)
document.cookie = 'username=; max-age=-1';

// Установка с опциями
function setCookie(name, value, options = {}) {
  options = {
    path: '/',
    ...options
  };

  if (options.expires instanceof Date) {
    options.expires = options.expires.toUTCString();
  }

  let updatedCookie = encodeURIComponent(name) + '=' + encodeURIComponent(value);

  for (let optionKey in options) {
    updatedCookie += '; ' + optionKey;
    let optionValue = options[optionKey];
    if (optionValue !== true) {
      updatedCookie += '=' + optionValue;
    }
  }

  document.cookie = updatedCookie;
}

// Использование
setCookie('user', 'Анна', { 'max-age': 3600, secure: true });
```

## Сравнение способов хранения

| Характеристика | LocalStorage | SessionStorage | Cookies |
|----------------|--------------|----------------|---------|
| **Размер** | ~5-10 MB | ~5-10 MB | ~4 KB |
| **Срок** | Бессрочно | До закрытия вкладки | Настраиваемый |
| **Отправка на сервер** | Нет | Нет | Да (с каждым запросом) |
| **API** | Синхронный | Синхронный | Строка |
| **Доступ** | JavaScript | JavaScript | JavaScript + сервер |
| **Безопасность** | XSS уязвим | XSS уязвим | HttpOnly защищает |

### Когда что использовать?

- **LocalStorage**: настройки UI, кэш данных, оффлайн-режим
- **SessionStorage**: временные данные формы, состояние в рамках сессии
- **Cookies**: аутентификация (токены), отслеживание, серверные настройки

## Fetch API

### Базовое использование

```javascript
// GET запрос
fetch('https://api.example.com/users')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  })
  .then(data => console.log(data))
  .catch(error => console.error('Ошибка:', error));

// С async/await
async function fetchUsers() {
  try {
    const response = await fetch('https://api.example.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка:', error);
  }
}
```

### POST запрос

```javascript
async function createUser(userData) {
  try {
    const response = await fetch('https://api.example.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + token
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка:', error);
    throw error;
  }
}

// Использование
createUser({ name: 'Анна', age: 25 });
```

### Другие методы

```javascript
// PUT (обновление)
fetch('/api/users/1', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Новое имя' })
});

// PATCH (частичное обновление)
fetch('/api/users/1', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ age: 26 })
});

// DELETE
fetch('/api/users/1', {
  method: 'DELETE'
});
```

### Обработка разных типов ответов

```javascript
const response = await fetch('/api/data');

// JSON
const data = await response.json();

// Текст
const text = await response.text();

// Blob (файлы, изображения)
const blob = await response.blob();
const imageUrl = URL.createObjectURL(blob);

// FormData
const formData = await response.formData();

// ArrayBuffer
const buffer = await response.arrayBuffer();
```

### Отмена запроса (AbortController)

```javascript
const controller = new AbortController();
const signal = controller.signal;

// Запрос с возможностью отмены
fetch('/api/data', { signal })
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('Запрос отменен');
    } else {
      console.error('Ошибка:', error);
    }
  });

// Отмена через 5 секунд
setTimeout(() => controller.abort(), 5000);

// Практический пример: отмена предыдущего поиска
let currentController = null;

async function search(query) {
  // Отменяем предыдущий запрос
  if (currentController) {
    currentController.abort();
  }
  
  currentController = new AbortController();
  
  try {
    const response = await fetch(`/api/search?q=${query}`, {
      signal: currentController.signal
    });
    const results = await response.json();
    return results;
  } catch (error) {
    if (error.name !== 'AbortError') {
      console.error('Ошибка:', error);
    }
  }
}
```

## CORS (Cross-Origin Resource Sharing)

### Что такое CORS?

Механизм безопасности браузера, который ограничивает запросы к другим доменам.

```javascript
// Запрос с example.com на api.other.com
fetch('https://api.other.com/data')
  .then(response => response.json())
  .catch(error => {
    // CORS error: No 'Access-Control-Allow-Origin' header
    console.error(error);
  });
```

### Простые запросы (Simple Requests)

Не требуют preflight проверки:
- Методы: `GET`, `POST`, `HEAD`
- Заголовки: `Accept`, `Accept-Language`, `Content-Language`, `Content-Type` (только `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`)

### Preflight запросы

Для сложных запросов браузер сначала отправляет `OPTIONS` запрос:

```javascript
// Этот запрос вызовет preflight
fetch('https://api.other.com/data', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'X-Custom-Header': 'value'
  },
  body: JSON.stringify({ data: 'value' })
});

// Браузер сначала отправит:
// OPTIONS /data
// Access-Control-Request-Method: PUT
// Access-Control-Request-Headers: content-type, x-custom-header

// Сервер должен ответить:
// Access-Control-Allow-Origin: https://example.com
// Access-Control-Allow-Methods: PUT, POST, GET
// Access-Control-Allow-Headers: content-type, x-custom-header
```

### Credentials (куки, авторизация)

```javascript
// Отправка cookies с запросом
fetch('https://api.other.com/data', {
  credentials: 'include' // 'same-origin' (по умолчанию) | 'omit'
});

// Сервер должен ответить:
// Access-Control-Allow-Credentials: true
// Access-Control-Allow-Origin: https://example.com (не *)
```

### Обход CORS (только для разработки!)

```javascript
// 1. Proxy в package.json (Create React App)
{
  "proxy": "https://api.example.com"
}

// 2. CORS proxy (не для production!)
fetch('https://cors-anywhere.herokuapp.com/https://api.example.com/data');

// 3. JSONP (устаревший метод)
function jsonp(url, callback) {
  const script = document.createElement('script');
  script.src = `${url}?callback=${callback}`;
  document.body.appendChild(script);
}
```

## Другие полезные API

### Geolocation

```javascript
if ('geolocation' in navigator) {
  navigator.geolocation.getCurrentPosition(
    position => {
      console.log('Широта:', position.coords.latitude);
      console.log('Долгота:', position.coords.longitude);
    },
    error => {
      console.error('Ошибка:', error.message);
    }
  );
}
```

### Notifications

```javascript
if ('Notification' in window) {
  Notification.requestPermission().then(permission => {
    if (permission === 'granted') {
      new Notification('Заголовок', {
        body: 'Текст уведомления',
        icon: '/icon.png'
      });
    }
  });
}
```

### Clipboard API

```javascript
// Копирование
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log('Скопировано!');
  } catch (error) {
    console.error('Ошибка:', error);
  }
}

// Вставка
async function pasteFromClipboard() {
  try {
    const text = await navigator.clipboard.readText();
    return text;
  } catch (error) {
    console.error('Ошибка:', error);
  }
}
```

## Практические советы

1. **Всегда проверяйте `response.ok`** — fetch не выбрасывает ошибку при 404/500
2. **Используйте LocalStorage для UI настроек**, SessionStorage для временных данных
3. **Не храните чувствительные данные** в LocalStorage (токены → HttpOnly cookies)
4. **Обрабатывайте CORS ошибки** на стороне сервера
5. **Используйте AbortController** для отмены запросов
6. **Всегда сериализуйте данные** для LocalStorage: `JSON.stringify/parse`

## Безопасность

### XSS (Cross-Site Scripting)

```javascript
// ❌ Опасно!
const userInput = '<img src=x onerror=alert(1)>';
element.innerHTML = userInput;

// ✅ Безопасно
element.textContent = userInput;

// Или используйте библиотеки для санитизации
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

### CSRF (Cross-Site Request Forgery)

```javascript
// Защита: CSRF токен в заголовках
fetch('/api/data', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': getCsrfToken(),
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
});
```
