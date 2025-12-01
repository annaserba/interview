---
title: "TypeScript основы"
description: "Базовые типы, интерфейсы, Generics"
category: "TypeScript"
difficulty: "medium"
tags: ["typescript", "types", "interfaces", "generics"]
order: 11
---

## Зачем TypeScript?

- **Статическая типизация** — ошибки на этапе разработки
- **Автодополнение** — лучший DX (Developer Experience)
- **Рефакторинг** — безопасное переименование
- **Документация** — типы как документация кода

## Базовые типы

```typescript
// Примитивы
let name: string = 'Анна';
let age: number = 25;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Массивы
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ['a', 'b', 'c'];

// Кортежи (tuple)
let person: [string, number] = ['Анна', 25];

// Any (избегайте!)
let anything: any = 'может быть чем угодно';
anything = 123;
anything = true;

// Unknown (безопаснее any)
let value: unknown = 'hello';
// value.toUpperCase(); // Error!
if (typeof value === 'string') {
  value.toUpperCase(); // ✓ OK
}

// Void (для функций без возврата)
function log(message: string): void {
  console.log(message);
}

// Never (функция никогда не вернет значение)
function throwError(message: string): never {
  throw new Error(message);
}

// Object
let user: object = { name: 'Анна' };
```

## Объединения и пересечения

### Union Types (|)

```typescript
// Может быть string ИЛИ number
let id: string | number;
id = '123'; // ✓
id = 123;   // ✓
// id = true; // ✗ Error

// С литералами
type Status = 'pending' | 'success' | 'error';
let status: Status = 'pending';
// status = 'invalid'; // ✗ Error

// Функция с union
function formatId(id: string | number): string {
  // Type guard
  if (typeof id === 'string') {
    return id.toUpperCase();
  }
  return id.toString();
}
```

### Intersection Types (&)

```typescript
// Объединяет несколько типов
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: number;
  department: string;
};

type EmployeePerson = Person & Employee;

const employee: EmployeePerson = {
  name: 'Анна',
  age: 25,
  employeeId: 123,
  department: 'IT'
};
```

## Интерфейсы

```typescript
// Базовый интерфейс
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Опциональное свойство
  readonly createdAt: Date; // Только для чтения
}

const user: User = {
  id: 1,
  name: 'Анна',
  email: 'anna@example.com',
  createdAt: new Date()
};

// user.createdAt = new Date(); // ✗ Error: readonly

// Расширение интерфейсов
interface Admin extends User {
  permissions: string[];
  role: 'admin' | 'superadmin';
}

const admin: Admin = {
  id: 1,
  name: 'Борис',
  email: 'boris@example.com',
  createdAt: new Date(),
  permissions: ['read', 'write'],
  role: 'admin'
};

// Методы в интерфейсах
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b
};

// Индексные сигнатуры
interface Dictionary {
  [key: string]: string;
}

const dict: Dictionary = {
  hello: 'привет',
  world: 'мир'
};
```

## Type vs Interface

```typescript
// Type — может описывать примитивы, union, tuple
type ID = string | number;
type Point = [number, number];

// Interface — только объекты, можно расширять
interface User {
  name: string;
}

interface User {
  age: number; // Добавляет свойство к существующему
}

// Когда использовать что?
// - Interface: для объектов, классов, API
// - Type: для union, tuple, сложных типов
```

## Функции

```typescript
// Типизация параметров и возврата
function add(a: number, b: number): number {
  return a + b;
}

// Опциональные параметры
function greet(name: string, greeting?: string): string {
  return `${greeting || 'Привет'}, ${name}!`;
}

// Параметры по умолчанию
function createUser(name: string, age: number = 18): User {
  return { name, age };
}

// Rest параметры
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

// Перегрузка функций
function format(value: string): string;
function format(value: number): string;
function format(value: string | number): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

// Типизация стрелочных функций
const multiply = (a: number, b: number): number => a * b;

// Функция как тип
type MathOperation = (a: number, b: number) => number;

const divide: MathOperation = (a, b) => a / b;
```

## Generics (Обобщения)

### Базовое использование

```typescript
// Функция с generic
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(123);
const str = identity<string>('hello');
const auto = identity('auto'); // TypeScript выведет тип сам

// Generic массив
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const first = getFirstElement([1, 2, 3]); // number | undefined
const firstStr = getFirstElement(['a', 'b']); // string | undefined
```

### Generic интерфейсы

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
}

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: 'Анна' },
  status: 200,
  message: 'Success'
};

const usersResponse: ApiResponse<User[]> = {
  data: [
    { id: 1, name: 'Анна' },
    { id: 2, name: 'Борис' }
  ],
  status: 200,
  message: 'Success'
};
```

### Generic классы

```typescript
class Storage<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  get(index: number): T | undefined {
    return this.items[index];
  }

  getAll(): T[] {
    return this.items;
  }
}

const numberStorage = new Storage<number>();
numberStorage.add(1);
numberStorage.add(2);

const stringStorage = new Storage<string>();
stringStorage.add('hello');
```

### Ограничения Generic (Constraints)

```typescript
// T должен иметь свойство length
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);
}

logLength('hello'); // ✓
logLength([1, 2, 3]); // ✓
// logLength(123); // ✗ Error

// Extends с keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Анна', age: 25 };
const name = getProperty(user, 'name'); // string
const age = getProperty(user, 'age');   // number
// getProperty(user, 'invalid'); // ✗ Error
```

## Utility Types

TypeScript предоставляет встроенные типы-утилиты:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Partial — все свойства опциональны
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

// Required — все свойства обязательны
type RequiredUser = Required<PartialUser>;

// Readonly — все свойства только для чтения
type ReadonlyUser = Readonly<User>;

// Pick — выбрать определенные свойства
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: number; name: string; }

// Omit — исключить определенные свойства
type UserWithoutEmail = Omit<User, 'email'>;
// { id: number; name: string; age: number; }

// Record — создать объект с ключами и значениями
type UserRoles = Record<string, User>;
// { [key: string]: User }

// ReturnType — тип возвращаемого значения функции
function getUser() {
  return { id: 1, name: 'Анна' };
}
type User = ReturnType<typeof getUser>;
// { id: number; name: string; }
```

## Практические примеры

### Типизация API запросов

```typescript
interface ApiResponse<T> {
  data: T;
  error?: string;
  status: number;
}

async function fetchData<T>(url: string): Promise<ApiResponse<T>> {
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    return {
      data,
      status: response.status
    };
  } catch (error) {
    return {
      data: {} as T,
      error: error.message,
      status: 500
    };
  }
}

// Использование
interface User {
  id: number;
  name: string;
}

const result = await fetchData<User[]>('/api/users');
if (!result.error) {
  result.data.forEach(user => console.log(user.name));
}
```

### Типизация React компонентов

```typescript
import React from 'react';

interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ 
  text, 
  onClick, 
  variant = 'primary',
  disabled = false 
}) => {
  return (
    <button 
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {text}
    </button>
  );
};

// Использование
<Button text="Нажми" onClick={() => console.log('Clicked')} />
```

### Типизация Redux

```typescript
// Actions
interface IncrementAction {
  type: 'INCREMENT';
  payload: number;
}

interface DecrementAction {
  type: 'DECREMENT';
  payload: number;
}

type CounterAction = IncrementAction | DecrementAction;

// Reducer
interface CounterState {
  count: number;
}

function counterReducer(
  state: CounterState = { count: 0 },
  action: CounterAction
): CounterState {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + action.payload };
    case 'DECREMENT':
      return { count: state.count - action.payload };
    default:
      return state;
  }
}
```

## Советы для собеседования

1. **Избегайте `any`** — используйте `unknown` или конкретные типы
2. **Используйте интерфейсы** для объектов и API
3. **Generic делают код переиспользуемым** — используйте для функций работы с данными
4. **Utility types экономят время** — знайте `Partial`, `Pick`, `Omit`
5. **Type guards** помогают работать с union types
6. **`keyof` и `typeof`** — мощные инструменты для работы с типами

## Конфигурация tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

**Важные флаги:**
- `strict: true` — включает все строгие проверки
- `noImplicitAny` — запрещает неявный `any`
- `strictNullChecks` — строгая проверка `null`/`undefined`
