---
title: "ООП и SOLID принципы"
description: "Объектно-ориентированное программирование и принципы проектирования"
category: "Architecture"
difficulty: "hard"
tags: ["oop", "solid", "design-principles", "architecture"]
order: 33
---

## Основы ООП

### Четыре столпа ООП

1. **Инкапсуляция** — сокрытие внутренней реализации
2. **Наследование** — переиспользование кода через иерархию
3. **Полиморфизм** — один интерфейс, разные реализации
4. **Абстракция** — выделение главного, скрытие деталей

## Инкапсуляция

Объединение данных и методов, ограничение прямого доступа к внутреннему состоянию.

### Приватные поля (ES2022)

```javascript
class BankAccount {
  #balance = 0  // Приватное поле
  #pin
  
  constructor(initialBalance, pin) {
    this.#balance = initialBalance
    this.#pin = pin
  }
  
  // Публичные методы для доступа
  getBalance(pin) {
    if (pin !== this.#pin) {
      throw new Error('Invalid PIN')
    }
    return this.#balance
  }
  
  deposit(amount) {
    if (amount <= 0) {
      throw new Error('Amount must be positive')
    }
    this.#balance += amount
  }
  
  withdraw(amount, pin) {
    if (pin !== this.#pin) {
      throw new Error('Invalid PIN')
    }
    if (amount > this.#balance) {
      throw new Error('Insufficient funds')
    }
    this.#balance -= amount
  }
  
  // Приватный метод
  #validatePin(pin) {
    return pin === this.#pin
  }
}

const account = new BankAccount(1000, '1234')
console.log(account.getBalance('1234')) // 1000
// console.log(account.#balance) // SyntaxError
```

### Геттеры и сеттеры

```javascript
class User {
  #email
  #age
  
  constructor(email, age) {
    this.email = email  // Использует setter
    this.age = age
  }
  
  get email() {
    return this.#email
  }
  
  set email(value) {
    if (!value.includes('@')) {
      throw new Error('Invalid email')
    }
    this.#email = value.toLowerCase()
  }
  
  get age() {
    return this.#age
  }
  
  set age(value) {
    if (value < 0 || value > 150) {
      throw new Error('Invalid age')
    }
    this.#age = value
  }
  
  get isAdult() {
    return this.#age >= 18
  }
}

const user = new User('John@Example.com', 25)
console.log(user.email)    // john@example.com
console.log(user.isAdult)  // true
```

## Наследование

Создание новых классов на основе существующих.

### Классическое наследование

```javascript
class Animal {
  constructor(name) {
    this.name = name
  }
  
  speak() {
    console.log(`${this.name} makes a sound`)
  }
  
  move() {
    console.log(`${this.name} moves`)
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name)  // Вызов конструктора родителя
    this.breed = breed
  }
  
  // Переопределение метода
  speak() {
    console.log(`${this.name} barks`)
  }
  
  // Новый метод
  fetch() {
    console.log(`${this.name} fetches the ball`)
  }
}

class Cat extends Animal {
  speak() {
    console.log(`${this.name} meows`)
  }
  
  climb() {
    console.log(`${this.name} climbs a tree`)
  }
}

const dog = new Dog('Rex', 'Labrador')
dog.speak()  // Rex barks
dog.move()   // Rex moves (унаследовано)
dog.fetch()  // Rex fetches the ball

const cat = new Cat('Whiskers')
cat.speak()  // Whiskers meows
cat.climb()  // Whiskers climbs a tree
```

### Композиция vs Наследование

```javascript
// ❌ Плохо: глубокая иерархия наследования
class Vehicle {}
class LandVehicle extends Vehicle {}
class Car extends LandVehicle {}
class ElectricCar extends Car {}

// ✅ Хорошо: композиция
class Engine {
  start() {
    console.log('Engine started')
  }
}

class ElectricMotor {
  start() {
    console.log('Electric motor started')
  }
}

class GPS {
  navigate(destination) {
    console.log(`Navigating to ${destination}`)
  }
}

class Car {
  constructor(engine) {
    this.engine = engine
    this.gps = new GPS()
  }
  
  start() {
    this.engine.start()
  }
  
  navigate(destination) {
    this.gps.navigate(destination)
  }
}

const gasCar = new Car(new Engine())
const electricCar = new Car(new ElectricMotor())

gasCar.start()      // Engine started
electricCar.start() // Electric motor started
```

## Полиморфизм

Один интерфейс, множество реализаций.

### Полиморфизм через наследование

```javascript
class Shape {
  area() {
    throw new Error('Method must be implemented')
  }
  
  perimeter() {
    throw new Error('Method must be implemented')
  }
}

class Circle extends Shape {
  constructor(radius) {
    super()
    this.radius = radius
  }
  
  area() {
    return Math.PI * this.radius ** 2
  }
  
  perimeter() {
    return 2 * Math.PI * this.radius
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super()
    this.width = width
    this.height = height
  }
  
  area() {
    return this.width * this.height
  }
  
  perimeter() {
    return 2 * (this.width + this.height)
  }
}

// Полиморфное использование
function printShapeInfo(shape) {
  console.log(`Area: ${shape.area()}`)
  console.log(`Perimeter: ${shape.perimeter()}`)
}

const circle = new Circle(5)
const rectangle = new Rectangle(4, 6)

printShapeInfo(circle)     // Работает с Circle
printShapeInfo(rectangle)  // Работает с Rectangle
```

### Duck Typing (Утиная типизация)

```javascript
// "Если это выглядит как утка и крякает как утка, то это утка"

class Duck {
  quack() {
    console.log('Quack!')
  }
  
  fly() {
    console.log('Duck flies')
  }
}

class Person {
  quack() {
    console.log('Person imitates: Quack!')
  }
  
  fly() {
    console.log('Person jumps')
  }
}

class Robot {
  quack() {
    console.log('Robot says: Quack!')
  }
  
  fly() {
    console.log('Robot activates propellers')
  }
}

// Функция работает с любым объектом, у которого есть методы quack и fly
function makeDuckDoThings(duck) {
  duck.quack()
  duck.fly()
}

makeDuckDoThings(new Duck())    // Quack! Duck flies
makeDuckDoThings(new Person())  // Person imitates: Quack! Person jumps
makeDuckDoThings(new Robot())   // Robot says: Quack! Robot activates propellers
```

## Абстракция

Выделение важных характеристик, скрытие сложности.

```javascript
// Абстрактный класс (концептуально, JS не поддерживает напрямую)
class PaymentProcessor {
  constructor() {
    if (new.target === PaymentProcessor) {
      throw new Error('Cannot instantiate abstract class')
    }
  }
  
  // Абстрактный метод
  processPayment(amount) {
    throw new Error('Method must be implemented')
  }
  
  // Конкретный метод
  validateAmount(amount) {
    if (amount <= 0) {
      throw new Error('Amount must be positive')
    }
    return true
  }
}

class CreditCardProcessor extends PaymentProcessor {
  processPayment(amount, cardNumber) {
    this.validateAmount(amount)
    console.log(`Processing $${amount} via credit card ${cardNumber}`)
    // Логика обработки кредитной карты
  }
}

class PayPalProcessor extends PaymentProcessor {
  processPayment(amount, email) {
    this.validateAmount(amount)
    console.log(`Processing $${amount} via PayPal for ${email}`)
    // Логика обработки PayPal
  }
}

class CryptoProcessor extends PaymentProcessor {
  processPayment(amount, wallet) {
    this.validateAmount(amount)
    console.log(`Processing $${amount} via crypto to wallet ${wallet}`)
    // Логика обработки криптовалюты
  }
}

// Использование
function checkout(processor, amount, details) {
  processor.processPayment(amount, details)
}

checkout(new CreditCardProcessor(), 100, '1234-5678-9012-3456')
checkout(new PayPalProcessor(), 50, 'user@example.com')
checkout(new CryptoProcessor(), 200, '0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
```

## SOLID принципы

### S - Single Responsibility Principle (Принцип единственной ответственности)

Класс должен иметь только одну причину для изменения.

```javascript
// ❌ Плохо: класс делает слишком много
class User {
  constructor(name, email) {
    this.name = name
    this.email = email
  }
  
  save() {
    // Сохранение в БД
    console.log('Saving to database...')
  }
  
  sendEmail(message) {
    // Отправка email
    console.log(`Sending email to ${this.email}: ${message}`)
  }
  
  generateReport() {
    // Генерация отчёта
    console.log('Generating report...')
  }
}

// ✅ Хорошо: разделение ответственности
class User {
  constructor(name, email) {
    this.name = name
    this.email = email
  }
}

class UserRepository {
  save(user) {
    console.log(`Saving ${user.name} to database...`)
  }
  
  findById(id) {
    console.log(`Finding user ${id}`)
  }
}

class EmailService {
  send(email, message) {
    console.log(`Sending email to ${email}: ${message}`)
  }
}

class ReportGenerator {
  generate(user) {
    console.log(`Generating report for ${user.name}`)
  }
}

// Использование
const user = new User('John', 'john@example.com')
const userRepo = new UserRepository()
const emailService = new EmailService()
const reportGen = new ReportGenerator()

userRepo.save(user)
emailService.send(user.email, 'Welcome!')
reportGen.generate(user)
```

### O - Open/Closed Principle (Принцип открытости/закрытости)

Открыт для расширения, закрыт для модификации.

```javascript
// ❌ Плохо: нужно модифицировать класс для добавления новых типов
class AreaCalculator {
  calculate(shape) {
    if (shape.type === 'circle') {
      return Math.PI * shape.radius ** 2
    } else if (shape.type === 'rectangle') {
      return shape.width * shape.height
    } else if (shape.type === 'triangle') {
      return (shape.base * shape.height) / 2
    }
    // Нужно добавлять новые if для каждой фигуры
  }
}

// ✅ Хорошо: расширяем через наследование/композицию
class Shape {
  area() {
    throw new Error('Method must be implemented')
  }
}

class Circle extends Shape {
  constructor(radius) {
    super()
    this.radius = radius
  }
  
  area() {
    return Math.PI * this.radius ** 2
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super()
    this.width = width
    this.height = height
  }
  
  area() {
    return this.width * this.height
  }
}

class Triangle extends Shape {
  constructor(base, height) {
    super()
    this.base = base
    this.height = height
  }
  
  area() {
    return (this.base * this.height) / 2
  }
}

// Калькулятор не нужно менять при добавлении новых фигур
class AreaCalculator {
  calculate(shape) {
    return shape.area()
  }
}

const calculator = new AreaCalculator()
console.log(calculator.calculate(new Circle(5)))
console.log(calculator.calculate(new Rectangle(4, 6)))
console.log(calculator.calculate(new Triangle(3, 4)))
```

### L - Liskov Substitution Principle (Принцип подстановки Барбары Лисков)

Объекты подклассов должны заменять объекты базового класса без изменения поведения программы.

```javascript
// ❌ Плохо: нарушение LSP
class Bird {
  fly() {
    console.log('Flying')
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error('Penguins cannot fly!')
  }
}

function makeBirdFly(bird) {
  bird.fly()  // Упадёт для Penguin
}

// ✅ Хорошо: правильная иерархия
class Bird {
  eat() {
    console.log('Eating')
  }
}

class FlyingBird extends Bird {
  fly() {
    console.log('Flying')
  }
}

class SwimmingBird extends Bird {
  swim() {
    console.log('Swimming')
  }
}

class Sparrow extends FlyingBird {}
class Penguin extends SwimmingBird {}

function makeFlyingBirdFly(bird) {
  bird.fly()  // Работает только с FlyingBird
}

makeFlyingBirdFly(new Sparrow())  // OK
// makeFlyingBirdFly(new Penguin())  // Ошибка компиляции (TypeScript)
```

### I - Interface Segregation Principle (Принцип разделения интерфейса)

Клиенты не должны зависеть от интерфейсов, которые они не используют.

```javascript
// ❌ Плохо: толстый интерфейс
class Worker {
  work() {
    throw new Error('Must implement')
  }
  
  eat() {
    throw new Error('Must implement')
  }
  
  sleep() {
    throw new Error('Must implement')
  }
}

class HumanWorker extends Worker {
  work() {
    console.log('Working')
  }
  
  eat() {
    console.log('Eating')
  }
  
  sleep() {
    console.log('Sleeping')
  }
}

class RobotWorker extends Worker {
  work() {
    console.log('Working')
  }
  
  eat() {
    // Роботы не едят!
    throw new Error('Robots do not eat')
  }
  
  sleep() {
    // Роботы не спят!
    throw new Error('Robots do not sleep')
  }
}

// ✅ Хорошо: разделённые интерфейсы
class Workable {
  work() {
    throw new Error('Must implement')
  }
}

class Eatable {
  eat() {
    throw new Error('Must implement')
  }
}

class Sleepable {
  sleep() {
    throw new Error('Must implement')
  }
}

class HumanWorker {
  constructor() {
    this.workable = new Workable()
    this.eatable = new Eatable()
    this.sleepable = new Sleepable()
  }
  
  work() {
    console.log('Human working')
  }
  
  eat() {
    console.log('Human eating')
  }
  
  sleep() {
    console.log('Human sleeping')
  }
}

class RobotWorker {
  work() {
    console.log('Robot working')
  }
  
  // Нет методов eat и sleep
}
```

### D - Dependency Inversion Principle (Принцип инверсии зависимостей)

Зависимость от абстракций, а не от конкретных реализаций.

```javascript
// ❌ Плохо: зависимость от конкретной реализации
class MySQLDatabase {
  save(data) {
    console.log('Saving to MySQL:', data)
  }
}

class UserService {
  constructor() {
    this.database = new MySQLDatabase()  // Жёсткая зависимость
  }
  
  saveUser(user) {
    this.database.save(user)
  }
}

// ✅ Хорошо: зависимость от абстракции
class Database {
  save(data) {
    throw new Error('Must implement')
  }
}

class MySQLDatabase extends Database {
  save(data) {
    console.log('Saving to MySQL:', data)
  }
}

class MongoDatabase extends Database {
  save(data) {
    console.log('Saving to MongoDB:', data)
  }
}

class PostgreSQLDatabase extends Database {
  save(data) {
    console.log('Saving to PostgreSQL:', data)
  }
}

class UserService {
  constructor(database) {
    this.database = database  // Инъекция зависимости
  }
  
  saveUser(user) {
    this.database.save(user)
  }
}

// Использование
const mysqlService = new UserService(new MySQLDatabase())
const mongoService = new UserService(new MongoDatabase())
const postgresService = new UserService(new PostgreSQLDatabase())

mysqlService.saveUser({ name: 'John' })
mongoService.saveUser({ name: 'Jane' })
postgresService.saveUser({ name: 'Bob' })
```

## Дополнительные принципы

### DRY (Don't Repeat Yourself)

```javascript
// ❌ Плохо: дублирование кода
function calculateCircleArea(radius) {
  return 3.14159 * radius * radius
}

function calculateCirclePerimeter(radius) {
  return 2 * 3.14159 * radius
}

function calculateSphereVolume(radius) {
  return (4/3) * 3.14159 * radius * radius * radius
}

// ✅ Хорошо: переиспользование
const PI = 3.14159

function calculateCircleArea(radius) {
  return PI * radius ** 2
}

function calculateCirclePerimeter(radius) {
  return 2 * PI * radius
}

function calculateSphereVolume(radius) {
  return (4/3) * PI * radius ** 3
}
```

### KISS (Keep It Simple, Stupid)

```javascript
// ❌ Плохо: излишне сложно
function isEven(number) {
  return number % 2 === 0 ? true : false
}

// ✅ Хорошо: просто
function isEven(number) {
  return number % 2 === 0
}
```

### YAGNI (You Aren't Gonna Need It)

```javascript
// ❌ Плохо: функциональность "на будущее"
class User {
  constructor(name) {
    this.name = name
    this.friends = []
    this.posts = []
    this.comments = []
    this.likes = []
    this.followers = []
    this.following = []
    // ... ещё 20 полей, которые может быть понадобятся
  }
}

// ✅ Хорошо: только то, что нужно сейчас
class User {
  constructor(name) {
    this.name = name
  }
}
```

## Вопросы для собеседования

### 1. Объясните четыре столпа ООП

**Ответ:**
- **Инкапсуляция**: сокрытие данных, доступ через методы
- **Наследование**: переиспользование кода через иерархию
- **Полиморфизм**: один интерфейс, разные реализации
- **Абстракция**: выделение главного, скрытие деталей

### 2. Композиция vs Наследование

**Ответ:**
- **Наследование**: "is-a" отношение (Dog is an Animal)
- **Композиция**: "has-a" отношение (Car has an Engine)

Предпочитайте композицию — она более гибкая и избегает проблем глубокой иерархии.

### 3. Что такое SOLID?

**Ответ:**
- **S**: Single Responsibility — одна ответственность
- **O**: Open/Closed — открыт для расширения, закрыт для модификации
- **L**: Liskov Substitution — подклассы заменяют базовый класс
- **I**: Interface Segregation — не зависеть от неиспользуемых методов
- **D**: Dependency Inversion — зависимость от абстракций

### 4. Что такое Duck Typing?

**Ответ:**
"Если выглядит как утка и крякает как утка, то это утка". JavaScript проверяет наличие методов/свойств во время выполнения, а не типы.

### 5. Зачем нужна инкапсуляция?

**Ответ:**
- Защита данных от некорректного использования
- Скрытие внутренней реализации
- Упрощение изменения кода
- Контроль доступа через методы

### 6. Что такое Dependency Injection?

**Ответ:**
Передача зависимостей извне вместо создания внутри класса. Упрощает тестирование и делает код более гибким.

```javascript
// DI
class Service {
  constructor(database) {
    this.db = database
  }
}
```

### 7. Когда нарушается принцип Liskov?

**Ответ:**
Когда подкласс:
- Выбрасывает исключения, которых нет в базовом классе
- Требует более строгие предусловия
- Даёт более слабые постусловия
- Изменяет поведение базового класса

### 8. В чём разница между абстракцией и инкапсуляцией?

**Ответ:**
- **Абстракция**: что делает объект (интерфейс)
- **Инкапсуляция**: как это реализовано (скрытие деталей)

Абстракция — дизайн, инкапсуляция — реализация.

### 9. Что такое принцип DRY?

**Ответ:**
Don't Repeat Yourself — избегайте дублирования кода. Переиспользуйте через функции, классы, модули.

### 10. Как применить SOLID в JavaScript?

**Ответ:**
- **S**: разделяйте классы по ответственности
- **O**: используйте наследование/композицию для расширения
- **L**: правильная иерархия классов
- **I**: маленькие интерфейсы (duck typing)
- **D**: dependency injection через конструктор
