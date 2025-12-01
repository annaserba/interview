---
title: "Типы данных и приведение типов"
description: "Разница между == и ===, явное и неявное преобразование типов"
category: "Основы JavaScript"
difficulty: "easy"
tags: ["types", "coercion", "comparison"]
order: 8
---

## Типы данных в JavaScript

### Примитивные типы (7 штук)

1. **Number** — числа (включая `NaN`, `Infinity`)
2. **String** — строки
3. **Boolean** — `true` / `false`
4. **Undefined** — неопределенное значение
5. **Null** — отсутствие значения
6. **Symbol** — уникальный идентификатор (ES6)
7. **BigInt** — большие целые числа (ES2020)

### Объектные типы

- **Object** — объекты, массивы, функции, даты и т.д.

```javascript
typeof 42;              // 'number'
typeof 'hello';         // 'string'
typeof true;            // 'boolean'
typeof undefined;       // 'undefined'
typeof Symbol('id');    // 'symbol'
typeof 123n;            // 'bigint'
typeof {};              // 'object'
typeof [];              // 'object' (!)
typeof null;            // 'object' (!) — историческая ошибка
typeof function() {};   // 'function'
```

## Разница между `==` и `===`

### `===` (строгое равенство)

Сравнивает **значение И тип** без приведения типов.

```javascript
5 === 5;        // true
5 === '5';      // false (разные типы)
null === null;  // true
undefined === undefined; // true
null === undefined;      // false
NaN === NaN;    // false (!)

// Для NaN используйте:
Number.isNaN(NaN); // true
Object.is(NaN, NaN); // true
```

### `==` (нестрогое равенство)

Сравнивает **значение** с приведением типов.

```javascript
5 == '5';       // true (строка приводится к числу)
0 == false;     // true
'' == false;    // true
null == undefined; // true (!)
[] == false;    // true
[] == '';       // true
```

**Правило:** Всегда используйте `===`, кроме проверки на `null`/`undefined`:

```javascript
// Проверка на null или undefined
if (value == null) {
  // value === null || value === undefined
}

// Лучше явно:
if (value === null || value === undefined) {
  // ...
}

// Или с nullish coalescing:
const result = value ?? 'default';
```

## Приведение типов

### К строке (String)

```javascript
String(123);        // '123'
String(true);       // 'true'
String(null);       // 'null'
String(undefined);  // 'undefined'
String([1, 2, 3]);  // '1,2,3'
String({});         // '[object Object]'

// Неявное приведение
123 + '';           // '123'
true + '';          // 'true'
```

### К числу (Number)

```javascript
Number('123');      // 123
Number('123abc');   // NaN
Number(true);       // 1
Number(false);      // 0
Number(null);       // 0
Number(undefined);  // NaN
Number('');         // 0
Number('  ');       // 0

// Неявное приведение
+'123';             // 123
'5' - 2;            // 3
'5' * 2;            // 10
'5' / 2;            // 2.5

// Особенности
parseInt('123px');  // 123
parseFloat('12.5em'); // 12.5
parseInt('0x10');   // 16 (hex)
```

### К булеву (Boolean)

```javascript
Boolean(1);         // true
Boolean(0);         // false
Boolean('hello');   // true
Boolean('');        // false
Boolean(null);      // false
Boolean(undefined); // false
Boolean({});        // true
Boolean([]);        // true

// Неявное приведение
!!value;            // Двойное отрицание
if (value) {}       // В условиях
```

### Falsy значения (приводятся к false)

Всего **8 falsy значений**:

```javascript
false
0
-0
0n (BigInt zero)
'' (пустая строка)
null
undefined
NaN
```

Все остальное — **truthy**:

```javascript
if ('0') {}        // true (непустая строка)
if ([]) {}         // true (пустой массив)
if ({}) {}         // true (пустой объект)
if (function(){}) {} // true
```

## Неявное приведение типов

### Сложение (+)

```javascript
1 + 2;              // 3 (число + число)
'1' + 2;            // '12' (строка + число → конкатенация)
1 + '2';            // '12'
1 + 2 + '3';        // '33' (слева направо: 3 + '3')
'1' + 2 + 3;        // '123' ('1' + 2 = '12', '12' + 3 = '123')

true + 1;           // 2
false + 1;          // 1
null + 1;           // 1
undefined + 1;      // NaN

[] + [];            // '' (пустая строка)
[] + {};            // '[object Object]'
{} + [];            // 0 (!) — {} интерпретируется как блок кода
```

### Другие операторы (-, *, /, %)

Приводят к числу:

```javascript
'5' - 2;            // 3
'5' * 2;            // 10
'10' / 2;           // 5
'10' % 3;           // 1

true - 1;           // 0
false - 1;          // -1
null - 1;           // -1
undefined - 1;      // NaN

'5' - 'abc';        // NaN
```

### Сравнение (<, >, <=, >=)

```javascript
'2' > 1;            // true (строка приводится к числу)
'01' == 1;          // true
'a' > 'b';          // false (лексикографическое сравнение)
'10' > '2';         // false (строки: '1' < '2')

null > 0;           // false
null == 0;          // false
null >= 0;          // true (!)

undefined > 0;      // false
undefined == 0;     // false
undefined >= 0;     // false
```

## Особые случаи

### NaN

```javascript
NaN === NaN;        // false
NaN == NaN;         // false
Object.is(NaN, NaN); // true
Number.isNaN(NaN);  // true
isNaN('hello');     // true (приводит к числу, потом проверяет)
Number.isNaN('hello'); // false (не приводит)
```

### Infinity

```javascript
1 / 0;              // Infinity
-1 / 0;             // -Infinity
Infinity + 1;       // Infinity
Infinity - Infinity; // NaN
Infinity / Infinity; // NaN
```

### null vs undefined

```javascript
null == undefined;  // true
null === undefined; // false

typeof null;        // 'object' (ошибка в спецификации)
typeof undefined;   // 'undefined'

null + 1;           // 1
undefined + 1;      // NaN
```

## Проверка типов

### Примитивы

```javascript
typeof value === 'string'
typeof value === 'number'
typeof value === 'boolean'
typeof value === 'undefined'
typeof value === 'symbol'
typeof value === 'bigint'
```

### Объекты

```javascript
Array.isArray([]);           // true
value instanceof Array;      // true
value instanceof Object;     // true

Object.prototype.toString.call([]); // '[object Array]'
Object.prototype.toString.call({}); // '[object Object]'
Object.prototype.toString.call(null); // '[object Null]'
```

### Проверка на число

```javascript
typeof value === 'number' && !isNaN(value)
Number.isFinite(value)      // Проверяет, что это конечное число
Number.isInteger(value)     // Проверяет, что это целое число
```

## Практические примеры

### Безопасное сложение

```javascript
function safeAdd(a, b) {
  const numA = Number(a);
  const numB = Number(b);
  
  if (isNaN(numA) || isNaN(numB)) {
    throw new Error('Невалидные числа');
  }
  
  return numA + numB;
}
```

### Проверка на пустое значение

```javascript
function isEmpty(value) {
  return value == null || value === '';
}

// Или более строго
function isEmpty(value) {
  return value === null || value === undefined || value === '';
}
```

### Приведение к булеву

```javascript
// ❌ Плохо
if (value == true) {}

// ✅ Хорошо
if (value) {}
if (Boolean(value)) {}
if (!!value) {}
```

## Советы для собеседования

1. **Всегда используйте `===`**, кроме проверки `value == null`
2. **Знайте все 8 falsy значений**
3. **`typeof null === 'object'`** — это баг, но он останется навсегда
4. **`NaN !== NaN`** — используйте `Number.isNaN()`
5. **Избегайте неявного приведения** — пишите явный код
6. **Для проверки массива** используйте `Array.isArray()`

## Шпаргалка по приведению

| Значение | String | Number | Boolean |
|----------|--------|--------|---------|
| `undefined` | `'undefined'` | `NaN` | `false` |
| `null` | `'null'` | `0` | `false` |
| `true` | `'true'` | `1` | `true` |
| `false` | `'false'` | `0` | `false` |
| `''` | `''` | `0` | `false` |
| `'123'` | `'123'` | `123` | `true` |
| `'abc'` | `'abc'` | `NaN` | `true` |
| `0` | `'0'` | `0` | `false` |
| `123` | `'123'` | `123` | `true` |
| `[]` | `''` | `0` | `true` |
| `[1,2]` | `'1,2'` | `NaN` | `true` |
| `{}` | `'[object Object]'` | `NaN` | `true` |
