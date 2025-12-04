---
title: "Структуры данных"
description: "Основные структуры данных и их применение"
category: "Лайв-кодинг"
difficulty: "medium"
tags: ["data-structures", "algorithms", "array", "linked-list", "tree", "graph", "hash-table"]
order: 37
---

## Основные структуры данных

Структура данных — способ организации и хранения данных для эффективного доступа и модификации.

### Критерии выбора структуры данных

- ⏱️ **Временная сложность** операций
- 💾 **Пространственная сложность**
- 🎯 **Тип операций** (поиск, вставка, удаление)
- 📊 **Характер данных** (упорядоченность, уникальность)

## Array (Массив)

### Описание

Последовательная коллекция элементов фиксированного или динамического размера.

```javascript
// Статический массив (в других языках)
const arr = new Array(5)  // [undefined, undefined, undefined, undefined, undefined]

// Динамический массив (JavaScript)
const dynamicArr = [1, 2, 3]
dynamicArr.push(4)  // [1, 2, 3, 4]
```

### Сложность операций

| Операция | Сложность |
|----------|-----------|
| Доступ по индексу | O(1) |
| Поиск элемента | O(n) |
| Вставка в конец | O(1) амортизированно |
| Вставка в начало | O(n) |
| Удаление из конца | O(1) |
| Удаление из начала | O(n) |

### Основные методы

```javascript
const arr = [1, 2, 3, 4, 5]

// Доступ
arr[0]  // 1 - O(1)
arr.length  // 5 - O(1)

// Добавление
arr.push(6)  // [1, 2, 3, 4, 5, 6] - O(1)
arr.unshift(0)  // [0, 1, 2, 3, 4, 5, 6] - O(n)

// Удаление
arr.pop()  // [0, 1, 2, 3, 4, 5] - O(1)
arr.shift()  // [1, 2, 3, 4, 5] - O(n)

// Поиск
arr.indexOf(3)  // 2 - O(n)
arr.includes(3)  // true - O(n)

// Вставка/удаление в середине
arr.splice(2, 1)  // [1, 2, 4, 5] - O(n)
arr.splice(2, 0, 3)  // [1, 2, 3, 4, 5] - O(n)

// Итерация
arr.forEach(x => console.log(x))  // O(n)
arr.map(x => x * 2)  // O(n)
arr.filter(x => x > 2)  // O(n)
arr.reduce((sum, x) => sum + x, 0)  // O(n)
```

### Когда использовать

✅ **Использовать:**
- Нужен быстрый доступ по индексу
- Известен размер заранее
- Частые операции чтения
- Итерация по всем элементам

❌ **Не использовать:**
- Частые вставки/удаления в начале
- Неизвестный размер с частыми изменениями
- Нужна быстрая вставка в середину

## Linked List (Связный список)

### Описание

Последовательность узлов, где каждый узел содержит данные и ссылку на следующий узел.

```javascript
class Node {
  constructor(value) {
    this.value = value
    this.next = null
  }
}

class LinkedList {
  constructor() {
    this.head = null
    this.tail = null
    this.size = 0
  }
  
  // Добавить в конец - O(1)
  append(value) {
    const newNode = new Node(value)
    
    if (!this.head) {
      this.head = newNode
      this.tail = newNode
    } else {
      this.tail.next = newNode
      this.tail = newNode
    }
    
    this.size++
  }
  
  // Добавить в начало - O(1)
  prepend(value) {
    const newNode = new Node(value)
    newNode.next = this.head
    this.head = newNode
    
    if (!this.tail) {
      this.tail = newNode
    }
    
    this.size++
  }
  
  // Найти элемент - O(n)
  find(value) {
    let current = this.head
    
    while (current) {
      if (current.value === value) {
        return current
      }
      current = current.next
    }
    
    return null
  }
  
  // Удалить элемент - O(n)
  delete(value) {
    if (!this.head) return
    
    // Удаление головы
    if (this.head.value === value) {
      this.head = this.head.next
      this.size--
      return
    }
    
    let current = this.head
    
    while (current.next) {
      if (current.next.value === value) {
        current.next = current.next.next
        
        // Обновить tail если удалили последний
        if (!current.next) {
          this.tail = current
        }
        
        this.size--
        return
      }
      current = current.next
    }
  }
  
  // Преобразовать в массив - O(n)
  toArray() {
    const result = []
    let current = this.head
    
    while (current) {
      result.push(current.value)
      current = current.next
    }
    
    return result
  }
}

// Использование
const list = new LinkedList()
list.append(1)
list.append(2)
list.append(3)
list.prepend(0)
console.log(list.toArray())  // [0, 1, 2, 3]
```

### Doubly Linked List (Двусвязный список)

```javascript
class DoublyNode {
  constructor(value) {
    this.value = value
    this.next = null
    this.prev = null
  }
}

class DoublyLinkedList {
  constructor() {
    this.head = null
    this.tail = null
    this.size = 0
  }
  
  append(value) {
    const newNode = new DoublyNode(value)
    
    if (!this.head) {
      this.head = newNode
      this.tail = newNode
    } else {
      newNode.prev = this.tail
      this.tail.next = newNode
      this.tail = newNode
    }
    
    this.size++
  }
  
  // Можем идти в обе стороны
  traverseBackward() {
    const result = []
    let current = this.tail
    
    while (current) {
      result.push(current.value)
      current = current.prev
    }
    
    return result
  }
}
```

### Сложность операций

| Операция | Singly | Doubly |
|----------|--------|--------|
| Доступ по индексу | O(n) | O(n) |
| Поиск элемента | O(n) | O(n) |
| Вставка в начало | O(1) | O(1) |
| Вставка в конец | O(1) | O(1) |
| Удаление из начала | O(1) | O(1) |
| Удаление из конца | O(n) | O(1) |

### Когда использовать

✅ **Использовать:**
- Частые вставки/удаления в начале
- Неизвестный размер
- Нужна очередь или стек
- Реализация LRU cache

❌ **Не использовать:**
- Нужен быстрый доступ по индексу
- Ограниченная память (overhead на указатели)

## Stack (Стек)

### Описание

LIFO (Last In, First Out) — последним пришёл, первым ушёл.

```javascript
class Stack {
  constructor() {
    this.items = []
  }
  
  // Добавить элемент - O(1)
  push(element) {
    this.items.push(element)
  }
  
  // Удалить и вернуть верхний элемент - O(1)
  pop() {
    if (this.isEmpty()) {
      return null
    }
    return this.items.pop()
  }
  
  // Посмотреть верхний элемент - O(1)
  peek() {
    if (this.isEmpty()) {
      return null
    }
    return this.items[this.items.length - 1]
  }
  
  // Проверка на пустоту - O(1)
  isEmpty() {
    return this.items.length === 0
  }
  
  // Размер - O(1)
  size() {
    return this.items.length
  }
  
  // Очистить - O(1)
  clear() {
    this.items = []
  }
}

// Использование
const stack = new Stack()
stack.push(1)
stack.push(2)
stack.push(3)
console.log(stack.pop())  // 3
console.log(stack.peek())  // 2
```

### Применение

```javascript
// 1. Проверка сбалансированности скобок
function isBalanced(str) {
  const stack = []
  const pairs = { '(': ')', '[': ']', '{': '}' }
  
  for (const char of str) {
    if (char in pairs) {
      stack.push(char)
    } else if (Object.values(pairs).includes(char)) {
      if (stack.length === 0) return false
      const last = stack.pop()
      if (pairs[last] !== char) return false
    }
  }
  
  return stack.length === 0
}

console.log(isBalanced('()[]{}'))  // true
console.log(isBalanced('([)]'))  // false

// 2. Обратная польская нотация (RPN)
function evalRPN(tokens) {
  const stack = []
  
  for (const token of tokens) {
    if (['+', '-', '*', '/'].includes(token)) {
      const b = stack.pop()
      const a = stack.pop()
      
      switch (token) {
        case '+': stack.push(a + b); break
        case '-': stack.push(a - b); break
        case '*': stack.push(a * b); break
        case '/': stack.push(Math.trunc(a / b)); break
      }
    } else {
      stack.push(Number(token))
    }
  }
  
  return stack.pop()
}

console.log(evalRPN(['2', '1', '+', '3', '*']))  // (2 + 1) * 3 = 9

// 3. История браузера (back/forward)
class BrowserHistory {
  constructor() {
    this.backStack = []
    this.forwardStack = []
    this.current = null
  }
  
  visit(url) {
    if (this.current) {
      this.backStack.push(this.current)
    }
    this.current = url
    this.forwardStack = []  // Очистить forward при новом посещении
  }
  
  back() {
    if (this.backStack.length === 0) return this.current
    
    this.forwardStack.push(this.current)
    this.current = this.backStack.pop()
    return this.current
  }
  
  forward() {
    if (this.forwardStack.length === 0) return this.current
    
    this.backStack.push(this.current)
    this.current = this.forwardStack.pop()
    return this.current
  }
}
```

## Queue (Очередь)

### Описание

FIFO (First In, First Out) — первым пришёл, первым ушёл.

```javascript
class Queue {
  constructor() {
    this.items = []
  }
  
  // Добавить в конец - O(1)
  enqueue(element) {
    this.items.push(element)
  }
  
  // Удалить из начала - O(n) для массива
  dequeue() {
    if (this.isEmpty()) {
      return null
    }
    return this.items.shift()
  }
  
  // Посмотреть первый элемент - O(1)
  front() {
    if (this.isEmpty()) {
      return null
    }
    return this.items[0]
  }
  
  isEmpty() {
    return this.items.length === 0
  }
  
  size() {
    return this.items.length
  }
}

// Оптимизированная очередь на связном списке - O(1) для всех операций
class QueueOptimized {
  constructor() {
    this.head = null
    this.tail = null
    this.size = 0
  }
  
  enqueue(value) {
    const newNode = { value, next: null }
    
    if (!this.head) {
      this.head = newNode
      this.tail = newNode
    } else {
      this.tail.next = newNode
      this.tail = newNode
    }
    
    this.size++
  }
  
  dequeue() {
    if (!this.head) return null
    
    const value = this.head.value
    this.head = this.head.next
    
    if (!this.head) {
      this.tail = null
    }
    
    this.size--
    return value
  }
  
  front() {
    return this.head ? this.head.value : null
  }
  
  isEmpty() {
    return this.size === 0
  }
}
```

### Применение

```javascript
// 1. BFS (Breadth-First Search)
function bfs(graph, start) {
  const queue = [start]
  const visited = new Set([start])
  const result = []
  
  while (queue.length > 0) {
    const node = queue.shift()
    result.push(node)
    
    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor)
        queue.push(neighbor)
      }
    }
  }
  
  return result
}

// 2. Task Queue (обработка задач)
class TaskQueue {
  constructor() {
    this.queue = []
    this.processing = false
  }
  
  async addTask(task) {
    this.queue.push(task)
    
    if (!this.processing) {
      await this.processTasks()
    }
  }
  
  async processTasks() {
    this.processing = true
    
    while (this.queue.length > 0) {
      const task = this.queue.shift()
      await task()
    }
    
    this.processing = false
  }
}
```

## Hash Table (Хеш-таблица)

### Описание

Структура данных для хранения пар ключ-значение с быстрым доступом.

```javascript
class HashTable {
  constructor(size = 53) {
    this.keyMap = new Array(size)
  }
  
  // Хеш-функция
  _hash(key) {
    let total = 0
    const PRIME = 31
    
    for (let i = 0; i < Math.min(key.length, 100); i++) {
      const char = key[i]
      const value = char.charCodeAt(0) - 96
      total = (total * PRIME + value) % this.keyMap.length
    }
    
    return total
  }
  
  // Установить значение - O(1) средний случай
  set(key, value) {
    const index = this._hash(key)
    
    if (!this.keyMap[index]) {
      this.keyMap[index] = []
    }
    
    // Проверить, существует ли ключ
    for (let i = 0; i < this.keyMap[index].length; i++) {
      if (this.keyMap[index][i][0] === key) {
        this.keyMap[index][i][1] = value
        return
      }
    }
    
    // Добавить новую пару
    this.keyMap[index].push([key, value])
  }
  
  // Получить значение - O(1) средний случай
  get(key) {
    const index = this._hash(key)
    
    if (this.keyMap[index]) {
      for (let i = 0; i < this.keyMap[index].length; i++) {
        if (this.keyMap[index][i][0] === key) {
          return this.keyMap[index][i][1]
        }
      }
    }
    
    return undefined
  }
  
  // Удалить - O(1) средний случай
  delete(key) {
    const index = this._hash(key)
    
    if (this.keyMap[index]) {
      for (let i = 0; i < this.keyMap[index].length; i++) {
        if (this.keyMap[index][i][0] === key) {
          this.keyMap[index].splice(i, 1)
          return true
        }
      }
    }
    
    return false
  }
  
  // Получить все ключи - O(n)
  keys() {
    const keys = []
    
    for (let i = 0; i < this.keyMap.length; i++) {
      if (this.keyMap[i]) {
        for (let j = 0; j < this.keyMap[i].length; j++) {
          keys.push(this.keyMap[i][j][0])
        }
      }
    }
    
    return keys
  }
  
  // Получить все значения - O(n)
  values() {
    const values = []
    
    for (let i = 0; i < this.keyMap.length; i++) {
      if (this.keyMap[i]) {
        for (let j = 0; j < this.keyMap[i].length; j++) {
          if (!values.includes(this.keyMap[i][j][1])) {
            values.push(this.keyMap[i][j][1])
          }
        }
      }
    }
    
    return values
  }
}

// Использование
const ht = new HashTable()
ht.set('name', 'John')
ht.set('age', 30)
ht.set('city', 'New York')

console.log(ht.get('name'))  // 'John'
console.log(ht.keys())  // ['name', 'age', 'city']
```

### JavaScript Map и Set

```javascript
// Map - встроенная хеш-таблица
const map = new Map()

map.set('key1', 'value1')  // O(1)
map.get('key1')  // 'value1' - O(1)
map.has('key1')  // true - O(1)
map.delete('key1')  // O(1)
map.size  // 0 - O(1)

// Можно использовать любые типы как ключи
map.set({}, 'object key')
map.set(function() {}, 'function key')

// Итерация
map.forEach((value, key) => {
  console.log(key, value)
})

for (const [key, value] of map) {
  console.log(key, value)
}

// Set - множество уникальных значений
const set = new Set()

set.add(1)  // O(1)
set.add(2)
set.add(2)  // Дубликат игнорируется
set.has(1)  // true - O(1)
set.delete(1)  // O(1)
set.size  // 1 - O(1)

// Удаление дубликатов из массива
const arr = [1, 2, 2, 3, 3, 4]
const unique = [...new Set(arr)]  // [1, 2, 3, 4]
```

### Применение

```javascript
// 1. Подсчёт частоты элементов
function countFrequency(arr) {
  const freq = new Map()
  
  for (const item of arr) {
    freq.set(item, (freq.get(item) || 0) + 1)
  }
  
  return freq
}

// 2. Two Sum с HashMap
function twoSum(nums, target) {
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

// 3. LRU Cache
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity
    this.cache = new Map()
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1
    
    // Переместить в конец (самый свежий)
    const value = this.cache.get(key)
    this.cache.delete(key)
    this.cache.set(key, value)
    
    return value
  }
  
  put(key, value) {
    // Удалить если существует
    if (this.cache.has(key)) {
      this.cache.delete(key)
    }
    
    // Добавить в конец
    this.cache.set(key, value)
    
    // Удалить самый старый если превышен размер
    if (this.cache.size > this.capacity) {
      const firstKey = this.cache.keys().next().value
      this.cache.delete(firstKey)
    }
  }
}
```

## Tree (Дерево)

### Binary Tree (Бинарное дерево)

```javascript
class TreeNode {
  constructor(value) {
    this.value = value
    this.left = null
    this.right = null
  }
}

class BinaryTree {
  constructor() {
    this.root = null
  }
  
  // Обход в глубину (DFS)
  
  // Pre-order: корень → левый → правый
  preOrder(node = this.root, result = []) {
    if (node) {
      result.push(node.value)
      this.preOrder(node.left, result)
      this.preOrder(node.right, result)
    }
    return result
  }
  
  // In-order: левый → корень → правый
  inOrder(node = this.root, result = []) {
    if (node) {
      this.inOrder(node.left, result)
      result.push(node.value)
      this.inOrder(node.right, result)
    }
    return result
  }
  
  // Post-order: левый → правый → корень
  postOrder(node = this.root, result = []) {
    if (node) {
      this.postOrder(node.left, result)
      this.postOrder(node.right, result)
      result.push(node.value)
    }
    return result
  }
  
  // Обход в ширину (BFS)
  levelOrder() {
    if (!this.root) return []
    
    const result = []
    const queue = [this.root]
    
    while (queue.length > 0) {
      const node = queue.shift()
      result.push(node.value)
      
      if (node.left) queue.push(node.left)
      if (node.right) queue.push(node.right)
    }
    
    return result
  }
  
  // Высота дерева
  height(node = this.root) {
    if (!node) return 0
    
    const leftHeight = this.height(node.left)
    const rightHeight = this.height(node.right)
    
    return 1 + Math.max(leftHeight, rightHeight)
  }
}

// Создание дерева
const tree = new BinaryTree()
tree.root = new TreeNode(1)
tree.root.left = new TreeNode(2)
tree.root.right = new TreeNode(3)
tree.root.left.left = new TreeNode(4)
tree.root.left.right = new TreeNode(5)

/*
       1
      / \
     2   3
    / \
   4   5
*/

console.log(tree.preOrder())   // [1, 2, 4, 5, 3]
console.log(tree.inOrder())    // [4, 2, 5, 1, 3]
console.log(tree.postOrder())  // [4, 5, 2, 3, 1]
console.log(tree.levelOrder()) // [1, 2, 3, 4, 5]
```

### Binary Search Tree (BST)

```javascript
class BSTNode {
  constructor(value) {
    this.value = value
    this.left = null
    this.right = null
  }
}

class BinarySearchTree {
  constructor() {
    this.root = null
  }
  
  // Вставка - O(log n) средний, O(n) худший
  insert(value) {
    const newNode = new BSTNode(value)
    
    if (!this.root) {
      this.root = newNode
      return this
    }
    
    let current = this.root
    
    while (true) {
      if (value === current.value) return undefined  // Дубликат
      
      if (value < current.value) {
        if (!current.left) {
          current.left = newNode
          return this
        }
        current = current.left
      } else {
        if (!current.right) {
          current.right = newNode
          return this
        }
        current = current.right
      }
    }
  }
  
  // Поиск - O(log n) средний, O(n) худший
  find(value) {
    if (!this.root) return null
    
    let current = this.root
    
    while (current) {
      if (value === current.value) return current
      
      if (value < current.value) {
        current = current.left
      } else {
        current = current.right
      }
    }
    
    return null
  }
  
  // Проверка на BST
  isValidBST(node = this.root, min = -Infinity, max = Infinity) {
    if (!node) return true
    
    if (node.value <= min || node.value >= max) {
      return false
    }
    
    return this.isValidBST(node.left, min, node.value) &&
           this.isValidBST(node.right, node.value, max)
  }
}

// Использование
const bst = new BinarySearchTree()
bst.insert(10)
bst.insert(5)
bst.insert(15)
bst.insert(2)
bst.insert(7)

/*
      10
     /  \
    5    15
   / \
  2   7
*/

console.log(bst.find(7))  // BSTNode { value: 7, ... }
console.log(bst.find(20))  // null
```

### Heap (Куча)

```javascript
class MaxHeap {
  constructor() {
    this.values = []
  }
  
  // Вставка - O(log n)
  insert(value) {
    this.values.push(value)
    this.bubbleUp()
  }
  
  bubbleUp() {
    let idx = this.values.length - 1
    const element = this.values[idx]
    
    while (idx > 0) {
      const parentIdx = Math.floor((idx - 1) / 2)
      const parent = this.values[parentIdx]
      
      if (element <= parent) break
      
      this.values[idx] = parent
      this.values[parentIdx] = element
      idx = parentIdx
    }
  }
  
  // Извлечь максимум - O(log n)
  extractMax() {
    const max = this.values[0]
    const end = this.values.pop()
    
    if (this.values.length > 0) {
      this.values[0] = end
      this.sinkDown()
    }
    
    return max
  }
  
  sinkDown() {
    let idx = 0
    const length = this.values.length
    const element = this.values[0]
    
    while (true) {
      const leftChildIdx = 2 * idx + 1
      const rightChildIdx = 2 * idx + 2
      let leftChild, rightChild
      let swap = null
      
      if (leftChildIdx < length) {
        leftChild = this.values[leftChildIdx]
        if (leftChild > element) {
          swap = leftChildIdx
        }
      }
      
      if (rightChildIdx < length) {
        rightChild = this.values[rightChildIdx]
        if (
          (swap === null && rightChild > element) ||
          (swap !== null && rightChild > leftChild)
        ) {
          swap = rightChildIdx
        }
      }
      
      if (swap === null) break
      
      this.values[idx] = this.values[swap]
      this.values[swap] = element
      idx = swap
    }
  }
}

// Использование
const heap = new MaxHeap()
heap.insert(41)
heap.insert(39)
heap.insert(33)
heap.insert(18)
heap.insert(27)
heap.insert(12)
heap.insert(55)

console.log(heap.extractMax())  // 55
console.log(heap.extractMax())  // 41
```

## Graph (Граф)

### Представление графа

```javascript
// 1. Adjacency List (Список смежности)
class Graph {
  constructor() {
    this.adjacencyList = {}
  }
  
  // Добавить вершину
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = []
    }
  }
  
  // Добавить ребро
  addEdge(v1, v2) {
    this.adjacencyList[v1].push(v2)
    this.adjacencyList[v2].push(v1)  // Для неориентированного графа
  }
  
  // Удалить ребро
  removeEdge(v1, v2) {
    this.adjacencyList[v1] = this.adjacencyList[v1].filter(v => v !== v2)
    this.adjacencyList[v2] = this.adjacencyList[v2].filter(v => v !== v1)
  }
  
  // Удалить вершину
  removeVertex(vertex) {
    while (this.adjacencyList[vertex].length) {
      const adjacentVertex = this.adjacencyList[vertex].pop()
      this.removeEdge(vertex, adjacentVertex)
    }
    delete this.adjacencyList[vertex]
  }
  
  // DFS (рекурсивный)
  dfsRecursive(start) {
    const result = []
    const visited = {}
    const adjacencyList = this.adjacencyList
    
    function dfs(vertex) {
      if (!vertex) return
      
      visited[vertex] = true
      result.push(vertex)
      
      adjacencyList[vertex].forEach(neighbor => {
        if (!visited[neighbor]) {
          dfs(neighbor)
        }
      })
    }
    
    dfs(start)
    return result
  }
  
  // DFS (итеративный)
  dfsIterative(start) {
    const stack = [start]
    const result = []
    const visited = {}
    
    visited[start] = true
    
    while (stack.length) {
      const vertex = stack.pop()
      result.push(vertex)
      
      this.adjacencyList[vertex].forEach(neighbor => {
        if (!visited[neighbor]) {
          visited[neighbor] = true
          stack.push(neighbor)
        }
      })
    }
    
    return result
  }
  
  // BFS
  bfs(start) {
    const queue = [start]
    const result = []
    const visited = {}
    
    visited[start] = true
    
    while (queue.length) {
      const vertex = queue.shift()
      result.push(vertex)
      
      this.adjacencyList[vertex].forEach(neighbor => {
        if (!visited[neighbor]) {
          visited[neighbor] = true
          queue.push(neighbor)
        }
      })
    }
    
    return result
  }
}

// Использование
const graph = new Graph()

graph.addVertex('A')
graph.addVertex('B')
graph.addVertex('C')
graph.addVertex('D')
graph.addVertex('E')
graph.addVertex('F')

graph.addEdge('A', 'B')
graph.addEdge('A', 'C')
graph.addEdge('B', 'D')
graph.addEdge('C', 'E')
graph.addEdge('D', 'E')
graph.addEdge('D', 'F')
graph.addEdge('E', 'F')

/*
    A
   / \
  B   C
  |   |
  D - E
   \ /
    F
*/

console.log(graph.dfsRecursive('A'))  // ['A', 'B', 'D', 'E', 'C', 'F']
console.log(graph.bfs('A'))  // ['A', 'B', 'C', 'D', 'E', 'F']
```

### Взвешенный граф (Dijkstra)

```javascript
class PriorityQueue {
  constructor() {
    this.values = []
  }
  
  enqueue(val, priority) {
    this.values.push({ val, priority })
    this.sort()
  }
  
  dequeue() {
    return this.values.shift()
  }
  
  sort() {
    this.values.sort((a, b) => a.priority - b.priority)
  }
}

class WeightedGraph {
  constructor() {
    this.adjacencyList = {}
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = []
    }
  }
  
  addEdge(v1, v2, weight) {
    this.adjacencyList[v1].push({ node: v2, weight })
    this.adjacencyList[v2].push({ node: v1, weight })
  }
  
  // Алгоритм Дейкстры
  dijkstra(start, finish) {
    const nodes = new PriorityQueue()
    const distances = {}
    const previous = {}
    const path = []
    
    // Инициализация
    for (const vertex in this.adjacencyList) {
      if (vertex === start) {
        distances[vertex] = 0
        nodes.enqueue(vertex, 0)
      } else {
        distances[vertex] = Infinity
        nodes.enqueue(vertex, Infinity)
      }
      previous[vertex] = null
    }
    
    // Пока есть узлы для посещения
    while (nodes.values.length) {
      const smallest = nodes.dequeue().val
      
      if (smallest === finish) {
        // Построить путь
        while (previous[smallest]) {
          path.push(smallest)
          smallest = previous[smallest]
        }
        break
      }
      
      if (smallest || distances[smallest] !== Infinity) {
        for (const neighbor of this.adjacencyList[smallest]) {
          // Вычислить новое расстояние
          const candidate = distances[smallest] + neighbor.weight
          
          if (candidate < distances[neighbor.node]) {
            distances[neighbor.node] = candidate
            previous[neighbor.node] = smallest
            nodes.enqueue(neighbor.node, candidate)
          }
        }
      }
    }
    
    return path.concat(smallest).reverse()
  }
}

// Использование
const wg = new WeightedGraph()
wg.addVertex('A')
wg.addVertex('B')
wg.addVertex('C')
wg.addVertex('D')
wg.addVertex('E')
wg.addVertex('F')

wg.addEdge('A', 'B', 4)
wg.addEdge('A', 'C', 2)
wg.addEdge('B', 'E', 3)
wg.addEdge('C', 'D', 2)
wg.addEdge('C', 'F', 4)
wg.addEdge('D', 'E', 3)
wg.addEdge('D', 'F', 1)
wg.addEdge('E', 'F', 1)

console.log(wg.dijkstra('A', 'E'))  // ['A', 'C', 'D', 'F', 'E']
```

## Сравнение структур данных

| Структура | Доступ | Поиск | Вставка | Удаление | Память |
|-----------|--------|-------|---------|----------|--------|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Linked List | O(n) | O(n) | O(1) | O(1) | O(n) |
| Stack | O(n) | O(n) | O(1) | O(1) | O(n) |
| Queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash Table | - | O(1)* | O(1)* | O(1)* | O(n) |
| BST | O(log n)* | O(log n)* | O(log n)* | O(log n)* | O(n) |
| Heap | - | O(n) | O(log n) | O(log n) | O(n) |
| Graph (List) | - | O(V+E) | O(1) | O(E) | O(V+E) |

\* средний случай

## Вопросы для собеседования

### 1. В чём разница между массивом и связным списком?

**Ответ:**
**Array:**
- Последовательное хранение в памяти
- O(1) доступ по индексу
- O(n) вставка/удаление в начале

**Linked List:**
- Узлы разбросаны в памяти
- O(n) доступ по индексу
- O(1) вставка/удаление в начале

**Выбор:** массив для частого доступа, список для частых вставок.

### 2. Что такое хеш-коллизия и как её решить?

**Ответ:**
Коллизия — когда два ключа дают одинаковый хеш.

**Решения:**
1. **Chaining** (цепочки) — список в каждой ячейке
2. **Open Addressing** — ищем следующую свободную ячейку
3. **Double Hashing** — вторая хеш-функция

JavaScript Map/Set используют chaining.

### 3. В чём разница между стеком и очередью?

**Ответ:**
- **Stack (LIFO)**: последним пришёл, первым ушёл
  - Применение: undo/redo, вызовы функций, скобки
  
- **Queue (FIFO)**: первым пришёл, первым ушёл
  - Применение: BFS, task queue, печать

### 4. Что такое BST и его свойства?

**Ответ:**
Binary Search Tree — бинарное дерево где:
- Левое поддерево < корень
- Правое поддерево > корень

**Операции:** O(log n) в среднем, O(n) в худшем (несбалансированное).

**Балансировка:** AVL, Red-Black деревья гарантируют O(log n).

### 5. Когда использовать DFS vs BFS?

**Ответ:**
**DFS (стек/рекурсия):**
- Найти путь между узлами
- Топологическая сортировка
- Обнаружение циклов
- Меньше памяти

**BFS (очередь):**
- Кратчайший путь (невзвешенный граф)
- Уровни дерева
- Ближайшие соседи

### 6. Что такое heap и где используется?

**Ответ:**
Heap — полное бинарное дерево с heap property:
- **Max Heap**: родитель ≥ детей
- **Min Heap**: родитель ≤ детей

**Применение:**
- Priority Queue
- Heap Sort
- Dijkstra, Prim алгоритмы
- Топ K элементов

**Операции:** insert O(log n), extractMax O(log n)

### 7. Как реализовать LRU Cache?

**Ответ:**
Использовать **HashMap + Doubly Linked List**:
- HashMap для O(1) доступа
- Doubly Linked List для O(1) перемещения

В JavaScript: `Map` сохраняет порядок вставки.

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity
    this.cache = new Map()
  }
  
  get(key) {
    if (!this.cache.has(key)) return -1
    const val = this.cache.get(key)
    this.cache.delete(key)
    this.cache.set(key, val)  // Переместить в конец
    return val
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key)
    }
    this.cache.set(key, value)
    if (this.cache.size > this.capacity) {
      this.cache.delete(this.cache.keys().next().value)
    }
  }
}
```

### 8. Что такое Trie и где используется?

**Ответ:**
Trie (prefix tree) — дерево для хранения строк.

**Применение:**
- Автодополнение
- Проверка орфографии
- IP routing
- Поиск префиксов

**Сложность:** O(m) где m — длина строки.

### 9. Как представить граф в памяти?

**Ответ:**
1. **Adjacency Matrix** (матрица смежности)
   - O(V²) память
   - O(1) проверка ребра
   - Для плотных графов

2. **Adjacency List** (список смежности)
   - O(V + E) память
   - O(degree) проверка ребра
   - Для разреженных графов (обычно лучше)

### 10. В чём разница между Array и Set?

**Ответ:**
**Array:**
- Упорядоченная коллекция
- Разрешены дубликаты
- Доступ по индексу O(1)
- Поиск O(n)

**Set:**
- Неупорядоченная (порядок вставки)
- Только уникальные значения
- Нет доступа по индексу
- Поиск O(1)

**Выбор:** Set для проверки наличия и удаления дубликатов.
