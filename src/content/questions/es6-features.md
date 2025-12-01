---
title: "ES6+ возможности"
description: "let/const, стрелочные функции, деструктуризация, rest/spread, шаблонные строки"
category: "Основы JavaScript"
difficulty: "easy"
tags: ["es6", "modern-javascript", "syntax"]
order: 9
---

## let и const vs var

### var — старый способ

```javascript
var x = 1;
var x = 2; // Можно переопределить

if (true) {
  var y = 3;
}
console.log(y); // 3 — нет блочной области видимости

for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Выведет: 3, 3, 3 (одна переменная i)
```

### let — блочная область видимости

```javascript
let x = 1;
// let x = 2; // Error: уже объявлена

if (true) {
  let y = 3;
}
// console.log(y); // Error: y не определена

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Выведет: 0, 1, 2 (каждая итерация — своя переменная)
```

### const — константа

```javascript
const PI = 3.14;
// PI = 3.15; // Error: нельзя переприсвоить

// Но объекты и массивы можно изменять:
const user = { name: 'Анна' };
user.name = 'Борис'; // ✓ Работает
user.age = 25;       // ✓ Работает
// user = {};        // ✗ Error

const arr = [1, 2, 3];
arr.push(4);         // ✓ Работает
// arr = [];         // ✗ Error
```

### Temporal Dead Zone (TDZ)

```javascript
console.log(x); // undefined (var hoisting)
var x = 1;

// console.log(y); // ReferenceError: Cannot access before initialization
let y = 2;

// console.log(z); // ReferenceError
const z = 3;
```

## Стрелочные функции

### Синтаксис

```javascript
// Обычная функция
function sum(a, b) {
  return a + b;
}

// Стрелочная функция
const sum = (a, b) => {
  return a + b;
};

// Короткая форма (неявный return)
const sum = (a, b) => a + b;

// Один параметр — скобки необязательны
const double = x => x * 2;

// Без параметров
const greet = () => 'Hello';

// Возврат объекта — нужны скобки
const createUser = (name, age) => ({ name, age });
```

### Отличия от обычных функций

1. **Нет своего `this`** — берут из внешнего scope
2. **Нет `arguments`**
3. **Нельзя использовать как конструктор** (с `new`)
4. **Нет `prototype`**

```javascript
const obj = {
  name: 'Объект',
  regularFunc: function() {
    console.log(this.name); // 'Объект'
  },
  arrowFunc: () => {
    console.log(this.name); // undefined (this из глобального scope)
  }
};

// arguments
function regular() {
  console.log(arguments); // [1, 2, 3]
}
regular(1, 2, 3);

const arrow = () => {
  // console.log(arguments); // ReferenceError
};

// Используйте rest параметры:
const arrow = (...args) => {
  console.log(args); // [1, 2, 3]
};
```

## Деструктуризация

### Массивы

```javascript
const arr = [1, 2, 3, 4, 5];

// Базовая деструктуризация
const [first, second] = arr;
console.log(first, second); // 1, 2

// Пропуск элементов
const [, , third] = arr;
console.log(third); // 3

// Rest оператор
const [head, ...tail] = arr;
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// Значения по умолчанию
const [a, b, c = 0] = [1, 2];
console.log(c); // 0

// Обмен значений
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y); // 2, 1
```

### Объекты

```javascript
const user = {
  name: 'Анна',
  age: 25,
  city: 'Москва'
};

// Базовая деструктуризация
const { name, age } = user;
console.log(name, age); // 'Анна', 25

// Переименование
const { name: userName, age: userAge } = user;
console.log(userName); // 'Анна'

// Значения по умолчанию
const { name, country = 'Россия' } = user;
console.log(country); // 'Россия'

// Rest оператор
const { name, ...rest } = user;
console.log(rest); // { age: 25, city: 'Москва' }

// Вложенная деструктуризация
const user = {
  name: 'Анна',
  address: {
    city: 'Москва',
    street: 'Ленина'
  }
};

const { address: { city, street } } = user;
console.log(city, street); // 'Москва', 'Ленина'
```

### В параметрах функции

```javascript
// Массив
function sum([a, b]) {
  return a + b;
}
sum([1, 2]); // 3

// Объект
function greet({ name, age }) {
  console.log(`${name}, ${age} лет`);
}
greet({ name: 'Анна', age: 25 });

// Со значениями по умолчанию
function createUser({ name = 'Аноним', age = 0 } = {}) {
  return { name, age };
}
createUser(); // { name: 'Аноним', age: 0 }
```

## Rest и Spread операторы

### Rest (...) — собирает в массив

```javascript
// В параметрах функции
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4); // 10

// В деструктуризации
const [first, ...rest] = [1, 2, 3, 4];
console.log(rest); // [2, 3, 4]

const { name, ...otherProps } = { name: 'Анна', age: 25, city: 'Москва' };
console.log(otherProps); // { age: 25, city: 'Москва' }
```

### Spread (...) — раскладывает массив/объект

```javascript
// Массивы
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// Копирование массива
const original = [1, 2, 3];
const copy = [...original];

// Объекты
const user = { name: 'Анна', age: 25 };
const updatedUser = { ...user, age: 26, city: 'Москва' };
console.log(updatedUser); // { name: 'Анна', age: 26, city: 'Москва' }

// Копирование объекта
const userCopy = { ...user };

// В вызовах функций
const numbers = [1, 2, 3];
Math.max(...numbers); // 3
```

## Шаблонные строки

```javascript
const name = 'Анна';
const age = 25;

// Старый способ
const message = 'Привет, ' + name + '! Тебе ' + age + ' лет.';

// Шаблонные строки
const message = `Привет, ${name}! Тебе ${age} лет.`;

// Многострочные
const html = `
  <div>
    <h1>${name}</h1>
    <p>Возраст: ${age}</p>
  </div>
`;

// Выражения
const price = 100;
const message = `Цена со скидкой: ${price * 0.9} руб.`;

// Вызов функций
const message = `Результат: ${calculateSum(1, 2, 3)}`;
```

### Tagged templates

```javascript
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] ? `<mark>${values[i]}</mark>` : '');
  }, '');
}

const name = 'Анна';
const age = 25;
const message = highlight`Привет, ${name}! Тебе ${age} лет.`;
// 'Привет, <mark>Анна</mark>! Тебе <mark>25</mark> лет.'
```

## Другие ES6+ возможности

### Default параметры

```javascript
function greet(name = 'Гость', greeting = 'Привет') {
  return `${greeting}, ${name}!`;
}

greet(); // 'Привет, Гость!'
greet('Анна'); // 'Привет, Анна!'
greet('Анна', 'Здравствуй'); // 'Здравствуй, Анна!'
```

### Сокращенная запись свойств

```javascript
const name = 'Анна';
const age = 25;

// Старый способ
const user = { name: name, age: age };

// ES6
const user = { name, age };

// Методы
const obj = {
  // Старый способ
  greet: function() {
    return 'Hello';
  },
  
  // ES6
  greet() {
    return 'Hello';
  }
};
```

### Вычисляемые имена свойств

```javascript
const propName = 'age';

const user = {
  name: 'Анна',
  [propName]: 25,
  ['is' + 'Admin']: true
};

console.log(user); // { name: 'Анна', age: 25, isAdmin: true }
```

### Optional chaining (?.)

```javascript
const user = {
  name: 'Анна',
  address: {
    city: 'Москва'
  }
};

// Старый способ
const street = user && user.address && user.address.street;

// Optional chaining
const street = user?.address?.street; // undefined

// С методами
user.greet?.(); // Вызовет, если метод существует

// С массивами
const firstItem = arr?.[0];
```

### Nullish coalescing (??)

```javascript
// || возвращает первое truthy значение
const value = 0 || 'default'; // 'default'
const value = '' || 'default'; // 'default'

// ?? возвращает первое не-null/undefined значение
const value = 0 ?? 'default'; // 0
const value = '' ?? 'default'; // ''
const value = null ?? 'default'; // 'default'
const value = undefined ?? 'default'; // 'default'
```

## Практические примеры

### Слияние объектов

```javascript
const defaults = { theme: 'light', lang: 'ru' };
const userSettings = { lang: 'en' };

const settings = { ...defaults, ...userSettings };
// { theme: 'light', lang: 'en' }
```

### Удаление свойства

```javascript
const user = { name: 'Анна', age: 25, password: '123' };

// Создаем новый объект без password
const { password, ...publicUser } = user;
console.log(publicUser); // { name: 'Анна', age: 25 }
```

### Условное добавление свойств

```javascript
const includeAge = true;

const user = {
  name: 'Анна',
  ...(includeAge && { age: 25 })
};
```

## Советы для собеседования

1. **Всегда используйте `const`** по умолчанию, `let` — если нужно переприсвоение
2. **Стрелочные функции не имеют `this`** — используйте для коллбэков
3. **Деструктуризация делает код чище** — используйте в параметрах функций
4. **`...rest` собирает, `...spread` раскладывает**
5. **`??` лучше `||`** для значений по умолчанию
6. **`?.` предотвращает ошибки** при обращении к несуществующим свойствам
