---
title: "Promise и async/await"
description: "Работа с асинхронным кодом, обработка ошибок, Promise.all, Promise.race"
category: "Основы JavaScript"
difficulty: "medium"
tags: ["promises", "async", "await", "asynchronous"]
order: 6
---

## Основы Promise

Promise — это объект, представляющий результат асинхронной операции.

### Три состояния Promise

1. **Pending** — ожидание (начальное состояние)
2. **Fulfilled** — выполнено успешно
3. **Rejected** — выполнено с ошибкой

```javascript
const promise = new Promise((resolve, reject) => {
  // Асинхронная операция
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('Успех!');
    } else {
      reject('Ошибка!');
    }
  }, 1000);
});

promise
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('Завершено'));
```

## Цепочки Promise

```javascript
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error('Ошибка:', error));
```

**Важно!** Всегда возвращайте Promise из `.then()`:

```javascript
// ❌ Неправильно
promise
  .then(data => {
    fetch('/api/other'); // Promise потерян!
  })
  .then(result => console.log(result)); // undefined

// ✅ Правильно
promise
  .then(data => {
    return fetch('/api/other'); // Возвращаем Promise
  })
  .then(response => response.json())
  .then(result => console.log(result));
```

## async/await

Синтаксический сахар над Promise, делающий асинхронный код похожим на синхронный.

```javascript
async function fetchUserPosts() {
  try {
    const userResponse = await fetch('/api/user');
    const user = await userResponse.json();
    
    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();
    
    return posts;
  } catch (error) {
    console.error('Ошибка:', error);
    throw error; // Пробрасываем дальше
  }
}

// Использование
fetchUserPosts()
  .then(posts => console.log(posts))
  .catch(error => console.error(error));
```

### Правила async/await

1. **`await` работает только внутри `async` функций**
2. **`async` функция всегда возвращает Promise**
3. **Используйте `try/catch` для обработки ошибок**

```javascript
async function example() {
  return 'Hello'; // Автоматически обернется в Promise.resolve('Hello')
}

example().then(result => console.log(result)); // 'Hello'
```

## Обработка ошибок

### С Promise

```javascript
fetch('/api/data')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  })
  .catch(error => {
    console.error('Ошибка:', error.message);
  });
```

### С async/await

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка:', error.message);
    // Можно вернуть значение по умолчанию
    return null;
  }
}
```

**Важно!** `fetch` не выбрасывает ошибку при HTTP 404 или 500. Проверяйте `response.ok`!

## Promise.all

Ждет выполнения **всех** Promise. Если хотя бы один отклонен — весь `Promise.all` отклоняется.

```javascript
const promise1 = fetch('/api/users');
const promise2 = fetch('/api/posts');
const promise3 = fetch('/api/comments');

Promise.all([promise1, promise2, promise3])
  .then(responses => {
    // Все запросы выполнены успешно
    return Promise.all(responses.map(r => r.json()));
  })
  .then(([users, posts, comments]) => {
    console.log(users, posts, comments);
  })
  .catch(error => {
    // Любой запрос упал
    console.error('Ошибка:', error);
  });
```

### С async/await

```javascript
async function fetchAllData() {
  try {
    const [users, posts, comments] = await Promise.all([
      fetch('/api/users').then(r => r.json()),
      fetch('/api/posts').then(r => r.json()),
      fetch('/api/comments').then(r => r.json())
    ]);
    
    return { users, posts, comments };
  } catch (error) {
    console.error('Ошибка:', error);
  }
}
```

## Promise.allSettled

Ждет выполнения **всех** Promise, независимо от результата (fulfilled или rejected).

```javascript
const promises = [
  fetch('/api/users'),
  fetch('/api/invalid-url'), // Упадет
  fetch('/api/posts')
];

Promise.allSettled(promises)
  .then(results => {
    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        console.log(`Promise ${index} успешен:`, result.value);
      } else {
        console.log(`Promise ${index} отклонен:`, result.reason);
      }
    });
  });
```

## Promise.race

Возвращает результат **первого** завершенного Promise (fulfilled или rejected).

```javascript
const timeout = new Promise((_, reject) => {
  setTimeout(() => reject(new Error('Timeout')), 5000);
});

const fetchData = fetch('/api/data');

Promise.race([fetchData, timeout])
  .then(response => response.json())
  .catch(error => console.error('Ошибка или таймаут:', error));
```

## Promise.any

Возвращает первый **успешный** Promise. Отклоняется только если все Promise отклонены.

```javascript
const promises = [
  fetch('/api/server1/data'),
  fetch('/api/server2/data'),
  fetch('/api/server3/data')
];

Promise.any(promises)
  .then(response => response.json())
  .then(data => console.log('Данные с первого ответившего сервера:', data))
  .catch(error => console.error('Все серверы недоступны'));
```

## Последовательное выполнение Promise

### Проблема: параллельное выполнение в цикле

```javascript
// ❌ Все запросы выполнятся параллельно
async function processUsers(userIds) {
  const results = [];
  for (const id of userIds) {
    const user = await fetch(`/api/users/${id}`);
    results.push(await user.json());
  }
  return results;
}
```

### Решение 1: reduce для последовательности

```javascript
function processUsersSequentially(userIds) {
  return userIds.reduce((promiseChain, id) => {
    return promiseChain.then(results => {
      return fetch(`/api/users/${id}`)
        .then(r => r.json())
        .then(user => [...results, user]);
    });
  }, Promise.resolve([]));
}
```

### Решение 2: for...of с await

```javascript
async function processUsersSequentially(userIds) {
  const results = [];
  for (const id of userIds) {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    results.push(user);
  }
  return results;
}
```

### Решение 3: параллельно с Promise.all

```javascript
async function processUsersParallel(userIds) {
  const promises = userIds.map(id => 
    fetch(`/api/users/${id}`).then(r => r.json())
  );
  return Promise.all(promises);
}
```

## Частые ошибки

### Ошибка 1: Забыли return

```javascript
// ❌ Неправильно
async function getData() {
  try {
    const response = await fetch('/api/data');
    response.json(); // Забыли return!
  } catch (error) {
    console.error(error);
  }
}

// ✅ Правильно
async function getData() {
  try {
    const response = await fetch('/api/data');
    return response.json();
  } catch (error) {
    console.error(error);
    throw error;
  }
}
```

### Ошибка 2: Не обработали ошибку

```javascript
// ❌ Unhandled Promise rejection
async function riskyOperation() {
  const data = await fetch('/api/data');
  return data.json();
}

riskyOperation(); // Если упадет — необработанная ошибка

// ✅ Правильно
riskyOperation().catch(error => console.error(error));
```

### Ошибка 3: Лишний await

```javascript
// ❌ Избыточно
async function getData() {
  return await fetch('/api/data'); // await не нужен
}

// ✅ Правильно
async function getData() {
  return fetch('/api/data'); // async функция и так вернет Promise
}

// Но если нужен try/catch, то await нужен:
async function getData() {
  try {
    return await fetch('/api/data'); // await нужен для catch
  } catch (error) {
    console.error(error);
  }
}
```

## Практические советы

1. **Используйте `Promise.all`** для параллельных запросов
2. **Всегда обрабатывайте ошибки** — `.catch()` или `try/catch`
3. **Проверяйте `response.ok`** при работе с `fetch`
4. **Не забывайте `return`** в цепочках Promise
5. **async/await читабельнее**, но Promise цепочки тоже полезны

## Шпаргалка

| Метод | Описание |
|-------|----------|
| `Promise.all()` | Все успешны или первая ошибка |
| `Promise.allSettled()` | Все завершены (успех или ошибка) |
| `Promise.race()` | Первый завершенный (успех или ошибка) |
| `Promise.any()` | Первый успешный или все ошибки |
