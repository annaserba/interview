---
title: "Оценка сложности алгоритмов"
description: "Big O нотация, временная и пространственная сложность"
category: "Лайв-кодинг"
difficulty: "medium"
tags: ["complexity", "big-o", "algorithms", "performance", "time-complexity"]
order: 36
---

## Что такое сложность алгоритма?

Сложность алгоритма — это оценка количества ресурсов (времени и памяти), необходимых для выполнения алгоритма в зависимости от размера входных данных.

### Зачем это нужно?

- 📊 Сравнение алгоритмов
- 🚀 Предсказание производительности
- 💡 Выбор оптимального решения
- 🎯 Понимание масштабируемости

## Big O нотация

Big O описывает **верхнюю границу** роста функции при увеличении входных данных.

### Основные классы сложности

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)

От лучшего к худшему →
```

### Визуализация роста

```
n = 10:
O(1)      = 1
O(log n)  = 3
O(n)      = 10
O(n log n)= 30
O(n²)     = 100
O(2ⁿ)     = 1,024
O(n!)     = 3,628,800

n = 100:
O(1)      = 1
O(log n)  = 7
O(n)      = 100
O(n log n)= 700
O(n²)     = 10,000
O(2ⁿ)     = 1.27 × 10³⁰
O(n!)     = 9.33 × 10¹⁵⁷
```

## Временная сложность

### O(1) - Константная

Время выполнения не зависит от размера входных данных.

```javascript
// Доступ к элементу массива
function getFirst(arr) {
  return arr[0]  // O(1)
}

// Доступ к свойству объекта
function getName(user) {
  return user.name  // O(1)
}

// Арифметические операции
function add(a, b) {
  return a + b  // O(1)
}

// HashMap операции
const map = new Map()
map.set('key', 'value')  // O(1)
map.get('key')           // O(1)
map.has('key')           // O(1)
map.delete('key')        // O(1)
```

**Примеры:**
- Доступ к элементу по индексу
- Вставка/удаление в начало/конец связного списка
- Push/pop в стек
- HashMap операции

### O(log n) - Логарифмическая

Время растёт логарифмически. Обычно делим задачу пополам на каждом шаге.

```javascript
// Бинарный поиск
function binarySearch(arr, target) {
  let left = 0
  let right = arr.length - 1
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2)
    
    if (arr[mid] === target) {
      return mid
    } else if (arr[mid] < target) {
      left = mid + 1
    } else {
      right = mid - 1
    }
  }
  
  return -1
}
// O(log n) - каждая итерация делит массив пополам

// Поиск в сбалансированном BST
function searchBST(root, target) {
  if (!root) return null
  
  if (root.val === target) return root
  
  if (target < root.val) {
    return searchBST(root.left, target)
  } else {
    return searchBST(root.right, target)
  }
}
// O(log n) для сбалансированного дерева
```

**Примеры:**
- Бинарный поиск
- Операции в сбалансированном BST
- Поиск в skip list
- Некоторые алгоритмы "разделяй и властвуй"

### O(n) - Линейная

Время растёт пропорционально размеру входных данных.

```javascript
// Поиск элемента в массиве
function findElement(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i
    }
  }
  return -1
}
// O(n) - в худшем случае проверяем все элементы

// Сумма элементов
function sum(arr) {
  let total = 0
  for (const num of arr) {
    total += num
  }
  return total
}
// O(n) - один проход по массиву

// Поиск максимума
function findMax(arr) {
  let max = arr[0]
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
      max = arr[i]
    }
  }
  return max
}
// O(n)
```

**Примеры:**
- Линейный поиск
- Обход массива/списка
- Подсчёт элементов
- Поиск min/max

### O(n log n) - Линейно-логарифмическая

Эффективные алгоритмы сортировки.

```javascript
// Merge Sort
function mergeSort(arr) {
  if (arr.length <= 1) return arr
  
  const mid = Math.floor(arr.length / 2)
  const left = mergeSort(arr.slice(0, mid))
  const right = mergeSort(arr.slice(mid))
  
  return merge(left, right)
}

function merge(left, right) {
  const result = []
  let i = 0, j = 0
  
  while (i < left.length && j < right.length) {
    if (left[i] < right[j]) {
      result.push(left[i++])
    } else {
      result.push(right[j++])
    }
  }
  
  return result.concat(left.slice(i)).concat(right.slice(j))
}
// O(n log n) - log n уровней рекурсии, n работы на каждом

// Quick Sort (средний случай)
function quickSort(arr) {
  if (arr.length <= 1) return arr
  
  const pivot = arr[arr.length - 1]
  const left = []
  const right = []
  
  for (let i = 0; i < arr.length - 1; i++) {
    if (arr[i] < pivot) {
      left.push(arr[i])
    } else {
      right.push(arr[i])
    }
  }
  
  return [...quickSort(left), pivot, ...quickSort(right)]
}
// O(n log n) в среднем, O(n²) в худшем

// Heap Sort
// O(n log n)
```

**Примеры:**
- Merge Sort
- Quick Sort (средний случай)
- Heap Sort
- Некоторые алгоритмы на деревьях

### O(n²) - Квадратичная

Вложенные циклы по одним и тем же данным.

```javascript
// Bubble Sort
function bubbleSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
      }
    }
  }
  return arr
}
// O(n²) - два вложенных цикла

// Поиск дубликатов (наивный подход)
function hasDuplicates(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) {
        return true
      }
    }
  }
  return false
}
// O(n²)

// Можно оптимизировать до O(n) с Set:
function hasDuplicatesOptimized(arr) {
  const seen = new Set()
  for (const item of arr) {
    if (seen.has(item)) return true
    seen.add(item)
  }
  return false
}
// O(n)
```

**Примеры:**
- Bubble Sort, Selection Sort, Insertion Sort
- Вложенные циклы
- Наивные алгоритмы сравнения

### O(2ⁿ) - Экспоненциальная

Очень медленно растёт. Обычно в рекурсивных решениях без мемоизации.

```javascript
// Fibonacci (наивная рекурсия)
function fibonacci(n) {
  if (n <= 1) return n
  return fibonacci(n - 1) + fibonacci(n - 2)
}
// O(2ⁿ) - каждый вызов порождает 2 новых

// Оптимизация с мемоизацией:
function fibonacciMemo(n, memo = {}) {
  if (n <= 1) return n
  if (memo[n]) return memo[n]
  
  memo[n] = fibonacciMemo(n - 1, memo) + fibonacciMemo(n - 2, memo)
  return memo[n]
}
// O(n) с мемоизацией

// Все подмножества множества
function subsets(nums) {
  const result = [[]]
  
  for (const num of nums) {
    const len = result.length
    for (let i = 0; i < len; i++) {
      result.push([...result[i], num])
    }
  }
  
  return result
}
// O(2ⁿ) - 2ⁿ подмножеств
```

**Примеры:**
- Наивный Fibonacci
- Генерация всех подмножеств
- Некоторые backtracking задачи

### O(n!) - Факториальная

Самая медленная. Перебор всех перестановок.

```javascript
// Все перестановки
function permutations(arr) {
  if (arr.length <= 1) return [arr]
  
  const result = []
  
  for (let i = 0; i < arr.length; i++) {
    const current = arr[i]
    const remaining = [...arr.slice(0, i), ...arr.slice(i + 1)]
    const perms = permutations(remaining)
    
    for (const perm of perms) {
      result.push([current, ...perm])
    }
  }
  
  return result
}
// O(n!) - n! перестановок

// Travelling Salesman Problem (brute force)
// O(n!)
```

**Примеры:**
- Генерация всех перестановок
- Travelling Salesman (brute force)
- Некоторые NP-полные задачи

## Пространственная сложность

Количество дополнительной памяти, используемой алгоритмом.

### O(1) - Константная память

```javascript
// Swap элементов
function swap(arr, i, j) {
  const temp = arr[i]  // O(1) память
  arr[i] = arr[j]
  arr[j] = temp
}

// Поиск максимума
function findMax(arr) {
  let max = arr[0]  // O(1) память
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
      max = arr[i]
    }
  }
  return max
}
```

### O(n) - Линейная память

```javascript
// Копирование массива
function copyArray(arr) {
  const copy = []
  for (const item of arr) {
    copy.push(item)
  }
  return copy
}
// O(n) память

// Рекурсивный Fibonacci (стек вызовов)
function fibonacci(n) {
  if (n <= 1) return n
  return fibonacci(n - 1) + fibonacci(n - 2)
}
// O(n) память из-за стека рекурсии

// HashMap для подсчёта
function countFrequency(arr) {
  const freq = new Map()  // O(n) память
  for (const item of arr) {
    freq.set(item, (freq.get(item) || 0) + 1)
  }
  return freq
}
```

### O(log n) - Логарифмическая память

```javascript
// Бинарный поиск (рекурсивный)
function binarySearch(arr, target, left = 0, right = arr.length - 1) {
  if (left > right) return -1
  
  const mid = Math.floor((left + right) / 2)
  
  if (arr[mid] === target) return mid
  
  if (arr[mid] < target) {
    return binarySearch(arr, target, mid + 1, right)
  } else {
    return binarySearch(arr, target, left, mid - 1)
  }
}
// O(log n) память из-за стека рекурсии
```

## Правила анализа сложности

### 1. Отбрасываем константы

```javascript
// O(2n) = O(n)
function example1(arr) {
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i])
  }
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i])
  }
}
// O(n + n) = O(2n) = O(n)

// O(n/2) = O(n)
function example2(arr) {
  for (let i = 0; i < arr.length; i += 2) {
    console.log(arr[i])
  }
}
// O(n/2) = O(n)
```

### 2. Берём доминирующий член

```javascript
// O(n² + n) = O(n²)
function example3(arr) {
  // O(n²)
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length; j++) {
      console.log(arr[i], arr[j])
    }
  }
  
  // O(n)
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i])
  }
}
// O(n² + n) = O(n²)

// O(n log n + n) = O(n log n)
function example4(arr) {
  arr.sort()  // O(n log n)
  
  for (const item of arr) {  // O(n)
    console.log(item)
  }
}
// O(n log n)
```

### 3. Разные переменные - разные обозначения

```javascript
// O(a + b), НЕ O(n)
function example5(arr1, arr2) {
  for (const item of arr1) {
    console.log(item)
  }
  for (const item of arr2) {
    console.log(item)
  }
}
// O(a + b)

// O(a * b), НЕ O(n²)
function example6(arr1, arr2) {
  for (const item1 of arr1) {
    for (const item2 of arr2) {
      console.log(item1, item2)
    }
  }
}
// O(a * b)
```

### 4. Учитываем худший случай

```javascript
// Худший случай: O(n)
function findElement(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i  // Может вернуться сразу
    }
  }
  return -1  // Или пройти весь массив
}
// Best case: O(1)
// Average case: O(n/2) = O(n)
// Worst case: O(n)
// Big O описывает worst case
```

## Амортизированная сложность

Средняя сложность операции при многократном выполнении.

```javascript
// Dynamic Array (как в JavaScript)
class DynamicArray {
  constructor() {
    this.data = new Array(1)
    this.size = 0
    this.capacity = 1
  }
  
  push(item) {
    if (this.size === this.capacity) {
      // Удвоить размер массива
      this.capacity *= 2
      const newData = new Array(this.capacity)
      for (let i = 0; i < this.size; i++) {
        newData[i] = this.data[i]
      }
      this.data = newData
    }
    
    this.data[this.size] = item
    this.size++
  }
}

// Отдельная операция push может быть O(n) (при resize)
// Но амортизированная сложность O(1)
// Потому что resize происходит редко
```

## Практические примеры

### Оптимизация Two Sum

```javascript
// ❌ Наивное решение O(n²)
function twoSumSlow(nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j]
      }
    }
  }
  return []
}

// ✅ Оптимизированное O(n)
function twoSumFast(nums, target) {
  const map = new Map()
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i]
    
    if (map.has(complement)) {
      return [map.get(complement), i]
    }
    
    map.set(nums[i], i)
  }
  
  return []
}
// Время: O(n²) → O(n)
// Память: O(1) → O(n)
// Trade-off: память за скорость
```

### Оптимизация поиска дубликатов

```javascript
// ❌ O(n²)
function hasDuplicatesSlow(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) return true
    }
  }
  return false
}

// ✅ O(n)
function hasDuplicatesFast(arr) {
  const seen = new Set()
  for (const item of arr) {
    if (seen.has(item)) return true
    seen.add(item)
  }
  return false
}
```

### Fibonacci оптимизация

```javascript
// ❌ O(2ⁿ) время, O(n) память
function fibSlow(n) {
  if (n <= 1) return n
  return fibSlow(n - 1) + fibSlow(n - 2)
}

// ✅ O(n) время, O(n) память (мемоизация)
function fibMemo(n, memo = {}) {
  if (n <= 1) return n
  if (memo[n]) return memo[n]
  
  memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo)
  return memo[n]
}

// ✅ O(n) время, O(1) память (итеративно)
function fibIterative(n) {
  if (n <= 1) return n
  
  let prev = 0
  let curr = 1
  
  for (let i = 2; i <= n; i++) {
    const next = prev + curr
    prev = curr
    curr = next
  }
  
  return curr
}
```

## Вопросы для собеседования

### 1. Что такое Big O нотация?

**Ответ:**
Big O описывает верхнюю границу роста времени/памяти алгоритма при увеличении входных данных. Показывает, как алгоритм масштабируется.

### 2. В чём разница между O(n) и O(2n)?

**Ответ:**
Нет разницы. В Big O отбрасываем константы: O(2n) = O(n). Важен порядок роста, а не точное количество операций.

### 3. Что лучше: O(n²) или O(n log n)?

**Ответ:**
O(n log n) лучше. При больших n разница огромная:
- n=1000: n²=1,000,000 vs n log n≈10,000
- O(n log n) используется в эффективных сортировках

### 4. Почему HashMap операции O(1)?

**Ответ:**
Хеш-функция вычисляет индекс за O(1), доступ по индексу тоже O(1). В худшем случае (коллизии) может быть O(n), но амортизированно O(1).

### 5. Как оптимизировать O(n²) до O(n)?

**Ответ:**
Часто помогает HashMap/Set:
- Вместо вложенных циклов поиска → HashMap
- Вместо проверки каждого с каждым → Set для быстрой проверки
- Trade-off: O(n) память за O(n) время

### 6. Что такое амортизированная сложность?

**Ответ:**
Средняя сложность операции при многократном выполнении. Пример: push в динамический массив — отдельная операция может быть O(n) (resize), но амортизированно O(1).

### 7. Какая сложность у сортировки?

**Ответ:**
- **Comparison sorts**: O(n log n) лучший случай (Merge, Quick, Heap)
- **Bubble/Selection/Insertion**: O(n²)
- **Non-comparison**: O(n) возможно (Counting, Radix, Bucket)

### 8. Рекурсия vs итерация: сложность?

**Ответ:**
**Время:** обычно одинаковое
**Память:** рекурсия использует стек O(глубина рекурсии), итерация O(1)

Рекурсия может быть проще для понимания, но рискует stack overflow.

### 9. Как анализировать сложность рекурсии?

**Ответ:**
1. Найти рекуррентное соотношение
2. Использовать Master Theorem или дерево рекурсии
3. Учесть глубину и работу на каждом уровне

Пример: Merge Sort
- T(n) = 2T(n/2) + O(n)
- log n уровней, O(n) работы на каждом
- Итого: O(n log n)

### 10. Что важнее: время или память?

**Ответ:**
Зависит от контекста:
- **Время критично**: real-time системы, UI
- **Память критична**: embedded системы, мобильные устройства
- Обычно: время важнее (память дешёвая)

Часто есть trade-off: кэширование (больше памяти, меньше времени).
