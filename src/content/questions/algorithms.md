---
title: "Практические алгоритмы"
description: "Палиндромы, уникальные элементы, flatten массива, частые буквы"
category: "Лайв-кодинг"
difficulty: "easy"
tags: ["algorithms", "strings", "arrays"]
order: 12
---

## Работа со строками

### Проверка на палиндром

```javascript
// Способ 1: Простой
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-zа-я0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}

console.log(isPalindrome('А роза упала на лапу Азора')); // true
console.log(isPalindrome('hello')); // false

// Способ 2: Два указателя (эффективнее)
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-zа-я0-9]/g, '');
  let left = 0;
  let right = cleaned.length - 1;
  
  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false;
    }
    left++;
    right--;
  }
  
  return true;
}
```

### Самая частая буква

```javascript
function mostFrequentChar(str) {
  const cleaned = str.toLowerCase().replace(/[^a-zа-я]/g, '');
  const freq = {};
  
  // Подсчет частоты
  for (const char of cleaned) {
    freq[char] = (freq[char] || 0) + 1;
  }
  
  // Поиск максимума
  let maxChar = '';
  let maxCount = 0;
  
  for (const char in freq) {
    if (freq[char] > maxCount) {
      maxCount = freq[char];
      maxChar = char;
    }
  }
  
  return { char: maxChar, count: maxCount };
}

console.log(mostFrequentChar('hello world'));
// { char: 'l', count: 3 }

// С использованием Map
function mostFrequentChar(str) {
  const cleaned = str.toLowerCase().replace(/[^a-zа-я]/g, '');
  const freq = new Map();
  
  for (const char of cleaned) {
    freq.set(char, (freq.get(char) || 0) + 1);
  }
  
  return [...freq.entries()].reduce((max, [char, count]) => {
    return count > max.count ? { char, count } : max;
  }, { char: '', count: 0 });
}
```

### Анаграммы

```javascript
function areAnagrams(str1, str2) {
  const normalize = str => str.toLowerCase()
    .replace(/[^a-zа-я]/g, '')
    .split('')
    .sort()
    .join('');
  
  return normalize(str1) === normalize(str2);
}

console.log(areAnagrams('listen', 'silent')); // true
console.log(areAnagrams('hello', 'world')); // false

// Более эффективный способ (без сортировки)
function areAnagrams(str1, str2) {
  const count = {};
  
  for (const char of str1.toLowerCase()) {
    if (/[a-zа-я]/.test(char)) {
      count[char] = (count[char] || 0) + 1;
    }
  }
  
  for (const char of str2.toLowerCase()) {
    if (/[a-zа-я]/.test(char)) {
      if (!count[char]) return false;
      count[char]--;
    }
  }
  
  return Object.values(count).every(val => val === 0);
}
```

## Работа с массивами

### Уникальные элементы

```javascript
// Способ 1: Set
function unique(arr) {
  return [...new Set(arr)];
}

console.log(unique([1, 2, 2, 3, 4, 4, 5])); // [1, 2, 3, 4, 5]

// Способ 2: filter
function unique(arr) {
  return arr.filter((item, index) => arr.indexOf(item) === index);
}

// Способ 3: reduce
function unique(arr) {
  return arr.reduce((acc, item) => {
    return acc.includes(item) ? acc : [...acc, item];
  }, []);
}

// Для массива объектов (по ключу)
function uniqueBy(arr, key) {
  const seen = new Set();
  return arr.filter(item => {
    const value = item[key];
    if (seen.has(value)) {
      return false;
    }
    seen.add(value);
    return true;
  });
}

const users = [
  { id: 1, name: 'Анна' },
  { id: 2, name: 'Борис' },
  { id: 1, name: 'Анна' }
];
console.log(uniqueBy(users, 'id'));
// [{ id: 1, name: 'Анна' }, { id: 2, name: 'Борис' }]
```

### Flatten массива

```javascript
// Способ 1: Рекурсия
function flatten(arr) {
  const result = [];
  
  for (const item of arr) {
    if (Array.isArray(item)) {
      result.push(...flatten(item));
    } else {
      result.push(item);
    }
  }
  
  return result;
}

console.log(flatten([1, [2, [3, [4]], 5]])); // [1, 2, 3, 4, 5]

// Способ 2: reduce
function flatten(arr) {
  return arr.reduce((acc, item) => {
    return acc.concat(Array.isArray(item) ? flatten(item) : item);
  }, []);
}

// Способ 3: flat() (встроенный метод)
const arr = [1, [2, [3, [4]], 5]];
console.log(arr.flat(Infinity)); // [1, 2, 3, 4, 5]
console.log(arr.flat(2)); // [1, 2, 3, [4], 5]

// С ограничением глубины
function flatten(arr, depth = 1) {
  if (depth === 0) return arr;
  
  return arr.reduce((acc, item) => {
    return acc.concat(
      Array.isArray(item) ? flatten(item, depth - 1) : item
    );
  }, []);
}
```

### Chunk — разбить массив на части

```javascript
function chunk(arr, size) {
  const result = [];
  
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  
  return result;
}

console.log(chunk([1, 2, 3, 4, 5, 6, 7], 3));
// [[1, 2, 3], [4, 5, 6], [7]]

// С reduce
function chunk(arr, size) {
  return arr.reduce((acc, item, index) => {
    const chunkIndex = Math.floor(index / size);
    
    if (!acc[chunkIndex]) {
      acc[chunkIndex] = [];
    }
    
    acc[chunkIndex].push(item);
    return acc;
  }, []);
}
```

### Пересечение массивов

```javascript
function intersection(arr1, arr2) {
  const set2 = new Set(arr2);
  return [...new Set(arr1.filter(item => set2.has(item)))];
}

console.log(intersection([1, 2, 3, 4], [3, 4, 5, 6])); // [3, 4]

// Для нескольких массивов
function intersection(...arrays) {
  if (arrays.length === 0) return [];
  
  return arrays.reduce((acc, arr) => {
    const set = new Set(arr);
    return acc.filter(item => set.has(item));
  });
}

console.log(intersection([1, 2, 3], [2, 3, 4], [3, 4, 5])); // [3]
```

### Разница массивов

```javascript
function difference(arr1, arr2) {
  const set2 = new Set(arr2);
  return arr1.filter(item => !set2.has(item));
}

console.log(difference([1, 2, 3, 4], [3, 4, 5, 6])); // [1, 2]
```

## Работа с числами

### Числа Фибоначчи

```javascript
// Рекурсия (медленно)
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Итеративно (быстро)
function fibonacci(n) {
  if (n <= 1) return n;
  
  let prev = 0;
  let curr = 1;
  
  for (let i = 2; i <= n; i++) {
    const next = prev + curr;
    prev = curr;
    curr = next;
  }
  
  return curr;
}

// Генератор последовательности
function* fibonacciSequence() {
  let prev = 0;
  let curr = 1;
  
  yield prev;
  yield curr;
  
  while (true) {
    const next = prev + curr;
    yield next;
    prev = curr;
    curr = next;
  }
}

const fib = fibonacciSequence();
console.log(fib.next().value); // 0
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
```

### Простое число

```javascript
function isPrime(n) {
  if (n <= 1) return false;
  if (n <= 3) return true;
  
  if (n % 2 === 0 || n % 3 === 0) return false;
  
  for (let i = 5; i * i <= n; i += 6) {
    if (n % i === 0 || n % (i + 2) === 0) {
      return false;
    }
  }
  
  return true;
}

console.log(isPrime(17)); // true
console.log(isPrime(18)); // false
```

### Факториал

```javascript
// Рекурсия
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// Итеративно
function factorial(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

console.log(factorial(5)); // 120
```

## Сортировка и поиск

### Бинарный поиск

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
      return mid;
    }
    
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return -1; // Не найдено
}

const arr = [1, 3, 5, 7, 9, 11, 13];
console.log(binarySearch(arr, 7)); // 3
console.log(binarySearch(arr, 6)); // -1
```

### Быстрая сортировка (QuickSort)

```javascript
function quickSort(arr) {
  if (arr.length <= 1) return arr;
  
  const pivot = arr[Math.floor(arr.length / 2)];
  const left = arr.filter(x => x < pivot);
  const middle = arr.filter(x => x === pivot);
  const right = arr.filter(x => x > pivot);
  
  return [...quickSort(left), ...middle, ...quickSort(right)];
}

console.log(quickSort([3, 6, 8, 10, 1, 2, 1]));
// [1, 1, 2, 3, 6, 8, 10]
```

## Практические задачи

### Группировка по свойству

```javascript
function groupBy(arr, key) {
  return arr.reduce((acc, item) => {
    const group = item[key];
    if (!acc[group]) {
      acc[group] = [];
    }
    acc[group].push(item);
    return acc;
  }, {});
}

const users = [
  { name: 'Анна', city: 'Москва' },
  { name: 'Борис', city: 'Санкт-Петербург' },
  { name: 'Виктор', city: 'Москва' }
];

console.log(groupBy(users, 'city'));
// {
//   'Москва': [{ name: 'Анна', ... }, { name: 'Виктор', ... }],
//   'Санкт-Петербург': [{ name: 'Борис', ... }]
// }
```

### Подсчет элементов

```javascript
function countBy(arr, fn) {
  return arr.reduce((acc, item) => {
    const key = fn(item);
    acc[key] = (acc[key] || 0) + 1;
    return acc;
  }, {});
}

const numbers = [1, 2, 3, 4, 5, 6];
console.log(countBy(numbers, n => n % 2 === 0 ? 'even' : 'odd'));
// { odd: 3, even: 3 }
```

### Глубокое клонирование

```javascript
function deepClone(obj) {
  // Примитивы и null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Массив
  if (Array.isArray(obj)) {
    return obj.map(item => deepClone(item));
  }
  
  // Объект
  const cloned = {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  
  return cloned;
}

const original = {
  name: 'Анна',
  address: {
    city: 'Москва',
    street: 'Ленина'
  },
  hobbies: ['чтение', 'спорт']
};

const copy = deepClone(original);
copy.address.city = 'Санкт-Петербург';
console.log(original.address.city); // 'Москва' (не изменился)

// Или используйте встроенный метод
const copy = structuredClone(original);
```

### Debounce (повторение)

```javascript
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}
```

### Throttle (повторение)

```javascript
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
```

## Советы для собеседования

1. **Начните с простого решения**, потом оптимизируйте
2. **Обсудите сложность** — O(n), O(n²), O(log n)
3. **Тестируйте edge cases**: пустые массивы, null, undefined
4. **Называйте переменные осмысленно**
5. **Используйте встроенные методы**, если они подходят
6. **Объясняйте свои мысли** вслух

## Сложность алгоритмов

| Операция | Сложность |
|----------|-----------|
| Поиск в массиве | O(n) |
| Бинарный поиск | O(log n) |
| Сортировка (QuickSort) | O(n log n) |
| Flatten (рекурсия) | O(n) |
| Unique (Set) | O(n) |
| Intersection | O(n + m) |
