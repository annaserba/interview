---
title: "Аутентификация и авторизация"
description: "Механизмы безопасности, JWT, OAuth, сессии и защита приложений"
category: "Security"
difficulty: "hard"
tags: ["authentication", "authorization", "jwt", "oauth", "security", "sessions"]
order: 34
---

## Основные понятия

### Аутентификация vs Авторизация

**Аутентификация (Authentication)** — проверка личности пользователя.
- "Кто вы?"
- Логин/пароль, биометрия, токены

**Авторизация (Authorization)** — проверка прав доступа.
- "Что вам разрешено делать?"
- Роли, разрешения, политики доступа

```javascript
// Аутентификация
if (user.password === hashedPassword) {
  console.log('Пользователь подтверждён')
}

// Авторизация
if (user.role === 'admin') {
  console.log('Доступ к админ-панели разрешён')
}
```

## Методы аутентификации

### 1. Session-based Authentication

Сервер создаёт сессию и сохраняет её ID в cookie.

```javascript
// Server (Express.js)
const express = require('express')
const session = require('express-session')
const bcrypt = require('bcrypt')

const app = express()

app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true,      // Только HTTPS
    httpOnly: true,    // Недоступна для JS
    maxAge: 24 * 60 * 60 * 1000  // 24 часа
  }
}))

// Логин
app.post('/login', async (req, res) => {
  const { email, password } = req.body
  
  // Найти пользователя
  const user = await User.findOne({ email })
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  // Проверить пароль
  const isValid = await bcrypt.compare(password, user.password)
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  // Создать сессию
  req.session.userId = user.id
  req.session.role = user.role
  
  res.json({ message: 'Logged in successfully' })
})

// Защищённый роут
app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Unauthorized' })
  }
  
  const user = User.findById(req.session.userId)
  res.json(user)
})

// Логаут
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Logout failed' })
    }
    res.clearCookie('connect.sid')
    res.json({ message: 'Logged out' })
  })
})
```

**Преимущества:**
- ✅ Простая реализация
- ✅ Легко отозвать сессию
- ✅ Сервер контролирует состояние

**Недостатки:**
- ❌ Не масштабируется (нужен shared storage)
- ❌ Проблемы с CORS
- ❌ Нагрузка на сервер

### 2. Token-based Authentication (JWT)

Клиент получает токен и отправляет его с каждым запросом.

```javascript
// Server
const jwt = require('jsonwebtoken')
const bcrypt = require('bcrypt')

const SECRET_KEY = 'your-secret-key'

// Логин
app.post('/login', async (req, res) => {
  const { email, password } = req.body
  
  const user = await User.findOne({ email })
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  const isValid = await bcrypt.compare(password, user.password)
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  // Создать JWT токен
  const token = jwt.sign(
    { 
      userId: user.id,
      email: user.email,
      role: user.role
    },
    SECRET_KEY,
    { expiresIn: '24h' }
  )
  
  res.json({ token })
})

// Middleware для проверки токена
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization']
  const token = authHeader && authHeader.split(' ')[1]  // Bearer TOKEN
  
  if (!token) {
    return res.status(401).json({ error: 'Token required' })
  }
  
  jwt.verify(token, SECRET_KEY, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid token' })
    }
    req.user = user
    next()
  })
}

// Защищённый роут
app.get('/profile', authenticateToken, (req, res) => {
  res.json({ user: req.user })
})

// Client
async function fetchProfile() {
  const token = localStorage.getItem('token')
  
  const response = await fetch('/profile', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  })
  
  return response.json()
}
```

**Структура JWT:**
```
header.payload.signature

// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "userId": "123",
  "email": "user@example.com",
  "role": "admin",
  "iat": 1640000000,
  "exp": 1640086400
}

// Signature
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**Преимущества:**
- ✅ Stateless (не нужно хранить на сервере)
- ✅ Масштабируется
- ✅ Работает с CORS
- ✅ Можно использовать между сервисами

**Недостатки:**
- ❌ Сложно отозвать токен
- ❌ Больше размер (передаётся с каждым запросом)
- ❌ Нужно хранить на клиенте

### 3. Refresh Token Pattern

Использование двух токенов: короткий access token и долгий refresh token.

```javascript
// Server
app.post('/login', async (req, res) => {
  const { email, password } = req.body
  
  const user = await User.findOne({ email })
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  // Access token (короткий, 15 минут)
  const accessToken = jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    ACCESS_SECRET,
    { expiresIn: '15m' }
  )
  
  // Refresh token (долгий, 7 дней)
  const refreshToken = jwt.sign(
    { userId: user.id },
    REFRESH_SECRET,
    { expiresIn: '7d' }
  )
  
  // Сохранить refresh token в БД
  await RefreshToken.create({
    userId: user.id,
    token: refreshToken,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  })
  
  res.json({ accessToken, refreshToken })
})

// Обновление access token
app.post('/refresh', async (req, res) => {
  const { refreshToken } = req.body
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token required' })
  }
  
  // Проверить наличие в БД
  const storedToken = await RefreshToken.findOne({ token: refreshToken })
  if (!storedToken) {
    return res.status(403).json({ error: 'Invalid refresh token' })
  }
  
  // Проверить валидность
  jwt.verify(refreshToken, REFRESH_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid refresh token' })
    }
    
    // Создать новый access token
    const accessToken = jwt.sign(
      { userId: user.userId },
      ACCESS_SECRET,
      { expiresIn: '15m' }
    )
    
    res.json({ accessToken })
  })
})

// Логаут (удалить refresh token)
app.post('/logout', async (req, res) => {
  const { refreshToken } = req.body
  await RefreshToken.deleteOne({ token: refreshToken })
  res.json({ message: 'Logged out' })
})

// Client - автоматическое обновление токена
let accessToken = localStorage.getItem('accessToken')
let refreshToken = localStorage.getItem('refreshToken')

async function fetchWithAuth(url, options = {}) {
  // Попытка с текущим токеном
  let response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'Authorization': `Bearer ${accessToken}`
    }
  })
  
  // Если токен истёк, обновить
  if (response.status === 401) {
    const refreshResponse = await fetch('/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refreshToken })
    })
    
    if (refreshResponse.ok) {
      const { accessToken: newToken } = await refreshResponse.json()
      accessToken = newToken
      localStorage.setItem('accessToken', newToken)
      
      // Повторить запрос с новым токеном
      response = await fetch(url, {
        ...options,
        headers: {
          ...options.headers,
          'Authorization': `Bearer ${accessToken}`
        }
      })
    } else {
      // Refresh token тоже истёк - перенаправить на логин
      window.location.href = '/login'
    }
  }
  
  return response
}
```

## OAuth 2.0

Протокол авторизации для делегирования доступа (например, "Войти через Google").

### OAuth Flow (Authorization Code)

```javascript
// 1. Перенаправление на провайдера
app.get('/auth/google', (req, res) => {
  const authUrl = `https://accounts.google.com/o/oauth2/v2/auth?` +
    `client_id=${CLIENT_ID}` +
    `&redirect_uri=${REDIRECT_URI}` +
    `&response_type=code` +
    `&scope=openid email profile`
  
  res.redirect(authUrl)
})

// 2. Callback после авторизации
app.get('/auth/google/callback', async (req, res) => {
  const { code } = req.query
  
  // 3. Обменять code на токен
  const tokenResponse = await fetch('https://oauth2.googleapis.com/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      code,
      client_id: CLIENT_ID,
      client_secret: CLIENT_SECRET,
      redirect_uri: REDIRECT_URI,
      grant_type: 'authorization_code'
    })
  })
  
  const { access_token, id_token } = await tokenResponse.json()
  
  // 4. Получить данные пользователя
  const userResponse = await fetch('https://www.googleapis.com/oauth2/v2/userinfo', {
    headers: { 'Authorization': `Bearer ${access_token}` }
  })
  
  const googleUser = await userResponse.json()
  
  // 5. Найти или создать пользователя
  let user = await User.findOne({ email: googleUser.email })
  if (!user) {
    user = await User.create({
      email: googleUser.email,
      name: googleUser.name,
      avatar: googleUser.picture,
      provider: 'google',
      providerId: googleUser.id
    })
  }
  
  // 6. Создать свою сессию/токен
  const token = jwt.sign({ userId: user.id }, SECRET_KEY, { expiresIn: '24h' })
  
  res.redirect(`/dashboard?token=${token}`)
})
```

### OAuth с библиотекой Passport.js

```javascript
const passport = require('passport')
const GoogleStrategy = require('passport-google-oauth20').Strategy

passport.use(new GoogleStrategy({
    clientID: GOOGLE_CLIENT_ID,
    clientSecret: GOOGLE_CLIENT_SECRET,
    callbackURL: '/auth/google/callback'
  },
  async (accessToken, refreshToken, profile, done) => {
    let user = await User.findOne({ providerId: profile.id })
    
    if (!user) {
      user = await User.create({
        email: profile.emails[0].value,
        name: profile.displayName,
        avatar: profile.photos[0].value,
        provider: 'google',
        providerId: profile.id
      })
    }
    
    done(null, user)
  }
))

app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
)

app.get('/auth/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/dashboard')
  }
)
```

## Хеширование паролей

```javascript
const bcrypt = require('bcrypt')

// Регистрация
app.post('/register', async (req, res) => {
  const { email, password } = req.body
  
  // Валидация пароля
  if (password.length < 8) {
    return res.status(400).json({ error: 'Password too short' })
  }
  
  // Хеширование (salt rounds = 10)
  const hashedPassword = await bcrypt.hash(password, 10)
  
  const user = await User.create({
    email,
    password: hashedPassword
  })
  
  res.json({ message: 'User created' })
})

// Логин
app.post('/login', async (req, res) => {
  const { email, password } = req.body
  
  const user = await User.findOne({ email })
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  // Сравнение
  const isValid = await bcrypt.compare(password, user.password)
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' })
  }
  
  const token = jwt.sign({ userId: user.id }, SECRET_KEY)
  res.json({ token })
})
```

## Авторизация (RBAC)

Role-Based Access Control — контроль доступа на основе ролей.

```javascript
// Модель пользователя с ролями
const userSchema = new Schema({
  email: String,
  password: String,
  role: {
    type: String,
    enum: ['user', 'moderator', 'admin'],
    default: 'user'
  },
  permissions: [String]
})

// Middleware для проверки роли
function requireRole(...roles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' })
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' })
    }
    
    next()
  }
}

// Middleware для проверки разрешений
function requirePermission(...permissions) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' })
    }
    
    const hasPermission = permissions.some(p => 
      req.user.permissions.includes(p)
    )
    
    if (!hasPermission) {
      return res.status(403).json({ error: 'Forbidden' })
    }
    
    next()
  }
}

// Использование
app.get('/admin/users', 
  authenticateToken,
  requireRole('admin'),
  (req, res) => {
    // Только для админов
  }
)

app.delete('/posts/:id',
  authenticateToken,
  requirePermission('posts:delete'),
  (req, res) => {
    // Только с разрешением posts:delete
  }
)

// Более гибкая система
const permissions = {
  user: ['posts:read', 'posts:create', 'profile:edit'],
  moderator: ['posts:read', 'posts:create', 'posts:edit', 'posts:delete', 'comments:moderate'],
  admin: ['*']  // Все разрешения
}

function hasPermission(user, permission) {
  const userPermissions = permissions[user.role]
  return userPermissions.includes('*') || userPermissions.includes(permission)
}
```

## Защита от атак

### CSRF (Cross-Site Request Forgery)

```javascript
const csrf = require('csurf')
const csrfProtection = csrf({ cookie: true })

app.get('/form', csrfProtection, (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() })
})

app.post('/submit', csrfProtection, (req, res) => {
  // Токен проверен автоматически
  res.json({ message: 'Success' })
})

// Client
// <form method="POST" action="/submit">
//   <input type="hidden" name="_csrf" value="{{ csrfToken }}">
// </form>
```

### XSS (Cross-Site Scripting)

```javascript
// ❌ Плохо: уязвимо к XSS
app.get('/search', (req, res) => {
  const query = req.query.q
  res.send(`<h1>Results for: ${query}</h1>`)
})

// ✅ Хорошо: экранирование
const escapeHtml = (str) => {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;')
}

app.get('/search', (req, res) => {
  const query = escapeHtml(req.query.q)
  res.send(`<h1>Results for: ${query}</h1>`)
})

// Client - используйте textContent вместо innerHTML
element.textContent = userInput  // ✅ Безопасно
// element.innerHTML = userInput  // ❌ Опасно
```

### Rate Limiting

```javascript
const rateLimit = require('express-rate-limit')

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 минут
  max: 5,  // Максимум 5 попыток
  message: 'Too many login attempts, please try again later'
})

app.post('/login', loginLimiter, async (req, res) => {
  // Логика логина
})
```

## Вопросы для собеседования

### 1. В чём разница между аутентификацией и авторизацией?

**Ответ:**
- **Аутентификация**: проверка личности ("Кто вы?")
- **Авторизация**: проверка прав доступа ("Что вам разрешено?")

Сначала аутентификация, потом авторизация.

### 2. Session vs JWT: когда что использовать?

**Ответ:**
**Session:**
- Монолитные приложения
- Нужен полный контроль над сессиями
- Легко отзывать доступ

**JWT:**
- Микросервисы, API
- Stateless архитектура
- Мобильные приложения

### 3. Как работает JWT?

**Ответ:**
JWT состоит из трёх частей: header, payload, signature.
- Сервер создаёт токен с данными пользователя
- Подписывает секретным ключом
- Клиент отправляет токен с каждым запросом
- Сервер проверяет подпись

Токен нельзя изменить без знания секрета.

### 4. Зачем нужен Refresh Token?

**Ответ:**
- Access token короткий (15 мин) — меньше риск при утечке
- Refresh token долгий (7 дней) — удобство для пользователя
- Refresh token можно отозвать (хранится в БД)
- Баланс между безопасностью и UX

### 5. Как защититься от XSS?

**Ответ:**
- Экранировать пользовательский ввод
- Использовать `textContent` вместо `innerHTML`
- Content Security Policy (CSP)
- HttpOnly cookies для токенов
- Валидация на сервере

### 6. Что такое CSRF и как защититься?

**Ответ:**
CSRF — атака, когда злоумышленник заставляет пользователя выполнить нежелательное действие.

Защита:
- CSRF токены
- SameSite cookie attribute
- Проверка Origin/Referer заголовков
- Двойная отправка cookie

### 7. Как правильно хранить пароли?

**Ответ:**
- Хешировать с помощью bcrypt/argon2
- Использовать salt (автоматически в bcrypt)
- Никогда не хранить в plain text
- Не использовать MD5/SHA1 (слишком быстрые)

### 8. Где хранить JWT на клиенте?

**Ответ:**
**LocalStorage:**
- ✅ Простой доступ
- ❌ Уязвим к XSS

**HttpOnly Cookie:**
- ✅ Защита от XSS
- ❌ Нужна защита от CSRF

**Memory (переменная):**
- ✅ Самый безопасный
- ❌ Теряется при перезагрузке

Лучший вариант: HttpOnly Cookie + CSRF защита.

### 9. Что такое OAuth 2.0?

**Ответ:**
Протокол авторизации для делегирования доступа. Позволяет приложению получить ограниченный доступ к ресурсам пользователя без передачи пароля.

Роли: Resource Owner, Client, Authorization Server, Resource Server.

### 10. Как реализовать "Запомнить меня"?

**Ответ:**
- Использовать долгий refresh token
- Сохранить в secure HttpOnly cookie
- При каждом запросе проверять и обновлять access token
- Дать возможность отозвать все сессии

```javascript
res.cookie('refreshToken', token, {
  httpOnly: true,
  secure: true,
  maxAge: 30 * 24 * 60 * 60 * 1000  // 30 дней
})
```
