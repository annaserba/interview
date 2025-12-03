---
title: "HTTP и сетевые протоколы"
description: "Протокол HTTP, методы запросов, статус-коды, заголовки и безопасность"
category: "Сеть"
difficulty: "medium"
tags: ["http", "networking", "api", "rest", "headers"]
order: 31
---

## Основы HTTP

### Что такое HTTP?

HTTP (HyperText Transfer Protocol) — протокол передачи данных в интернете. Работает по модели клиент-сервер.

**Основные характеристики:**
- **Stateless** — каждый запрос независим
- **Text-based** — читаемый формат
- **Request-Response** — модель взаимодействия
- **Port 80** (HTTP) и **443** (HTTPS)

### Структура HTTP запроса

```
GET /api/users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer token123
Content-Type: application/json

{"name": "John"}
```

**Компоненты:**
1. **Стартовая строка**: метод, путь, версия протокола
2. **Заголовки**: метаданные запроса
3. **Пустая строка**: разделитель
4. **Тело**: данные (опционально)

### Структура HTTP ответа

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 45
Cache-Control: max-age=3600
Set-Cookie: session=abc123

{"id": 1, "name": "John", "email": "john@example.com"}
```

## HTTP методы

### GET

Получение данных. Идемпотентный, безопасный.

```javascript
// Получить список пользователей
fetch('https://api.example.com/users')
  .then(response => response.json())
  .then(data => console.log(data))

// С query параметрами
fetch('https://api.example.com/users?page=1&limit=10')
```

### POST

Создание ресурса. Не идемпотентный.

```javascript
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'John',
    email: 'john@example.com'
  })
})
```

### PUT

Полное обновление ресурса. Идемпотентный.

```javascript
fetch('https://api.example.com/users/1', {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    id: 1,
    name: 'John Updated',
    email: 'john.new@example.com'
  })
})
```

### PATCH

Частичное обновление ресурса.

```javascript
fetch('https://api.example.com/users/1', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'John Updated'
  })
})
```

### DELETE

Удаление ресурса. Идемпотентный.

```javascript
fetch('https://api.example.com/users/1', {
  method: 'DELETE'
})
```

### OPTIONS

Получение доступных методов для ресурса. Используется в CORS preflight.

```javascript
fetch('https://api.example.com/users', {
  method: 'OPTIONS'
})
```

### HEAD

Как GET, но без тела ответа. Для проверки существования ресурса.

```javascript
fetch('https://api.example.com/users/1', {
  method: 'HEAD'
})
  .then(response => {
    console.log('Exists:', response.ok)
    console.log('Content-Length:', response.headers.get('Content-Length'))
  })
```

## HTTP статус-коды

### 1xx — Информационные

- **100 Continue** — сервер готов принять тело запроса
- **101 Switching Protocols** — переключение протокола (WebSocket)

### 2xx — Успешные

- **200 OK** — запрос выполнен успешно
- **201 Created** — ресурс создан
- **202 Accepted** — запрос принят, но ещё обрабатывается
- **204 No Content** — успешно, но нет содержимого

### 3xx — Перенаправления

- **301 Moved Permanently** — постоянное перенаправление
- **302 Found** — временное перенаправление
- **304 Not Modified** — ресурс не изменился (кэш)
- **307 Temporary Redirect** — временное перенаправление (сохраняет метод)
- **308 Permanent Redirect** — постоянное перенаправление (сохраняет метод)

### 4xx — Ошибки клиента

- **400 Bad Request** — неверный запрос
- **401 Unauthorized** — требуется аутентификация
- **403 Forbidden** — доступ запрещён
- **404 Not Found** — ресурс не найден
- **405 Method Not Allowed** — метод не поддерживается
- **409 Conflict** — конфликт (например, дубликат)
- **422 Unprocessable Entity** — ошибка валидации
- **429 Too Many Requests** — превышен лимит запросов

### 5xx — Ошибки сервера

- **500 Internal Server Error** — внутренняя ошибка сервера
- **502 Bad Gateway** — неверный ответ от upstream сервера
- **503 Service Unavailable** — сервис недоступен
- **504 Gateway Timeout** — таймаут от upstream сервера

## HTTP заголовки

### Request Headers

```javascript
fetch('https://api.example.com/users', {
  headers: {
    // Аутентификация
    'Authorization': 'Bearer token123',
    
    // Тип контента
    'Content-Type': 'application/json',
    
    // Принимаемые форматы
    'Accept': 'application/json',
    
    // Язык
    'Accept-Language': 'ru-RU,ru;q=0.9,en;q=0.8',
    
    // Кэширование
    'Cache-Control': 'no-cache',
    
    // User Agent
    'User-Agent': 'MyApp/1.0',
    
    // Referer
    'Referer': 'https://example.com/page',
    
    // CORS
    'Origin': 'https://example.com'
  }
})
```

### Response Headers

```javascript
// Сервер устанавливает заголовки
res.setHeader('Content-Type', 'application/json')
res.setHeader('Cache-Control', 'public, max-age=3600')
res.setHeader('ETag', '"abc123"')
res.setHeader('Last-Modified', 'Wed, 21 Oct 2024 07:28:00 GMT')
res.setHeader('Set-Cookie', 'session=abc123; HttpOnly; Secure')
res.setHeader('Access-Control-Allow-Origin', '*')

// Чтение заголовков ответа
fetch('https://api.example.com/users')
  .then(response => {
    console.log('Content-Type:', response.headers.get('Content-Type'))
    console.log('Cache-Control:', response.headers.get('Cache-Control'))
    console.log('ETag:', response.headers.get('ETag'))
    
    return response.json()
  })
```

### Важные заголовки

**Content-Type:**
```
application/json
application/x-www-form-urlencoded
multipart/form-data
text/html
text/plain
```

**Cache-Control:**
```
no-cache          // Проверять с сервером
no-store          // Не кэшировать
public            // Можно кэшировать везде
private           // Только в браузере
max-age=3600      // Время жизни в секундах
must-revalidate   // Проверять после истечения
```

**CORS заголовки:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true
```

## CORS (Cross-Origin Resource Sharing)

### Что такое CORS?

Механизм безопасности браузера, ограничивающий запросы между разными доменами.

### Simple Request

Не требует preflight запроса:
- Методы: GET, HEAD, POST
- Заголовки: Accept, Accept-Language, Content-Language, Content-Type
- Content-Type: application/x-www-form-urlencoded, multipart/form-data, text/plain

```javascript
// Simple request
fetch('https://api.example.com/users')
  .then(response => response.json())
```

### Preflight Request

Для сложных запросов браузер сначала отправляет OPTIONS:

```javascript
// Браузер автоматически отправит OPTIONS
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  body: JSON.stringify({ name: 'John' })
})

// Сервер должен ответить на OPTIONS:
// Access-Control-Allow-Origin: https://example.com
// Access-Control-Allow-Methods: POST
// Access-Control-Allow-Headers: Content-Type, Authorization
```

### Настройка CORS на сервере

```javascript
// Express.js
const cors = require('cors')

// Разрешить все домены
app.use(cors())

// Настроить конкретные домены
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400
}))

// Вручную
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', req.headers.origin)
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE')
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  res.header('Access-Control-Allow-Credentials', 'true')
  
  if (req.method === 'OPTIONS') {
    return res.sendStatus(200)
  }
  
  next()
})
```

## Cookies

### Установка Cookie

```javascript
// Сервер
res.setHeader('Set-Cookie', [
  'session=abc123; HttpOnly; Secure; SameSite=Strict; Max-Age=3600',
  'theme=dark; Path=/; Max-Age=31536000'
])

// Клиент (JavaScript)
document.cookie = 'theme=dark; max-age=31536000; path=/'
```

### Атрибуты Cookie

- **HttpOnly** — недоступна для JavaScript (защита от XSS)
- **Secure** — только через HTTPS
- **SameSite** — защита от CSRF
  - `Strict` — только same-site запросы
  - `Lax` — same-site + top-level navigation
  - `None` — все запросы (требует Secure)
- **Max-Age** — время жизни в секундах
- **Expires** — дата истечения
- **Domain** — домен
- **Path** — путь

### Чтение Cookie

```javascript
// Парсинг cookies
function getCookie(name) {
  const value = `; ${document.cookie}`
  const parts = value.split(`; ${name}=`)
  if (parts.length === 2) {
    return parts.pop().split(';').shift()
  }
}

const theme = getCookie('theme')
```

## HTTPS

### Что такое HTTPS?

HTTP + TLS/SSL шифрование. Защищает от:
- Перехвата данных (Man-in-the-Middle)
- Подмены данных
- Прослушивания

### Процесс установки соединения

1. **Client Hello** — клиент отправляет поддерживаемые алгоритмы
2. **Server Hello** — сервер выбирает алгоритм и отправляет сертификат
3. **Certificate Verification** — клиент проверяет сертификат
4. **Key Exchange** — обмен ключами шифрования
5. **Encrypted Communication** — зашифрованная передача данных

### Mixed Content

```html
<!-- ❌ Небезопасно на HTTPS странице -->
<script src="http://example.com/script.js"></script>
<img src="http://example.com/image.jpg">

<!-- ✅ Безопасно -->
<script src="https://example.com/script.js"></script>
<img src="https://example.com/image.jpg">

<!-- ✅ Protocol-relative URL -->
<script src="//example.com/script.js"></script>
```

## REST API Best Practices

### Именование эндпоинтов

```
✅ Хорошо:
GET    /users              # Список пользователей
GET    /users/123          # Конкретный пользователь
POST   /users              # Создать пользователя
PUT    /users/123          # Обновить пользователя
DELETE /users/123          # Удалить пользователя
GET    /users/123/posts    # Посты пользователя

❌ Плохо:
GET    /getUsers
POST   /createUser
GET    /user/123/getPosts
```

### Версионирование API

```
# URL versioning
https://api.example.com/v1/users
https://api.example.com/v2/users

# Header versioning
GET /users
Accept: application/vnd.api.v1+json

# Query parameter
GET /users?version=1
```

### Пагинация

```javascript
// Offset-based
GET /users?offset=20&limit=10

// Cursor-based
GET /users?cursor=abc123&limit=10

// Page-based
GET /users?page=3&per_page=10

// Response
{
  "data": [...],
  "pagination": {
    "total": 100,
    "page": 3,
    "per_page": 10,
    "total_pages": 10,
    "next": "/users?page=4",
    "prev": "/users?page=2"
  }
}
```

### Фильтрация и сортировка

```javascript
// Фильтрация
GET /users?status=active&role=admin

// Сортировка
GET /users?sort=name&order=asc
GET /users?sort=-created_at  // - для desc

// Поиск
GET /users?q=john

// Выбор полей
GET /users?fields=id,name,email
```

## TLS (Transport Layer Security)

### Что такое TLS?

TLS — криптографический протокол для безопасной передачи данных. Преемник SSL.

**Версии:**
- TLS 1.0 (1999) — устарел
- TLS 1.1 (2006) — устарел
- TLS 1.2 (2008) — широко используется
- TLS 1.3 (2018) — современный стандарт

### TLS Handshake (TLS 1.2)

```
Client                                Server

1. ClientHello
   (версии TLS, cipher suites) ────────>

                                2. ServerHello
                                   (выбранная версия, cipher)
                                   Certificate
                                   ServerKeyExchange
                            <──────── ServerHelloDone

3. ClientKeyExchange
   ChangeCipherSpec
   Finished                 ────────>

                                4. ChangeCipherSpec
                            <──────── Finished

5. Encrypted Application Data <─────> Encrypted Application Data
```

**Шаги:**
1. **Client Hello**: клиент отправляет поддерживаемые версии и шифры
2. **Server Hello**: сервер выбирает версию и шифр, отправляет сертификат
3. **Key Exchange**: обмен ключами для шифрования
4. **Finished**: подтверждение установки соединения
5. **Application Data**: зашифрованная передача данных

### TLS 1.3 улучшения

```
Client                                Server

1. ClientHello
   + KeyShare              ────────>

                                2. ServerHello
                                   + KeyShare
                                   {Certificate}
                                   {CertificateVerify}
                            <──────── {Finished}

3. {Finished}              ────────>

4. Application Data        <─────> Application Data
```

**Преимущества TLS 1.3:**
- ⚡ **1-RTT handshake** (вместо 2-RTT в TLS 1.2)
- 🔒 **0-RTT resumption** — мгновенное возобновление
- 🛡️ **Удалены слабые шифры** (RC4, MD5, SHA-1)
- 🔐 **Forward Secrecy** обязателен
- 📦 **Меньше сообщений** в handshake

### Cipher Suites

```
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256

TLS_          - протокол
ECDHE         - key exchange (Elliptic Curve Diffie-Hellman Ephemeral)
RSA           - аутентификация (подпись сертификата)
AES_128_GCM   - шифрование (Advanced Encryption Standard, 128 бит, GCM режим)
SHA256        - хеш для HMAC
```

**Современные cipher suites (TLS 1.3):**
```
TLS_AES_128_GCM_SHA256
TLS_AES_256_GCM_SHA384
TLS_CHACHA20_POLY1305_SHA256
```

## QUIC Protocol

### Что такое QUIC?

QUIC (Quick UDP Internet Connections) — транспортный протокол от Google, работающий поверх UDP.

**Ключевые особенности:**
- 🚀 Быстрое установление соединения (0-RTT)
- 📦 Встроенное шифрование (TLS 1.3)
- 🔄 Multiplexing без head-of-line blocking
- 📡 Connection migration (смена IP без разрыва)
- 🛠️ Улучшенная обработка потерь пакетов

### QUIC vs TCP + TLS

```
TCP + TLS 1.2:
Client ─────> SYN
       <───── SYN-ACK
       ─────> ACK
       ─────> ClientHello
       <───── ServerHello, Certificate
       ─────> KeyExchange, Finished
       <───── Finished
       ─────> HTTP Request
       <───── HTTP Response

Total: 3-4 RTT до первого байта данных

QUIC:
Client ─────> ClientHello + HTTP Request
       <───── ServerHello + HTTP Response

Total: 1 RTT (или 0-RTT при повторном подключении)
```

### QUIC Streams

```javascript
// Концептуально: множественные потоки в одном соединении
const connection = new QuicConnection('example.com')

// Поток 1: HTML
const stream1 = connection.createStream()
stream1.send('GET /index.html')

// Поток 2: CSS (параллельно)
const stream2 = connection.createStream()
stream2.send('GET /styles.css')

// Поток 3: JS (параллельно)
const stream3 = connection.createStream()
stream3.send('GET /script.js')

// Потери пакетов в stream1 НЕ блокируют stream2 и stream3
```

### Connection Migration

```
Client меняет сеть (WiFi → 4G):

Old IP: 192.168.1.100
New IP: 10.0.0.50

QUIC Connection ID: abc123 (не меняется!)

Client ─────> Packet with Connection ID: abc123
              From: 10.0.0.50 (новый IP)
       <───── Server продолжает соединение без разрыва

TCP бы разорвал соединение и начал заново
```

## HTTP/3

### Что такое HTTP/3?

HTTP/3 — новая версия HTTP, работающая поверх QUIC вместо TCP.

**Стек протоколов:**
```
HTTP/1.1:  HTTP/1.1 → TCP → IP
HTTP/2:    HTTP/2   → TCP → TLS → IP
HTTP/3:    HTTP/3   → QUIC (с TLS 1.3) → UDP → IP
```

### Преимущества HTTP/3

#### 1. Быстрое установление соединения

```
HTTP/2 (TCP + TLS):
- 1 RTT для TCP handshake
- 1-2 RTT для TLS handshake
- Total: 2-3 RTT

HTTP/3 (QUIC):
- 1 RTT для всего (QUIC + TLS + HTTP)
- 0 RTT при повторном подключении
```

#### 2. Нет Head-of-Line Blocking

```
HTTP/2 (TCP):
Stream 1: [Packet 1] [Packet 2] [LOST] [Packet 4]
Stream 2: [Packet 1] [Packet 2] [Packet 3] [Packet 4]

Packet 3 из Stream 1 потерян → ВСЕ потоки блокируются
TCP ждёт повторной передачи

HTTP/3 (QUIC):
Stream 1: [Packet 1] [Packet 2] [LOST] [Packet 4]
Stream 2: [Packet 1] [Packet 2] [Packet 3] [Packet 4]

Packet 3 из Stream 1 потерян → только Stream 1 блокируется
Stream 2 продолжает работать
```

#### 3. Connection Migration

```javascript
// Пользователь переключается с WiFi на мобильную сеть
// HTTP/2: соединение разрывается, нужно переподключаться
// HTTP/3: соединение продолжается без разрыва

// Пример: загрузка видео
const video = document.querySelector('video')

// HTTP/3 автоматически продолжит загрузку при смене сети
// Пользователь не заметит разрыва
```

### Обнаружение HTTP/3

```
1. DNS запись:
example.com.  IN HTTPS 1 . alpn=h3

2. Alt-Svc заголовок (HTTP/2 → HTTP/3):
HTTP/2 Response:
Alt-Svc: h3=":443"; ma=86400

Клиент: "Ага, сервер поддерживает HTTP/3 на порту 443"
Следующий запрос: использует HTTP/3
```

### Настройка сервера (Nginx)

```nginx
# Nginx 1.25+ с поддержкой HTTP/3
server {
    listen 443 ssl;
    listen 443 quic reuseport;  # HTTP/3
    
    http2 on;  # HTTP/2
    http3 on;  # HTTP/3
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # Сообщить клиенту о поддержке HTTP/3
    add_header Alt-Svc 'h3=":443"; ma=86400';
    
    location / {
        root /var/www/html;
    }
}
```

### Сравнение протоколов

| Характеристика | HTTP/1.1 | HTTP/2 | HTTP/3 |
|----------------|----------|--------|--------|
| **Транспорт** | TCP | TCP | QUIC (UDP) |
| **Формат** | Текстовый | Бинарный | Бинарный |
| **Multiplexing** | ❌ | ✅ | ✅ |
| **Head-of-line blocking** | ✅ (HTTP) | ✅ (TCP) | ❌ |
| **Server Push** | ❌ | ✅ | ✅ |
| **Header Compression** | ❌ | ✅ (HPACK) | ✅ (QPACK) |
| **Connection Setup** | 2-3 RTT | 2-3 RTT | 1 RTT (0 RTT) |
| **Connection Migration** | ❌ | ❌ | ✅ |
| **TLS** | Опционально | Обязательно | Встроено |

### Когда использовать что?

**HTTP/1.1:**
- Легаси системы
- Простые сайты
- Отладка (текстовый формат)

**HTTP/2:**
- Современные веб-приложения
- Множество ресурсов
- Хорошая поддержка браузерами

**HTTP/3:**
- Мобильные приложения (connection migration)
- Видео стриминг
- Real-time приложения
- Плохое качество сети (лучше обработка потерь)

## Вопросы для собеседования

### 1. В чём разница между PUT и PATCH?

**Ответ:**
- **PUT**: полная замена ресурса (все поля)
- **PATCH**: частичное обновление (только изменённые поля)

PUT идемпотентный — повторные запросы дают тот же результат.

### 2. Что такое идемпотентность?

**Ответ:**
Свойство операции, при котором повторное выполнение даёт тот же результат.

**Идемпотентные**: GET, PUT, DELETE, HEAD, OPTIONS
**Не идемпотентные**: POST, PATCH

### 3. Объясните CORS и preflight запрос

**Ответ:**
CORS — механизм безопасности браузера для кросс-доменных запросов.

Preflight (OPTIONS) отправляется для:
- Методов кроме GET, HEAD, POST
- Кастомных заголовков
- Content-Type кроме form-data, urlencoded, text/plain

Сервер должен ответить с Access-Control-* заголовками.

### 4. В чём разница между 401 и 403?

**Ответ:**
- **401 Unauthorized**: не аутентифицирован (нужен логин)
- **403 Forbidden**: аутентифицирован, но нет прав доступа

### 5. Что такое HTTP/2 и его преимущества?

**Ответ:**
HTTP/2 — новая версия протокола с:
- **Multiplexing** — несколько запросов в одном соединении
- **Server Push** — сервер может отправлять ресурсы заранее
- **Header Compression** — сжатие заголовков (HPACK)
- **Binary Protocol** — бинарный формат вместо текстового
- **Prioritization** — приоритизация запросов

### 6. Как работает кэширование в HTTP?

**Ответ:**
**Cache-Control заголовки:**
- `max-age` — время жизни кэша
- `no-cache` — проверять с сервером
- `no-store` — не кэшировать

**Валидация:**
- `ETag` — хэш ресурса
- `Last-Modified` — дата изменения
- `If-None-Match` / `If-Modified-Since` — условные запросы

Сервер отвечает 304 Not Modified если ресурс не изменился.

### 7. Что такое Content Security Policy (CSP)?

**Ответ:**
HTTP заголовок для защиты от XSS и injection атак:

```
Content-Security-Policy: 
  default-src 'self'; 
  script-src 'self' https://trusted.com; 
  style-src 'self' 'unsafe-inline'
```

Ограничивает источники скриптов, стилей, изображений и т.д.

### 8. Как реализовать rate limiting?

**Ответ:**
Ограничение количества запросов от клиента:

```javascript
// Заголовки ответа
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

// При превышении
429 Too Many Requests
Retry-After: 60
```

Алгоритмы: Token Bucket, Leaky Bucket, Fixed Window, Sliding Window.

### 9. В чём разница между HTTP/1.1 и HTTP/2?

**Ответ:**
| HTTP/1.1 | HTTP/2 |
|----------|--------|
| Текстовый | Бинарный |
| 1 запрос = 1 соединение | Multiplexing |
| Head-of-line blocking | Нет blocking |
| Нет приоритизации | Есть приоритизация |
| Нет server push | Есть server push |
| Большие заголовки | Сжатие заголовков |

### 10. Что такое TLS и как работает handshake?

**Ответ:**
TLS — криптографический протокол для безопасной передачи данных.

**TLS 1.2 Handshake:**
1. Client Hello (версии, cipher suites)
2. Server Hello (выбор, сертификат)
3. Key Exchange (обмен ключами)
4. Finished (подтверждение)

**TLS 1.3:** 1-RTT вместо 2-RTT, 0-RTT resumption, удалены слабые шифры.

### 11. В чём преимущества HTTP/3?

**Ответ:**
HTTP/3 работает поверх QUIC (UDP) вместо TCP:

**Преимущества:**
- ⚡ Быстрое соединение (0-1 RTT)
- 🔄 Нет head-of-line blocking
- 📡 Connection migration (смена сети без разрыва)
- 🛠️ Лучшая обработка потерь пакетов
- 🔒 Встроенное шифрование (TLS 1.3)

**Когда использовать:** мобильные приложения, видео, плохая сеть.

### 12. Что такое QUIC?

**Ответ:**
QUIC — транспортный протокол поверх UDP с встроенным TLS 1.3.

**Отличия от TCP:**
- Multiplexing без блокировки потоков
- 0-RTT установка соединения
- Connection ID вместо IP:Port (migration)
- Улучшенный congestion control

HTTP/3 = HTTP/2 семантика + QUIC транспорт.

### 13. Что такое WebSocket и чем отличается от HTTP?

**Ответ:**
WebSocket — протокол для двусторонней связи в реальном времени.

**Отличия:**
- HTTP: request-response, stateless
- WebSocket: persistent connection, bidirectional, stateful

Используется для: чаты, real-time обновления, игры, стриминг.
