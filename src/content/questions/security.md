---
title: "Безопасность веб-приложений"
description: "XSS, CSRF, защита от атак, безопасное хранение данных"
category: "Браузерные API"
difficulty: "medium"
tags: ["security", "xss", "csrf", "safety"]
order: 14
---

## XSS (Cross-Site Scripting)

### Что это?

Внедрение вредоносного JavaScript кода на страницу через пользовательский ввод.

### Типы XSS

#### 1. Reflected XSS (отраженный)

Вредоносный код передается через URL и отображается на странице.

```javascript
// ❌ Уязвимый код
const searchQuery = new URLSearchParams(window.location.search).get('q');
document.getElementById('result').innerHTML = `Результаты для: ${searchQuery}`;

// Атака: ?q=<script>alert('XSS')</script>
```

#### 2. Stored XSS (хранимый)

Вредоносный код сохраняется в базе данных и отображается всем пользователям.

```javascript
// ❌ Уязвимый код
const comment = getUserComment(); // Из БД
document.getElementById('comments').innerHTML += `<div>${comment}</div>`;

// Если в БД: <script>alert('XSS')</script>
```

#### 3. DOM-based XSS

Атака происходит полностью на клиенте.

```javascript
// ❌ Уязвимый код
const name = location.hash.substring(1);
document.write('Привет, ' + name);

// Атака: #<img src=x onerror=alert('XSS')>
```

### Защита от XSS

#### 1. Используйте textContent вместо innerHTML

```javascript
// ❌ Опасно
element.innerHTML = userInput;

// ✅ Безопасно
element.textContent = userInput;
```

#### 2. Санитизация HTML

```javascript
// Используйте библиотеку DOMPurify
import DOMPurify from 'dompurify';

const dirty = '<img src=x onerror=alert(1)>';
const clean = DOMPurify.sanitize(dirty);
element.innerHTML = clean; // Безопасно
```

#### 3. Экранирование специальных символов

```javascript
function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };
  return text.replace(/[&<>"']/g, m => map[m]);
}

const userInput = '<script>alert("XSS")</script>';
element.innerHTML = escapeHtml(userInput);
// Отобразится как текст: &lt;script&gt;alert("XSS")&lt;/script&gt;
```

#### 4. Content Security Policy (CSP)

```html
<!-- В HTML -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' https://trusted.com">

<!-- Или в HTTP заголовке -->
Content-Security-Policy: default-src 'self'; script-src 'self'
```

```javascript
// Запрещает inline скрипты
// ❌ Не сработает с CSP
<script>alert('XSS')</script>

// ✅ Только внешние скрипты
<script src="/app.js"></script>
```

## CSRF (Cross-Site Request Forgery)

### Что это?

Атака, при которой злоумышленник заставляет пользователя выполнить нежелательное действие на сайте, где он авторизован.

### Пример атаки

```html
<!-- Злоумышленник размещает на своем сайте: -->
<img src="https://bank.com/transfer?to=attacker&amount=1000">

<!-- Если пользователь авторизован на bank.com, 
     запрос выполнится с его cookies -->
```

### Защита от CSRF

#### 1. CSRF токен

```javascript
// Сервер генерирует токен и отправляет клиенту
// Клиент отправляет токен с каждым запросом

// В форме
<form action="/transfer" method="POST">
  <input type="hidden" name="csrf_token" value="random_token_here">
  <!-- остальные поля -->
</form>

// В AJAX запросах
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': getCsrfToken()
  },
  body: JSON.stringify({ to: 'user', amount: 100 })
});

function getCsrfToken() {
  return document.querySelector('meta[name="csrf-token"]').content;
}
```

#### 2. SameSite cookies

```javascript
// Сервер устанавливает cookie с атрибутом SameSite
Set-Cookie: sessionId=abc123; SameSite=Strict; Secure; HttpOnly

// Strict — cookie не отправляется при переходе с других сайтов
// Lax — cookie отправляется только при GET запросах с других сайтов
// None — cookie отправляется всегда (требует Secure)
```

#### 3. Проверка Origin/Referer

```javascript
// На сервере
app.post('/api/transfer', (req, res) => {
  const origin = req.get('Origin');
  const referer = req.get('Referer');
  
  if (!origin || !origin.startsWith('https://mysite.com')) {
    return res.status(403).send('Forbidden');
  }
  
  // Обработка запроса
});
```

#### 4. Двойная отправка cookie

```javascript
// Сервер устанавливает CSRF токен в cookie и требует его в заголовке
// Злоумышленник не может прочитать cookie из-за Same-Origin Policy

fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': getCookie('csrf_token')
  },
  body: JSON.stringify(data)
});
```

## Безопасное хранение данных

### LocalStorage и SessionStorage

```javascript
// ❌ НЕ храните чувствительные данные!
localStorage.setItem('token', 'secret_token'); // Доступно через XSS
localStorage.setItem('password', 'user_password'); // Никогда!

// ✅ Только для UI настроек
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'ru');
```

### Cookies

```javascript
// ❌ Плохо — доступно через JavaScript
document.cookie = 'token=secret_token';

// ✅ Хорошо — HttpOnly (недоступно через JavaScript)
// Устанавливается только на сервере:
Set-Cookie: token=secret_token; HttpOnly; Secure; SameSite=Strict

// Secure — только через HTTPS
// HttpOnly — недоступно через document.cookie
// SameSite — защита от CSRF
```

### Токены авторизации

```javascript
// ❌ Плохо
localStorage.setItem('accessToken', token);

// ✅ Лучше — HttpOnly cookie (устанавливается сервером)
// Или хранить в памяти (теряется при перезагрузке)
let accessToken = null;

async function login(credentials) {
  const response = await fetch('/api/login', {
    method: 'POST',
    body: JSON.stringify(credentials)
  });
  const { token } = await response.json();
  accessToken = token; // В памяти
}

// Refresh token — в HttpOnly cookie
```

## Другие угрозы

### Clickjacking

Атака, при которой пользователь кликает на невидимый элемент.

```html
<!-- Защита: X-Frame-Options -->
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN

<!-- Или CSP -->
Content-Security-Policy: frame-ancestors 'none'
```

```css
/* Защита на клиенте */
html {
  /* Если страница в iframe, скрыть */
}

if (window.top !== window.self) {
  window.top.location = window.self.location;
}
```

### SQL Injection (на бекенде)

```javascript
// ❌ Уязвимо
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ Используйте параметризованные запросы
const query = 'SELECT * FROM users WHERE id = ?';
db.query(query, [userId]);
```

### Open Redirect

```javascript
// ❌ Уязвимо
const redirectUrl = new URLSearchParams(location.search).get('redirect');
window.location = redirectUrl; // Может перенаправить на вредоносный сайт

// ✅ Валидация
const allowedDomains = ['mysite.com', 'api.mysite.com'];
const url = new URL(redirectUrl);

if (allowedDomains.includes(url.hostname)) {
  window.location = redirectUrl;
} else {
  window.location = '/';
}
```

### Prototype Pollution

```javascript
// ❌ Уязвимо
function merge(target, source) {
  for (let key in source) {
    target[key] = source[key];
  }
  return target;
}

const malicious = JSON.parse('{"__proto__": {"isAdmin": true}}');
merge({}, malicious);
// Теперь все объекты имеют isAdmin: true

// ✅ Защита
function merge(target, source) {
  for (let key in source) {
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
      continue;
    }
    target[key] = source[key];
  }
  return target;
}

// Или используйте Object.assign / spread
const result = { ...target, ...source };
```

## Лучшие практики

### 1. Валидация ввода

```javascript
// На клиенте И на сервере
function validateEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

function sanitizeInput(input) {
  // Удалить опасные символы
  return input.replace(/[<>]/g, '');
}
```

### 2. HTTPS везде

```javascript
// Перенаправление на HTTPS
if (location.protocol !== 'https:') {
  location.replace(`https:${location.href.substring(location.protocol.length)}`);
}

// Или на сервере (HSTS)
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### 3. Ограничение rate limiting

```javascript
// На сервере
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100 // Максимум 100 запросов
});

app.use('/api/', limiter);
```

### 4. Логирование и мониторинг

```javascript
// Логировать подозрительную активность
function logSecurityEvent(event) {
  console.error('Security event:', {
    type: event.type,
    user: event.user,
    timestamp: new Date(),
    details: event.details
  });
  
  // Отправить на сервер
  fetch('/api/security-log', {
    method: 'POST',
    body: JSON.stringify(event)
  });
}

// Пример использования
if (failedLoginAttempts > 5) {
  logSecurityEvent({
    type: 'BRUTE_FORCE_ATTEMPT',
    user: username,
    details: { attempts: failedLoginAttempts }
  });
}
```

### 5. Обновление зависимостей

```bash
# Проверка уязвимостей
npm audit

# Автоматическое исправление
npm audit fix

# Обновление зависимостей
npm update
```

## Чек-лист безопасности

### Frontend
- ✅ Используйте `textContent` вместо `innerHTML` для пользовательского ввода
- ✅ Санитизируйте HTML (DOMPurify)
- ✅ Включите CSP
- ✅ Не храните токены в LocalStorage
- ✅ Валидируйте все входные данные
- ✅ Используйте HTTPS

### Backend
- ✅ CSRF токены для форм
- ✅ HttpOnly cookies для токенов
- ✅ SameSite cookies
- ✅ Параметризованные SQL запросы
- ✅ Rate limiting
- ✅ CORS настройки
- ✅ Валидация на сервере

### Общее
- ✅ Регулярные обновления зависимостей
- ✅ Логирование безопасности
- ✅ Мониторинг подозрительной активности
- ✅ Тестирование на уязвимости

## Советы для собеседования

1. **XSS — самая частая уязвимость** — всегда санитизируйте ввод
2. **Никогда не храните токены в LocalStorage** — используйте HttpOnly cookies
3. **CSRF токены обязательны** для изменяющих запросов
4. **CSP защищает от XSS** — знайте базовые директивы
5. **Валидация на клиенте И сервере** — клиент можно обойти
6. **HTTPS обязателен** для production
7. **Обновляйте зависимости** — используйте `npm audit`
