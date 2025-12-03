---
title: "Design Patterns в JavaScript"
description: "Паттерны проектирования и их применение в современном JavaScript"
category: "Architecture"
difficulty: "hard"
tags: ["patterns", "architecture", "oop", "design"]
order: 30
---

## Creational Patterns (Порождающие)

### Singleton

```javascript
// Гарантирует единственный экземпляр класса

class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance
    }
    
    this.connection = null
    Database.instance = this
  }
  
  connect() {
    if (!this.connection) {
      this.connection = { connected: true }
      console.log('Connected to database')
    }
    return this.connection
  }
}

const db1 = new Database()
const db2 = new Database()
console.log(db1 === db2) // true

// Современный подход с модулями
// database.js
class Database {
  connect() {
    console.log('Connected')
  }
}

export default new Database()

// Использование
import db from './database.js'
db.connect()
```

### Factory

```javascript
// Создание объектов без указания конкретного класса

class Button {
  constructor(type) {
    this.type = type
  }
  
  render() {
    return `<button class="${this.type}">Click me</button>`
  }
}

class Link {
  constructor(type) {
    this.type = type
  }
  
  render() {
    return `<a class="${this.type}" href="#">Click me</a>`
  }
}

class ElementFactory {
  createElement(type, elementType) {
    switch (elementType) {
      case 'button':
        return new Button(type)
      case 'link':
        return new Link(type)
      default:
        throw new Error('Unknown element type')
    }
  }
}

const factory = new ElementFactory()
const primaryButton = factory.createElement('primary', 'button')
const secondaryLink = factory.createElement('secondary', 'link')
```

### Builder

```javascript
// Пошаговое создание сложных объектов

class QueryBuilder {
  constructor() {
    this.query = {
      select: [],
      from: '',
      where: [],
      orderBy: [],
      limit: null
    }
  }
  
  select(...fields) {
    this.query.select.push(...fields)
    return this
  }
  
  from(table) {
    this.query.from = table
    return this
  }
  
  where(condition) {
    this.query.where.push(condition)
    return this
  }
  
  orderBy(field, direction = 'ASC') {
    this.query.orderBy.push({ field, direction })
    return this
  }
  
  limit(count) {
    this.query.limit = count
    return this
  }
  
  build() {
    let sql = `SELECT ${this.query.select.join(', ')} FROM ${this.query.from}`
    
    if (this.query.where.length > 0) {
      sql += ` WHERE ${this.query.where.join(' AND ')}`
    }
    
    if (this.query.orderBy.length > 0) {
      const orderBy = this.query.orderBy
        .map(o => `${o.field} ${o.direction}`)
        .join(', ')
      sql += ` ORDER BY ${orderBy}`
    }
    
    if (this.query.limit) {
      sql += ` LIMIT ${this.query.limit}`
    }
    
    return sql
  }
}

// Использование
const query = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('age > 18')
  .where('active = true')
  .orderBy('name', 'ASC')
  .limit(10)
  .build()

console.log(query)
// SELECT id, name, email FROM users WHERE age > 18 AND active = true ORDER BY name ASC LIMIT 10
```

## Structural Patterns (Структурные)

### Adapter

```javascript
// Адаптация интерфейса одного класса к другому

// Старый API
class OldAPI {
  getData() {
    return { data: [1, 2, 3] }
  }
}

// Новый API ожидает другой формат
class NewAPI {
  fetchData() {
    return { items: [], total: 0 }
  }
}

// Адаптер
class APIAdapter {
  constructor(oldAPI) {
    this.oldAPI = oldAPI
  }
  
  fetchData() {
    const result = this.oldAPI.getData()
    return {
      items: result.data,
      total: result.data.length
    }
  }
}

const oldAPI = new OldAPI()
const adapter = new APIAdapter(oldAPI)
const data = adapter.fetchData()
console.log(data) // { items: [1, 2, 3], total: 3 }
```

### Decorator

```javascript
// Динамическое добавление функциональности объекту

class Coffee {
  cost() {
    return 5
  }
  
  description() {
    return 'Coffee'
  }
}

class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee
  }
  
  cost() {
    return this.coffee.cost() + 2
  }
  
  description() {
    return this.coffee.description() + ', Milk'
  }
}

class SugarDecorator {
  constructor(coffee) {
    this.coffee = coffee
  }
  
  cost() {
    return this.coffee.cost() + 1
  }
  
  description() {
    return this.coffee.description() + ', Sugar'
  }
}

let coffee = new Coffee()
console.log(coffee.description(), coffee.cost()) // Coffee 5

coffee = new MilkDecorator(coffee)
console.log(coffee.description(), coffee.cost()) // Coffee, Milk 7

coffee = new SugarDecorator(coffee)
console.log(coffee.description(), coffee.cost()) // Coffee, Milk, Sugar 8

// Современный подход с декораторами
function log(target, name, descriptor) {
  const original = descriptor.value
  descriptor.value = function(...args) {
    console.log(`Calling ${name} with`, args)
    return original.apply(this, args)
  }
  return descriptor
}

class Calculator {
  @log
  add(a, b) {
    return a + b
  }
}
```

### Proxy

```javascript
// Контроль доступа к объекту

const user = {
  name: 'John',
  age: 30,
  _password: 'secret123'
}

const userProxy = new Proxy(user, {
  get(target, prop) {
    if (prop.startsWith('_')) {
      throw new Error('Access denied')
    }
    console.log(`Getting ${String(prop)}`)
    return target[prop]
  },
  
  set(target, prop, value) {
    if (prop === 'age' && typeof value !== 'number') {
      throw new TypeError('Age must be a number')
    }
    console.log(`Setting ${String(prop)} = ${value}`)
    target[prop] = value
    return true
  }
})

console.log(userProxy.name) // Getting name, John
userProxy.age = 31 // Setting age = 31
// userProxy._password // Error: Access denied
```

### Facade

```javascript
// Упрощённый интерфейс к сложной системе

class CPU {
  freeze() { console.log('CPU: Freeze') }
  jump(position) { console.log(`CPU: Jump to ${position}`) }
  execute() { console.log('CPU: Execute') }
}

class Memory {
  load(position, data) {
    console.log(`Memory: Load ${data} at ${position}`)
  }
}

class HardDrive {
  read(sector, size) {
    console.log(`HardDrive: Read ${size} bytes from sector ${sector}`)
    return 'boot data'
  }
}

// Фасад
class ComputerFacade {
  constructor() {
    this.cpu = new CPU()
    this.memory = new Memory()
    this.hardDrive = new HardDrive()
  }
  
  start() {
    console.log('Starting computer...')
    this.cpu.freeze()
    const bootData = this.hardDrive.read(0, 1024)
    this.memory.load(0, bootData)
    this.cpu.jump(0)
    this.cpu.execute()
    console.log('Computer started!')
  }
}

const computer = new ComputerFacade()
computer.start()
```

## Behavioral Patterns (Поведенческие)

### Observer (Pub/Sub)

```javascript
// Подписка на события

class EventEmitter {
  constructor() {
    this.events = {}
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = []
    }
    this.events[event].push(listener)
    
    // Возврат функции отписки
    return () => this.off(event, listener)
  }
  
  off(event, listener) {
    if (!this.events[event]) return
    
    this.events[event] = this.events[event].filter(l => l !== listener)
  }
  
  emit(event, ...args) {
    if (!this.events[event]) return
    
    this.events[event].forEach(listener => {
      listener(...args)
    })
  }
  
  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args)
      this.off(event, wrapper)
    }
    this.on(event, wrapper)
  }
}

// Использование
const emitter = new EventEmitter()

const unsubscribe = emitter.on('user:login', (user) => {
  console.log('User logged in:', user.name)
})

emitter.emit('user:login', { name: 'John' })
unsubscribe()
emitter.emit('user:login', { name: 'Jane' }) // Не выведется
```

### Strategy

```javascript
// Выбор алгоритма во время выполнения

class PaymentStrategy {
  pay(amount) {
    throw new Error('pay() must be implemented')
  }
}

class CreditCardStrategy extends PaymentStrategy {
  constructor(cardNumber) {
    super()
    this.cardNumber = cardNumber
  }
  
  pay(amount) {
    console.log(`Paid ${amount} with credit card ${this.cardNumber}`)
  }
}

class PayPalStrategy extends PaymentStrategy {
  constructor(email) {
    super()
    this.email = email
  }
  
  pay(amount) {
    console.log(`Paid ${amount} with PayPal account ${this.email}`)
  }
}

class CryptoStrategy extends PaymentStrategy {
  constructor(wallet) {
    super()
    this.wallet = wallet
  }
  
  pay(amount) {
    console.log(`Paid ${amount} with crypto wallet ${this.wallet}`)
  }
}

class ShoppingCart {
  constructor() {
    this.items = []
    this.paymentStrategy = null
  }
  
  addItem(item) {
    this.items.push(item)
  }
  
  setPaymentStrategy(strategy) {
    this.paymentStrategy = strategy
  }
  
  checkout() {
    const total = this.items.reduce((sum, item) => sum + item.price, 0)
    this.paymentStrategy.pay(total)
  }
}

const cart = new ShoppingCart()
cart.addItem({ name: 'Book', price: 10 })
cart.addItem({ name: 'Pen', price: 2 })

cart.setPaymentStrategy(new CreditCardStrategy('1234-5678'))
cart.checkout() // Paid 12 with credit card 1234-5678

cart.setPaymentStrategy(new PayPalStrategy('user@example.com'))
cart.checkout() // Paid 12 with PayPal account user@example.com
```

### Command

```javascript
// Инкапсуляция запроса как объекта

class Command {
  execute() {}
  undo() {}
}

class AddTextCommand extends Command {
  constructor(editor, text) {
    super()
    this.editor = editor
    this.text = text
  }
  
  execute() {
    this.editor.content += this.text
  }
  
  undo() {
    this.editor.content = this.editor.content.slice(0, -this.text.length)
  }
}

class DeleteTextCommand extends Command {
  constructor(editor, length) {
    super()
    this.editor = editor
    this.length = length
    this.deletedText = ''
  }
  
  execute() {
    this.deletedText = this.editor.content.slice(-this.length)
    this.editor.content = this.editor.content.slice(0, -this.length)
  }
  
  undo() {
    this.editor.content += this.deletedText
  }
}

class Editor {
  constructor() {
    this.content = ''
    this.history = []
    this.currentIndex = -1
  }
  
  executeCommand(command) {
    command.execute()
    
    // Удалить "будущие" команды при новом действии
    this.history = this.history.slice(0, this.currentIndex + 1)
    this.history.push(command)
    this.currentIndex++
  }
  
  undo() {
    if (this.currentIndex >= 0) {
      this.history[this.currentIndex].undo()
      this.currentIndex--
    }
  }
  
  redo() {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++
      this.history[this.currentIndex].execute()
    }
  }
}

const editor = new Editor()
editor.executeCommand(new AddTextCommand(editor, 'Hello '))
editor.executeCommand(new AddTextCommand(editor, 'World'))
console.log(editor.content) // Hello World

editor.undo()
console.log(editor.content) // Hello 

editor.redo()
console.log(editor.content) // Hello World
```

### State

```javascript
// Изменение поведения объекта при изменении состояния

class State {
  handle(context) {
    throw new Error('handle() must be implemented')
  }
}

class DraftState extends State {
  handle(context) {
    console.log('Document is in draft state')
    context.setState(new ReviewState())
  }
}

class ReviewState extends State {
  handle(context) {
    console.log('Document is under review')
    context.setState(new PublishedState())
  }
}

class PublishedState extends State {
  handle(context) {
    console.log('Document is published')
    context.setState(new ArchivedState())
  }
}

class ArchivedState extends State {
  handle(context) {
    console.log('Document is archived')
  }
}

class Document {
  constructor() {
    this.state = new DraftState()
  }
  
  setState(state) {
    this.state = state
  }
  
  publish() {
    this.state.handle(this)
  }
}

const doc = new Document()
doc.publish() // Document is in draft state
doc.publish() // Document is under review
doc.publish() // Document is published
doc.publish() // Document is archived
```

## Вопросы для собеседования

### 1. Когда использовать Singleton?

**Ответ:**
- Управление глобальным состоянием
- Подключение к БД
- Логгер
- Конфигурация

**Минусы**: усложняет тестирование, скрытые зависимости.

### 2. В чём разница между Factory и Builder?

**Ответ:**
- **Factory**: создаёт объект за один шаг, выбор типа
- **Builder**: пошаговое создание, сложная конфигурация

### 3. Объясните паттерн Observer

**Ответ:**
Один объект (subject) уведомляет множество подписчиков (observers) об изменениях. Используется в:
- Event listeners
- Reactive frameworks (Vue, React)
- Pub/Sub системы

### 4. Когда использовать Strategy?

**Ответ:**
Когда нужно выбирать алгоритм во время выполнения:
- Способы оплаты
- Сортировка данных
- Валидация форм
- Форматирование данных

### 5. Что такое Dependency Injection?

**Ответ:**
Передача зависимостей извне вместо создания внутри класса:

```javascript
// ❌ Плохо
class UserService {
  constructor() {
    this.api = new API() // Жёсткая зависимость
  }
}

// ✅ Хорошо
class UserService {
  constructor(api) {
    this.api = api // Инъекция зависимости
  }
}
```

Упрощает тестирование и переиспользование.
