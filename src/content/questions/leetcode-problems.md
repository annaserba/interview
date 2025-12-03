---
title: "LeetCode задачи для собеседований"
description: "Популярные алгоритмические задачи и паттерны решений"
category: "Лайв-кодинг"
difficulty: "hard"
tags: ["leetcode", "algorithms", "data-structures", "coding-interview"]
order: 35
---

## Популярные паттерны решений

### 1. Two Pointers (Два указателя)

Используется для работы с отсортированными массивами или строками.

#### Two Sum II (Sorted Array)

```javascript
/**
 * Найти два числа, сумма которых равна target
 * Input: numbers = [2,7,11,15], target = 9
 * Output: [1,2] (индексы 1-based)
 */
function twoSum(numbers, target) {
  let left = 0
  let right = numbers.length - 1
  
  while (left < right) {
    const sum = numbers[left] + numbers[right]
    
    if (sum === target) {
      return [left + 1, right + 1]  // 1-based индексы
    } else if (sum < target) {
      left++
    } else {
      right--
    }
  }
  
  return []
}

// Сложность: O(n) время, O(1) память
```

#### Valid Palindrome

```javascript
/**
 * Проверить, является ли строка палиндромом
 * Input: "A man, a plan, a canal: Panama"
 * Output: true
 */
function isPalindrome(s) {
  // Очистить строку от не-буквенно-цифровых символов
  const cleaned = s.toLowerCase().replace(/[^a-z0-9]/g, '')
  
  let left = 0
  let right = cleaned.length - 1
  
  while (left < right) {
    if (cleaned[left] !== cleaned[right]) {
      return false
    }
    left++
    right--
  }
  
  return true
}

// Сложность: O(n) время, O(n) память
```

#### Container With Most Water

```javascript
/**
 * Найти максимальную площадь воды между двумя линиями
 * Input: height = [1,8,6,2,5,4,8,3,7]
 * Output: 49
 */
function maxArea(height) {
  let left = 0
  let right = height.length - 1
  let maxArea = 0
  
  while (left < right) {
    const width = right - left
    const minHeight = Math.min(height[left], height[right])
    const area = width * minHeight
    
    maxArea = Math.max(maxArea, area)
    
    // Двигаем указатель с меньшей высотой
    if (height[left] < height[right]) {
      left++
    } else {
      right--
    }
  }
  
  return maxArea
}

// Сложность: O(n) время, O(1) память
```

### 2. Sliding Window (Скользящее окно)

Для подмассивов/подстрок с определёнными свойствами.

#### Longest Substring Without Repeating Characters

```javascript
/**
 * Найти длину самой длинной подстроки без повторяющихся символов
 * Input: s = "abcabcbb"
 * Output: 3 ("abc")
 */
function lengthOfLongestSubstring(s) {
  const seen = new Set()
  let left = 0
  let maxLength = 0
  
  for (let right = 0; right < s.length; right++) {
    // Если символ уже есть, сдвигаем левую границу
    while (seen.has(s[right])) {
      seen.delete(s[left])
      left++
    }
    
    seen.add(s[right])
    maxLength = Math.max(maxLength, right - left + 1)
  }
  
  return maxLength
}

// Сложность: O(n) время, O(min(n, m)) память (m - размер алфавита)
```

#### Minimum Window Substring

```javascript
/**
 * Найти минимальное окно в s, содержащее все символы из t
 * Input: s = "ADOBECODEBANC", t = "ABC"
 * Output: "BANC"
 */
function minWindow(s, t) {
  if (s.length < t.length) return ''
  
  // Подсчитать частоты символов в t
  const need = new Map()
  for (const char of t) {
    need.set(char, (need.get(char) || 0) + 1)
  }
  
  let left = 0
  let right = 0
  let required = need.size
  let formed = 0
  let minLen = Infinity
  let minLeft = 0
  
  const window = new Map()
  
  while (right < s.length) {
    const char = s[right]
    window.set(char, (window.get(char) || 0) + 1)
    
    // Если достигли нужной частоты символа
    if (need.has(char) && window.get(char) === need.get(char)) {
      formed++
    }
    
    // Пытаемся сжать окно
    while (formed === required && left <= right) {
      // Обновить результат
      if (right - left + 1 < minLen) {
        minLen = right - left + 1
        minLeft = left
      }
      
      const leftChar = s[left]
      window.set(leftChar, window.get(leftChar) - 1)
      
      if (need.has(leftChar) && window.get(leftChar) < need.get(leftChar)) {
        formed--
      }
      
      left++
    }
    
    right++
  }
  
  return minLen === Infinity ? '' : s.substring(minLeft, minLeft + minLen)
}

// Сложность: O(|S| + |T|) время, O(|S| + |T|) память
```

### 3. Binary Search (Бинарный поиск)

Для отсортированных данных или поиска оптимального значения.

#### Search in Rotated Sorted Array

```javascript
/**
 * Поиск в повёрнутом отсортированном массиве
 * Input: nums = [4,5,6,7,0,1,2], target = 0
 * Output: 4
 */
function search(nums, target) {
  let left = 0
  let right = nums.length - 1
  
  while (left <= right) {
    const mid = Math.floor((left + right) / 2)
    
    if (nums[mid] === target) {
      return mid
    }
    
    // Определить, какая половина отсортирована
    if (nums[left] <= nums[mid]) {
      // Левая половина отсортирована
      if (nums[left] <= target && target < nums[mid]) {
        right = mid - 1
      } else {
        left = mid + 1
      }
    } else {
      // Правая половина отсортирована
      if (nums[mid] < target && target <= nums[right]) {
        left = mid + 1
      } else {
        right = mid - 1
      }
    }
  }
  
  return -1
}

// Сложность: O(log n) время, O(1) память
```

#### Find Minimum in Rotated Sorted Array

```javascript
/**
 * Найти минимум в повёрнутом отсортированном массиве
 * Input: nums = [3,4,5,1,2]
 * Output: 1
 */
function findMin(nums) {
  let left = 0
  let right = nums.length - 1
  
  while (left < right) {
    const mid = Math.floor((left + right) / 2)
    
    if (nums[mid] > nums[right]) {
      // Минимум справа
      left = mid + 1
    } else {
      // Минимум слева или mid
      right = mid
    }
  }
  
  return nums[left]
}

// Сложность: O(log n) время, O(1) память
```

### 4. Dynamic Programming (Динамическое программирование)

#### Climbing Stairs

```javascript
/**
 * Сколькими способами можно подняться на n ступенек (1 или 2 за раз)
 * Input: n = 3
 * Output: 3 (1+1+1, 1+2, 2+1)
 */
function climbStairs(n) {
  if (n <= 2) return n
  
  let prev2 = 1  // n=1
  let prev1 = 2  // n=2
  
  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2
    prev2 = prev1
    prev1 = current
  }
  
  return prev1
}

// Сложность: O(n) время, O(1) память
```

#### Coin Change

```javascript
/**
 * Минимальное количество монет для суммы amount
 * Input: coins = [1,2,5], amount = 11
 * Output: 3 (5+5+1)
 */
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity)
  dp[0] = 0
  
  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (i >= coin) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1)
      }
    }
  }
  
  return dp[amount] === Infinity ? -1 : dp[amount]
}

// Сложность: O(amount * coins.length) время, O(amount) память
```

#### Longest Increasing Subsequence

```javascript
/**
 * Длина самой длинной возрастающей подпоследовательности
 * Input: nums = [10,9,2,5,3,7,101,18]
 * Output: 4 ([2,3,7,101])
 */
function lengthOfLIS(nums) {
  if (nums.length === 0) return 0
  
  const dp = new Array(nums.length).fill(1)
  
  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[i] > nums[j]) {
        dp[i] = Math.max(dp[i], dp[j] + 1)
      }
    }
  }
  
  return Math.max(...dp)
}

// Сложность: O(n²) время, O(n) память

// Оптимизированное решение с бинарным поиском O(n log n)
function lengthOfLISOptimized(nums) {
  const tails = []
  
  for (const num of nums) {
    let left = 0
    let right = tails.length
    
    // Бинарный поиск позиции для вставки
    while (left < right) {
      const mid = Math.floor((left + right) / 2)
      if (tails[mid] < num) {
        left = mid + 1
      } else {
        right = mid
      }
    }
    
    if (left === tails.length) {
      tails.push(num)
    } else {
      tails[left] = num
    }
  }
  
  return tails.length
}
```

### 5. Backtracking (Перебор с возвратом)

#### Permutations

```javascript
/**
 * Все перестановки массива
 * Input: nums = [1,2,3]
 * Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
 */
function permute(nums) {
  const result = []
  
  function backtrack(current, remaining) {
    if (remaining.length === 0) {
      result.push([...current])
      return
    }
    
    for (let i = 0; i < remaining.length; i++) {
      current.push(remaining[i])
      const newRemaining = [...remaining.slice(0, i), ...remaining.slice(i + 1)]
      backtrack(current, newRemaining)
      current.pop()
    }
  }
  
  backtrack([], nums)
  return result
}

// Сложность: O(n! * n) время, O(n!) память
```

#### Combination Sum

```javascript
/**
 * Все комбинации чисел, дающие target (можно повторять)
 * Input: candidates = [2,3,6,7], target = 7
 * Output: [[2,2,3],[7]]
 */
function combinationSum(candidates, target) {
  const result = []
  
  function backtrack(start, current, sum) {
    if (sum === target) {
      result.push([...current])
      return
    }
    
    if (sum > target) return
    
    for (let i = start; i < candidates.length; i++) {
      current.push(candidates[i])
      backtrack(i, current, sum + candidates[i])  // i, не i+1 (можно повторять)
      current.pop()
    }
  }
  
  backtrack(0, [], 0)
  return result
}

// Сложность: O(n^(target/min)) время
```

#### Generate Parentheses

```javascript
/**
 * Все комбинации правильных скобок для n пар
 * Input: n = 3
 * Output: ["((()))","(()())","(())()","()(())","()()()"]
 */
function generateParenthesis(n) {
  const result = []
  
  function backtrack(current, open, close) {
    if (current.length === 2 * n) {
      result.push(current)
      return
    }
    
    if (open < n) {
      backtrack(current + '(', open + 1, close)
    }
    
    if (close < open) {
      backtrack(current + ')', open, close + 1)
    }
  }
  
  backtrack('', 0, 0)
  return result
}

// Сложность: O(4^n / √n) время (Catalan number)
```

### 6. Graph (Графы)

#### Number of Islands

```javascript
/**
 * Количество островов в сетке
 * Input: grid = [
 *   ["1","1","0","0","0"],
 *   ["1","1","0","0","0"],
 *   ["0","0","1","0","0"],
 *   ["0","0","0","1","1"]
 * ]
 * Output: 3
 */
function numIslands(grid) {
  if (!grid || grid.length === 0) return 0
  
  const rows = grid.length
  const cols = grid[0].length
  let count = 0
  
  function dfs(i, j) {
    if (i < 0 || i >= rows || j < 0 || j >= cols || grid[i][j] === '0') {
      return
    }
    
    grid[i][j] = '0'  // Пометить как посещённую
    
    // Обойти соседей
    dfs(i + 1, j)
    dfs(i - 1, j)
    dfs(i, j + 1)
    dfs(i, j - 1)
  }
  
  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      if (grid[i][j] === '1') {
        count++
        dfs(i, j)
      }
    }
  }
  
  return count
}

// Сложность: O(m * n) время, O(m * n) память (рекурсия)
```

#### Course Schedule (Topological Sort)

```javascript
/**
 * Можно ли пройти все курсы с учётом prerequisites
 * Input: numCourses = 2, prerequisites = [[1,0]]
 * Output: true (сначала курс 0, потом 1)
 */
function canFinish(numCourses, prerequisites) {
  // Построить граф смежности
  const graph = Array.from({ length: numCourses }, () => [])
  for (const [course, prereq] of prerequisites) {
    graph[prereq].push(course)
  }
  
  const visited = new Array(numCourses).fill(0)
  // 0 - не посещён, 1 - в процессе, 2 - завершён
  
  function hasCycle(course) {
    if (visited[course] === 1) return true  // Цикл!
    if (visited[course] === 2) return false
    
    visited[course] = 1
    
    for (const next of graph[course]) {
      if (hasCycle(next)) return true
    }
    
    visited[course] = 2
    return false
  }
  
  for (let i = 0; i < numCourses; i++) {
    if (hasCycle(i)) return false
  }
  
  return true
}

// Сложность: O(V + E) время, O(V + E) память
```

### 7. Tree (Деревья)

#### Maximum Depth of Binary Tree

```javascript
/**
 * Максимальная глубина бинарного дерева
 */
function maxDepth(root) {
  if (!root) return 0
  
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right))
}

// Сложность: O(n) время, O(h) память (h - высота)
```

#### Validate Binary Search Tree

```javascript
/**
 * Проверить, является ли дерево корректным BST
 */
function isValidBST(root) {
  function validate(node, min, max) {
    if (!node) return true
    
    if (node.val <= min || node.val >= max) {
      return false
    }
    
    return validate(node.left, min, node.val) && 
           validate(node.right, node.val, max)
  }
  
  return validate(root, -Infinity, Infinity)
}

// Сложность: O(n) время, O(h) память
```

#### Lowest Common Ancestor

```javascript
/**
 * Найти наименьшего общего предка двух узлов
 */
function lowestCommonAncestor(root, p, q) {
  if (!root || root === p || root === q) {
    return root
  }
  
  const left = lowestCommonAncestor(root.left, p, q)
  const right = lowestCommonAncestor(root.right, p, q)
  
  if (left && right) return root
  
  return left || right
}

// Сложность: O(n) время, O(h) память
```

### 8. Linked List (Связные списки)

#### Reverse Linked List

```javascript
/**
 * Развернуть связный список
 * Input: 1->2->3->4->5
 * Output: 5->4->3->2->1
 */
function reverseList(head) {
  let prev = null
  let current = head
  
  while (current) {
    const next = current.next
    current.next = prev
    prev = current
    current = next
  }
  
  return prev
}

// Сложность: O(n) время, O(1) память
```

#### Merge Two Sorted Lists

```javascript
/**
 * Слить два отсортированных списка
 */
function mergeTwoLists(l1, l2) {
  const dummy = { next: null }
  let current = dummy
  
  while (l1 && l2) {
    if (l1.val < l2.val) {
      current.next = l1
      l1 = l1.next
    } else {
      current.next = l2
      l2 = l2.next
    }
    current = current.next
  }
  
  current.next = l1 || l2
  
  return dummy.next
}

// Сложность: O(n + m) время, O(1) память
```

#### Detect Cycle

```javascript
/**
 * Определить, есть ли цикл в списке (Floyd's Algorithm)
 */
function hasCycle(head) {
  let slow = head
  let fast = head
  
  while (fast && fast.next) {
    slow = slow.next
    fast = fast.next.next
    
    if (slow === fast) {
      return true
    }
  }
  
  return false
}

// Сложность: O(n) время, O(1) память
```

## Топ-10 задач для подготовки

### 1. Two Sum (Easy)
Найти два числа в массиве, дающие target сумму.

### 2. Valid Parentheses (Easy)
Проверить правильность расстановки скобок.

### 3. Merge Two Sorted Lists (Easy)
Слить два отсортированных связных списка.

### 4. Best Time to Buy and Sell Stock (Easy)
Найти максимальную прибыль от покупки/продажи акций.

### 5. Maximum Subarray (Medium)
Найти подмассив с максимальной суммой (Kadane's Algorithm).

### 6. 3Sum (Medium)
Найти все тройки чисел с суммой 0.

### 7. Container With Most Water (Medium)
Найти максимальную площадь воды между линиями.

### 8. Longest Substring Without Repeating Characters (Medium)
Самая длинная подстрока без повторов.

### 9. Merge Intervals (Medium)
Объединить пересекающиеся интервалы.

### 10. LRU Cache (Medium)
Реализовать LRU кэш с O(1) операциями.

## Советы по решению

### 1. Понять задачу
- Прочитать несколько раз
- Разобрать примеры
- Уточнить edge cases

### 2. Придумать подход
- Brute force сначала
- Оптимизировать потом
- Выбрать структуру данных

### 3. Написать код
- Начать с простого
- Тестировать на ходу
- Обрабатывать edge cases

### 4. Анализ сложности
- Время: O(?)
- Память: O(?)
- Можно ли лучше?

### 5. Тестирование
- Обычные случаи
- Граничные случаи
- Пустой ввод

## Вопросы для собеседования

### 1. Какие паттерны решений вы знаете?

**Ответ:**
- Two Pointers
- Sliding Window
- Binary Search
- Dynamic Programming
- Backtracking
- BFS/DFS
- Greedy

### 2. Как оптимизировать решение?

**Ответ:**
1. Использовать правильную структуру данных
2. Избегать вложенных циклов (HashMap)
3. Сортировка может помочь
4. Кэширование результатов (DP)
5. Два прохода лучше, чем O(n²)

### 3. В чём разница между BFS и DFS?

**Ответ:**
- **BFS**: очередь, кратчайший путь, уровни
- **DFS**: стек/рекурсия, все пути, топологическая сортировка

BFS для кратчайшего пути, DFS для обхода всех вариантов.

### 4. Когда использовать HashMap?

**Ответ:**
- Поиск за O(1)
- Подсчёт частот
- Проверка дубликатов
- Two Sum задачи

Компромисс: O(n) память за O(1) поиск.

### 5. Что такое динамическое программирование?

**Ответ:**
Оптимизация через сохранение результатов подзадач.

Признаки:
- Оптимальная подструктура
- Перекрывающиеся подзадачи

Подходы: top-down (мемоизация), bottom-up (табуляция).
