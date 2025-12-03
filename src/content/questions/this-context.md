---
title: "Контекст this в JavaScript"
description: "Как определяется this в разных ситуациях: функции, методы, стрелочные функции, bind/call/apply"
category: "JavaScript"
difficulty: "medium"
tags: ["this", "context", "bind", "call", "apply"]
order: 5
---

## Что такое `this`?

`this` — это ключевое слово, которое ссылается на контекст выполнения функции. Его значение зависит от того, **как** была вызвана функция.

## Правила определения `this`

### 1. Глобальный контекст

```javascript
console.log(this); // Window (в браузере) или global (в Node.js)

function regularFunction() {
  console.log(this); // Window в нестрогом режиме, undefined в строгом
}

'use strict';
function strictFunction() {
  console.log(this); // undefined
}
```

### 2. Метод объекта

```javascript
const person = {
  name: 'Анна',
  greet: function() {
    console.log(this.name); // 'Анна'
  }
};

person.greet(); // this = person
```

**Важно!** Если метод присвоить переменной:

```javascript
const greetFunc = person.greet;
greetFunc(); // this = undefined (strict mode) или Window
```

### 3. Конструктор (new)

```javascript
function Person(name) {
  this.name = name;
  this.greet = function() {
    console.log(`Привет, я ${this.name}`);
  };
}

const anna = new Person('Анна');
anna.greet(); // this = anna
```

### 4. Стрелочные функции

Стрелочные функции **НЕ имеют своего `this`**. Они берут `this` из внешнего лексического окружения.

```javascript
const person = {
  name: 'Анна',
  greet: function() {
    const arrow = () => {
      console.log(this.name); // 'Анна' — берет this из greet
    };
    arrow();
  }
};

person.greet();
```

**Проблема с обычными функциями:**

```javascript
const person = {
  name: 'Анна',
  friends: ['Борис', 'Виктор'],
  showFriends: function() {
    this.friends.forEach(function(friend) {
      console.log(this.name + ' знает ' + friend);
      // Error! this = undefined
    });
  }
};

// Решение 1: стрелочная функция
showFriends: function() {
  this.friends.forEach(friend => {
    console.log(this.name + ' знает ' + friend); // Работает!
  });
}

// Решение 2: bind
showFriends: function() {
  this.friends.forEach(function(friend) {
    console.log(this.name + ' знает ' + friend);
  }.bind(this));
}
```

### 5. call, apply, bind

Эти методы позволяют явно задать `this`.

#### `call()`

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: 'Анна' };

greet.call(person, 'Привет', '!'); // Привет, Анна!
```

#### `apply()`

Как `call`, но аргументы передаются массивом:

```javascript
greet.apply(person, ['Привет', '!']); // Привет, Анна!
```

#### `bind()`

Создает новую функцию с привязанным `this`:

```javascript
const boundGreet = greet.bind(person);
boundGreet('Привет', '!'); // Привет, Анна!

// Можно частично применить аргументы
const boundHello = greet.bind(person, 'Привет');
boundHello('!'); // Привет, Анна!
```

## Приоритет правил

1. **`new`** — `this` = новый объект
2. **`call/apply/bind`** — `this` = явно указанный объект
3. **Метод объекта** — `this` = объект, на котором вызван метод
4. **Глобальный контекст** — `this` = `window`/`global` или `undefined`

**Стрелочные функции игнорируют все эти правила** и берут `this` из внешнего scope.

## Частые ошибки

### Ошибка 1: Потеря контекста

```javascript
const button = {
  text: 'Нажми меня',
  click: function() {
    console.log(this.text);
  }
};

document.querySelector('#btn').addEventListener('click', button.click);
// Error! this = button element, а не объект button

// Решение 1: стрелочная функция
addEventListener('click', () => button.click());

// Решение 2: bind
addEventListener('click', button.click.bind(button));
```

### Ошибка 2: this в setTimeout

```javascript
const person = {
  name: 'Анна',
  greet: function() {
    setTimeout(function() {
      console.log(this.name); // undefined
    }, 1000);
  }
};

// Решение: стрелочная функция
greet: function() {
  setTimeout(() => {
    console.log(this.name); // 'Анна'
  }, 1000);
}
```

### Ошибка 3: this в классах

```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    this.count++;
  }
}

const counter = new Counter();
const incrementFunc = counter.increment;
incrementFunc(); // Error! this = undefined

// Решение 1: bind в конструкторе
constructor() {
  this.count = 0;
  this.increment = this.increment.bind(this);
}

// Решение 2: стрелочная функция (class field)
increment = () => {
  this.count++;
}
```

## Практические примеры

### Пример 1: Цепочка вызовов (chaining)

```javascript
const calculator = {
  value: 0,
  add(n) {
    this.value += n;
    return this; // Возвращаем this для цепочки
  },
  multiply(n) {
    this.value *= n;
    return this;
  },
  result() {
    return this.value;
  }
};

calculator.add(5).multiply(2).add(3).result(); // 13
```

### Пример 2: Заимствование методов

```javascript
const arrayLike = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3
};

// Заимствуем метод slice у массива
const arr = Array.prototype.slice.call(arrayLike);
console.log(arr); // ['a', 'b', 'c']

// Или с помощью spread
const arr2 = [...arrayLike]; // Error! arrayLike не итерируемый
const arr3 = Array.from(arrayLike); // ['a', 'b', 'c'] ✓
```

## Шпаргалка

| Вызов | this |
|-------|------|
| `func()` | `undefined` (strict) / `window` |
| `obj.method()` | `obj` |
| `new Func()` | новый объект |
| `func.call(obj)` | `obj` |
| `func.apply(obj)` | `obj` |
| `func.bind(obj)()` | `obj` |
| `() => {}` | `this` из внешнего scope |

## Советы для собеседования

1. **Всегда спрашивайте**: "Как вызвана функция?"
2. **Помните**: стрелочные функции не имеют своего `this`
3. **Используйте `bind`** для сохранения контекста в коллбэках
4. **В классах** предпочитайте стрелочные функции для методов-обработчиков
