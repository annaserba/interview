---
title: "Функции высшего порядка"
description: "Реализация map, filter, reduce, debounce и throttle"
category: "Лайв-кодинг"
difficulty: "medium"
tags: ["functions", "higher-order", "debounce", "throttle"]
order: 3
---

## Что такое функции высшего порядка?

Функции, которые принимают другие функции в качестве аргументов или возвращают функции.

## Реализация базовых методов

### Собственный `map`

```javascript
Array.prototype.myMap = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this));
  }
  return result;
};

// Использование
const numbers = [1, 2, 3];
const doubled = numbers.myMap(x => x * 2);
console.log(doubled); // [2, 4, 6]
```

### Собственный `filter`

```javascript
Array.prototype.myFilter = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (callback(this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

// Использование
const numbers = [1, 2, 3, 4, 5];
const even = numbers.myFilter(x => x % 2 === 0);
console.log(even); // [2, 4]
```

### Собственный `reduce`

```javascript
Array.prototype.myReduce = function(callback, initialValue) {
  let accumulator = initialValue !== undefined ? initialValue : this[0];
  let startIndex = initialValue !== undefined ? 0 : 1;
  
  for (let i = startIndex; i < this.length; i++) {
    accumulator = callback(accumulator, this[i], i, this);
  }
  
  return accumulator;
};

// Использование
const numbers = [1, 2, 3, 4];
const sum = numbers.myReduce((acc, curr) => acc + curr, 0);
console.log(sum); // 10
```

## Debounce — ОБЯЗАТЕЛЬНО!

Откладывает выполнение функции до тех пор, пока не пройдет определенное время после последнего вызова.

```javascript
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    // Очищаем предыдущий таймер
    clearTimeout(timeoutId);
    
    // Устанавливаем новый таймер
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// Использование
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((e) => {
  console.log('Поиск:', e.target.value);
  // API запрос
}, 300);

searchInput.addEventListener('input', debouncedSearch);
```

### Debounce с немедленным выполнением

```javascript
function debounce(func, delay, immediate = false) {
  let timeoutId;
  
  return function(...args) {
    const callNow = immediate && !timeoutId;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!immediate) {
        func.apply(this, args);
      }
    }, delay);
    
    if (callNow) {
      func.apply(this, args);
    }
  };
}
```

## Throttle — ОБЯЗАТЕЛЬНО!

Ограничивает частоту выполнения функции — не чаще одного раза в указанный период.

```javascript
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Использование
const handleScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 1000);

window.addEventListener('scroll', handleScroll);
```

### Throttle с trailing call

```javascript
function throttle(func, limit) {
  let inThrottle;
  let lastFunc;
  let lastRan;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      lastRan = Date.now();
      inThrottle = true;
    } else {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(() => {
        if ((Date.now() - lastRan) >= limit) {
          func.apply(this, args);
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  };
}
```

## Разница между Debounce и Throttle

| Debounce | Throttle |
|----------|----------|
| Выполняет функцию после паузы | Выполняет функцию с интервалом |
| Для поиска, валидации ввода | Для скролла, ресайза окна |
| Ждет окончания активности | Ограничивает частоту вызовов |

## Мемоизация

```javascript
function memoize(fn) {
  const cache = {};
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      console.log('Из кэша:', key);
      return cache[key];
    }
    
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

// Использование
const fibonacci = memoize(function(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // Вычисляет
console.log(fibonacci(10)); // Из кэша
```

## Практические советы

1. **Всегда используйте `apply` или `call`** для сохранения контекста `this`
2. **Используйте `...args`** для работы с любым количеством аргументов
3. **Помните про замыкания** — они сохраняют ссылки на переменные
4. **Тестируйте edge cases**: пустые массивы, отсутствие `initialValue` в `reduce`
