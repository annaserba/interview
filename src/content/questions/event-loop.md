---
title: "Event Loop и асинхронность"
description: "Call stack, Web APIs, callback queue, microtasks и macrotasks"
category: "JavaScript"
difficulty: "hard"
tags: ["event-loop", "async", "promises", "microtasks"]
order: 4
---

## Как работает Event Loop

Event Loop — это механизм, который позволяет JavaScript выполнять асинхронные операции, несмотря на то, что он однопоточный.

### Основные компоненты

1. **Call Stack** — стек вызовов функций
2. **Web APIs** — браузерные API (setTimeout, fetch, DOM events)
3. **Callback Queue (Task Queue)** — очередь макрозадач
4. **Microtask Queue** — очередь микрозадач (Promise, queueMicrotask)
5. **Event Loop** — цикл, который проверяет стек и очереди

## Порядок выполнения

```javascript
console.log('1'); // Синхронный код

setTimeout(() => {
  console.log('2'); // Macrotask
}, 0);

Promise.resolve().then(() => {
  console.log('3'); // Microtask
});

console.log('4'); // Синхронный код

// Вывод: 1, 4, 3, 2
```

### Почему такой порядок?

1. Выполняется весь синхронный код: `1`, `4`
2. Call Stack пуст → проверяем Microtask Queue
3. Выполняем все микрозадачи: `3`
4. Microtask Queue пуст → берем одну задачу из Callback Queue
5. Выполняем макрозадачу: `2`

## Microtasks vs Macrotasks

### Microtasks (приоритет выше)
- `Promise.then/catch/finally`
- `queueMicrotask()`
- `MutationObserver`
- `process.nextTick()` (Node.js)

### Macrotasks
- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- `requestAnimationFrame`
- I/O операции
- UI rendering

## Сложные примеры

### Пример 1: Вложенные Promise и setTimeout

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout 1');
  Promise.resolve().then(() => console.log('Promise in Timeout 1'));
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    setTimeout(() => console.log('Timeout in Promise 1'), 0);
  })
  .then(() => console.log('Promise 2'));

console.log('End');

// Вывод:
// Start
// End
// Promise 1
// Promise 2
// Timeout 1
// Promise in Timeout 1
// Timeout in Promise 1
```

### Пример 2: Цепочка микрозадач

```javascript
Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    return Promise.resolve();
  })
  .then(() => console.log('Promise 2'))
  .then(() => console.log('Promise 3'));

Promise.resolve()
  .then(() => console.log('Promise 4'))
  .then(() => console.log('Promise 5'));

// Вывод:
// Promise 1
// Promise 4
// Promise 2
// Promise 5
// Promise 3
```

### Пример 3: async/await и Event Loop

```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

async1();

new Promise((resolve) => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
});

console.log('script end');

// Вывод:
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

## Визуализация работы

```
┌─────────────────────────────┐
│      Call Stack             │
│  (выполняется сейчас)       │
└─────────────────────────────┘
           ↓
┌─────────────────────────────┐
│   Microtask Queue           │
│  (Promise, queueMicrotask)  │ ← Приоритет!
└─────────────────────────────┘
           ↓
┌─────────────────────────────┐
│   Macrotask Queue           │
│  (setTimeout, setInterval)  │
└─────────────────────────────┘
```

## Алгоритм Event Loop

1. Выполнить все задачи из Call Stack
2. Выполнить **ВСЕ** микрозадачи из Microtask Queue
3. Если нужно — отрисовать UI
4. Взять **ОДНУ** макрозадачу из Macrotask Queue
5. Вернуться к шагу 1

## Частые ошибки

### Ошибка 1: Бесконечный цикл микрозадач

```javascript
function recursiveMicrotask() {
  Promise.resolve().then(() => {
    console.log('Microtask');
    recursiveMicrotask(); // Блокирует Event Loop!
  });
}

recursiveMicrotask(); // Макрозадачи никогда не выполнятся
```

### Ошибка 2: Неправильное понимание setTimeout(fn, 0)

```javascript
// setTimeout(fn, 0) НЕ выполнится сразу!
// Он добавится в Macrotask Queue

setTimeout(() => console.log('Timeout'), 0);
Promise.resolve().then(() => console.log('Promise'));

// Вывод: Promise, Timeout
```

## Практические советы

1. **Promise всегда быстрее setTimeout** — используйте для срочных задач
2. **Не блокируйте Event Loop** — разбивайте тяжелые вычисления
3. **async/await — это синтаксический сахар** над Promise
4. **Помните про порядок**: синхронный код → микрозадачи → макрозадачи

## Полезные ссылки

- [Loupe — визуализация Event Loop](http://latentflip.com/loupe/)
- [Jake Archibald — In The Loop](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
