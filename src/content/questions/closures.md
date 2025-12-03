---
title: "Что такое замыкания (closures)?"
description: "Объяснение концепции замыканий в JavaScript с примерами"
category: "JavaScript"
difficulty: "medium"
tags: ["closures", "scope", "functions"]
order: 1
---

## Определение

**Замыкание (closure)** — это функция, которая имеет доступ к переменным из внешней (объемлющей) функции даже после того, как внешняя функция завершила выполнение.

## Как это работает

Когда функция создается, она сохраняет ссылку на свою лексическую область видимости. Это означает, что функция "запоминает" окружение, в котором она была создана.

```javascript
function createCounter() {
  let count = 0;
  
  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

## Практическое применение

### 1. Приватные переменные

```javascript
function createPerson(name) {
  let age = 0;
  
  return {
    getName: () => name,
    getAge: () => age,
    setAge: (newAge) => {
      if (newAge > 0) {
        age = newAge;
      }
    }
  };
}

const person = createPerson('Иван');
console.log(person.getName()); // 'Иван'
person.setAge(25);
console.log(person.getAge()); // 25
```

### 2. Функции-фабрики

```javascript
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### 3. Обработчики событий

```javascript
function setupButton(buttonId) {
  let clickCount = 0;
  
  document.getElementById(buttonId).addEventListener('click', function() {
    clickCount++;
    console.log(`Кнопка нажата ${clickCount} раз`);
  });
}
```

## Важные моменты

1. **Память**: Замыкания сохраняют ссылки на переменные, что может привести к утечкам памяти, если не управлять ими правильно.

2. **Производительность**: Создание множества замыканий может повлиять на производительность.

3. **Циклы и замыкания**: Классическая проблема с `var` в циклах:

```javascript
// Проблема
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // Выведет 3, 3, 3
  }, 100);
}

// Решение с let
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // Выведет 0, 1, 2
  }, 100);
}

// Решение с замыканием
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j); // Выведет 0, 1, 2
    }, 100);
  })(i);
}
```

## Заключение

Замыкания — это мощный инструмент JavaScript, который позволяет создавать приватные переменные, функции-фабрики и эффективно работать с асинхронным кодом. Понимание замыканий критически важно для работы с JavaScript.
