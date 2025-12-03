---
title: "JavaScript: Продвинутые концепции"
description: "Сложные темы JavaScript для senior разработчиков"
category: "JavaScript"
difficulty: "hard"
tags: ["javascript", "advanced", "prototypes", "memory", "optimization"]
order: 26
---

## Прототипы и наследование

### Прототипная цепочка

```javascript
function Animal(name) {
  this.name = name
}

Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound`)
}

function Dog(name, breed) {
  Animal.call(this, name) // Вызов конструктора родителя
  this.breed = breed
}

// Наследование
Dog.prototype = Object.create(Animal.prototype)
Dog.prototype.constructor = Dog

Dog.prototype.bark = function() {
  console.log(`${this.name} barks`)
}

const dog = new Dog('Rex', 'Labrador')
dog.speak() // Rex makes a sound
dog.bark()  // Rex barks

// Проверка цепочки прототипов
console.log(dog instanceof Dog)    // true
console.log(dog instanceof Animal) // true
console.log(Object.getPrototypeOf(dog) === Dog.prototype) // true
```

### Object.create vs new

```javascript
// Object.create - создаёт объект с указанным прототипом
const proto = {
  greet() {
    console.log(`Hello, ${this.name}`)
  }
}

const person = Object.create(proto)
person.name = 'John'
person.greet() // Hello, John

// new - вызывает конструктор и устанавливает прототип
function Person(name) {
  this.name = name
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`)
}

const person2 = new Person('Jane')
person2.greet() // Hello, Jane
```

## Генераторы и итераторы

### Итераторы

```javascript
const iterableObject = {
  data: [1, 2, 3, 4, 5],
  
  [Symbol.iterator]() {
    let index = 0
    const data = this.data
    
    return {
      next() {
        if (index < data.length) {
          return { value: data[index++], done: false }
        }
        return { value: undefined, done: true }
      }
    }
  }
}

for (const value of iterableObject) {
  console.log(value) // 1, 2, 3, 4, 5
}

// Ручное использование итератора
const iterator = iterableObject[Symbol.iterator]()
console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next()) // { value: 2, done: false }
```

### Генераторы

```javascript
function* numberGenerator() {
  yield 1
  yield 2
  yield 3
  return 4 // не попадёт в for...of
}

const gen = numberGenerator()
console.log(gen.next()) // { value: 1, done: false }
console.log(gen.next()) // { value: 2, done: false }
console.log(gen.next()) // { value: 3, done: false }
console.log(gen.next()) // { value: 4, done: true }

// Бесконечный генератор
function* infiniteSequence() {
  let i = 0
  while (true) {
    yield i++
  }
}

// Генератор с параметрами
function* fibonacci() {
  let [prev, curr] = [0, 1]
  while (true) {
    yield curr
    ;[prev, curr] = [curr, prev + curr]
  }
}

const fib = fibonacci()
console.log(fib.next().value) // 1
console.log(fib.next().value) // 1
console.log(fib.next().value) // 2
console.log(fib.next().value) // 3
console.log(fib.next().value) // 5

// Делегирование генераторов
function* gen1() {
  yield 1
  yield 2
}

function* gen2() {
  yield* gen1() // Делегирование
  yield 3
  yield 4
}

console.log([...gen2()]) // [1, 2, 3, 4]
```

### Практическое применение

```javascript
// Ленивая загрузка данных
function* lazyFetch(url, pageSize = 10) {
  let page = 1
  let hasMore = true
  
  while (hasMore) {
    const response = await fetch(`${url}?page=${page}&size=${pageSize}`)
    const data = await response.json()
    
    if (data.length < pageSize) {
      hasMore = false
    }
    
    yield* data
    page++
  }
}

// Использование
async function loadAllUsers() {
  for await (const user of lazyFetch('/api/users')) {
    console.log(user)
  }
}
```

## Proxy и Reflect

### Proxy для валидации

```javascript
const validator = {
  set(target, property, value) {
    if (property === 'age') {
      if (typeof value !== 'number' || value < 0 || value > 150) {
        throw new TypeError('Age must be a number between 0 and 150')
      }
    }
    
    if (property === 'email') {
      if (!value.includes('@')) {
        throw new TypeError('Invalid email')
      }
    }
    
    target[property] = value
    return true
  }
}

const person = new Proxy({}, validator)
person.age = 30      // OK
person.email = 'test@example.com' // OK
// person.age = -5   // Error: Age must be a number between 0 and 150
```

### Proxy для логирования

```javascript
function createLoggingProxy(target, name) {
  return new Proxy(target, {
    get(target, property) {
      console.log(`[${name}] Getting ${String(property)}`)
      return Reflect.get(target, property)
    },
    
    set(target, property, value) {
      console.log(`[${name}] Setting ${String(property)} = ${value}`)
      return Reflect.set(target, property, value)
    },
    
    deleteProperty(target, property) {
      console.log(`[${name}] Deleting ${String(property)}`)
      return Reflect.deleteProperty(target, property)
    }
  })
}

const user = createLoggingProxy({ name: 'John' }, 'User')
user.age = 30        // [User] Setting age = 30
console.log(user.name) // [User] Getting name
delete user.age      // [User] Deleting age
```

### Реактивность на Proxy

```javascript
function reactive(target) {
  const handlers = new Map()
  
  const proxy = new Proxy(target, {
    get(target, property) {
      return Reflect.get(target, property)
    },
    
    set(target, property, value) {
      const oldValue = target[property]
      const result = Reflect.set(target, property, value)
      
      if (oldValue !== value) {
        // Вызвать все подписчики
        const propertyHandlers = handlers.get(property)
        if (propertyHandlers) {
          propertyHandlers.forEach(handler => handler(value, oldValue))
        }
      }
      
      return result
    }
  })
  
  proxy.$watch = function(property, handler) {
    if (!handlers.has(property)) {
      handlers.set(property, new Set())
    }
    handlers.get(property).add(handler)
    
    // Вернуть функцию отписки
    return () => handlers.get(property).delete(handler)
  }
  
  return proxy
}

// Использование
const state = reactive({ count: 0 })

const unwatch = state.$watch('count', (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`)
})

state.count = 1 // Count changed from 0 to 1
state.count = 2 // Count changed from 1 to 2

unwatch()
state.count = 3 // Ничего не выведется
```

## WeakMap и WeakSet

### WeakMap для приватных данных

```javascript
const privateData = new WeakMap()

class User {
  constructor(name, password) {
    this.name = name
    privateData.set(this, { password })
  }
  
  checkPassword(password) {
    return privateData.get(this).password === password
  }
  
  changePassword(oldPassword, newPassword) {
    if (this.checkPassword(oldPassword)) {
      privateData.get(this).password = newPassword
      return true
    }
    return false
  }
}

const user = new User('John', 'secret123')
console.log(user.password) // undefined
console.log(user.checkPassword('secret123')) // true
```

### WeakMap для кэширования

```javascript
const cache = new WeakMap()

function expensiveOperation(obj) {
  if (cache.has(obj)) {
    console.log('Returning cached result')
    return cache.get(obj)
  }
  
  console.log('Computing result')
  const result = { computed: true, data: obj.value * 2 }
  cache.set(obj, result)
  return result
}

const obj1 = { value: 5 }
expensiveOperation(obj1) // Computing result
expensiveOperation(obj1) // Returning cached result

// Когда obj1 будет удалён, кэш автоматически очистится
```

## Управление памятью

### Memory Leaks

```javascript
// ❌ Утечка памяти - забыли отписаться
class Component {
  constructor() {
    window.addEventListener('resize', this.handleResize)
  }
  
  handleResize() {
    console.log('Resized')
  }
}

// ✅ Правильно - отписываемся
class Component {
  constructor() {
    this.handleResize = this.handleResize.bind(this)
    window.addEventListener('resize', this.handleResize)
  }
  
  destroy() {
    window.removeEventListener('resize', this.handleResize)
  }
  
  handleResize() {
    console.log('Resized')
  }
}

// ❌ Утечка - замыкание держит ссылку
function createHeavyObject() {
  const heavyData = new Array(1000000).fill('data')
  
  return {
    getData: () => heavyData // Замыкание держит весь массив
  }
}

// ✅ Правильно - освобождаем память
function createHeavyObject() {
  let heavyData = new Array(1000000).fill('data')
  
  return {
    getData: () => {
      const data = heavyData
      heavyData = null // Освобождаем ссылку
      return data
    }
  }
}
```

### Object Pool

```javascript
class ObjectPool {
  constructor(factory, reset, initialSize = 10) {
    this.factory = factory
    this.reset = reset
    this.pool = []
    
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(factory())
    }
  }
  
  acquire() {
    return this.pool.length > 0 
      ? this.pool.pop() 
      : this.factory()
  }
  
  release(obj) {
    this.reset(obj)
    this.pool.push(obj)
  }
}

// Использование
const vectorPool = new ObjectPool(
  () => ({ x: 0, y: 0 }),
  (v) => { v.x = 0; v.y = 0 },
  100
)

function simulate() {
  const vectors = []
  
  for (let i = 0; i < 1000; i++) {
    const v = vectorPool.acquire()
    v.x = Math.random()
    v.y = Math.random()
    vectors.push(v)
  }
  
  // Освобождаем объекты
  vectors.forEach(v => vectorPool.release(v))
}
```

## Symbol

```javascript
// Уникальные ключи
const ID = Symbol('id')
const user = {
  name: 'John',
  [ID]: 12345
}

console.log(user[ID]) // 12345
console.log(Object.keys(user)) // ['name'] - Symbol не перечисляется

// Well-known symbols
const obj = {
  [Symbol.iterator]() {
    let i = 0
    return {
      next: () => ({ value: i++, done: i > 3 })
    }
  }
}

console.log([...obj]) // [0, 1, 2]

// Symbol.toStringTag
class MyClass {
  get [Symbol.toStringTag]() {
    return 'MyClass'
  }
}

console.log(Object.prototype.toString.call(new MyClass())) 
// [object MyClass]

// Symbol.toPrimitive
const obj2 = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return 42
    if (hint === 'string') return 'hello'
    return true
  }
}

console.log(+obj2)      // 42
console.log(`${obj2}`)  // hello
console.log(obj2 + '')  // true
```

## Вопросы для собеседования

### 1. Объясните прототипное наследование

**Ответ:**
В JavaScript объекты наследуют свойства и методы через прототипную цепочку. Каждый объект имеет внутреннюю ссылку `[[Prototype]]` на другой объект. При обращении к свойству JavaScript ищет его сначала в самом объекте, затем в прототипе, затем в прототипе прототипа и т.д.

```javascript
const obj = {}
console.log(obj.toString) // Найдено в Object.prototype
```

### 2. В чём разница между Map/Set и WeakMap/WeakSet?

**Ответ:**
- **Map/Set**: ключи могут быть любыми, сильные ссылки, перечисляемые
- **WeakMap/WeakSet**: ключи только объекты, слабые ссылки (позволяют GC), не перечисляемые

WeakMap/WeakSet используются для:
- Приватных данных
- Кэширования
- Метаданных объектов

### 3. Что такое генераторы и зачем они нужны?

**Ответ:**
Генераторы - функции, которые можно приостанавливать и возобновлять. Используются для:
- Ленивых вычислений
- Бесконечных последовательностей
- Асинхронных итераций
- Управления потоком выполнения

### 4. Как работает Proxy?

**Ответ:**
Proxy позволяет перехватывать и переопределять операции с объектами (get, set, delete и др.). Используется для:
- Валидации
- Логирования
- Реактивности (Vue 3)
- Виртуальных свойств

### 5. Как избежать memory leaks?

**Ответ:**
- Отписываться от событий
- Очищать таймеры и интервалы
- Использовать WeakMap/WeakSet
- Избегать глобальных переменных
- Освобождать ссылки на большие объекты
- Использовать Chrome DevTools Memory Profiler

### 6. Что такое Symbol и зачем он нужен?

**Ответ:**
Symbol - примитивный тип для создания уникальных идентификаторов. Используется для:
- Уникальных ключей объектов
- Well-known symbols (iterator, toStringTag)
- Приватных свойств
- Метапрограммирования

### 7. Реализуйте debounce и throttle

**Ответ:**
```javascript
function debounce(fn, delay) {
  let timeoutId
  return function(...args) {
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => fn.apply(this, args), delay)
  }
}

function throttle(fn, delay) {
  let lastCall = 0
  return function(...args) {
    const now = Date.now()
    if (now - lastCall >= delay) {
      lastCall = now
      fn.apply(this, args)
    }
  }
}
```

### 8. Что такое Event Loop?

**Ответ:**
Event Loop - механизм, который обрабатывает асинхронные операции в JavaScript:
1. Call Stack - выполнение синхронного кода
2. Web APIs - асинхронные операции (setTimeout, fetch)
3. Callback Queue - очередь макрозадач
4. Microtask Queue - очередь микрозадач (Promise, queueMicrotask)

Порядок: Call Stack → Microtasks → Macrotasks
