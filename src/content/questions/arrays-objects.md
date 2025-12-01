---
title: "Работа с массивами и объектами"
description: "Группировка, фильтрация, сортировка и трансформация данных"
category: "Лайв-кодинг"
difficulty: "medium"
tags: ["arrays", "objects", "data-manipulation"]
order: 2
---

## Типичные задачи

### Группировка данных

```javascript
// Задача: Сгруппировать пользователей по городу
const users = [
  { id: 1, name: 'Анна', city: 'Москва', age: 25 },
  { id: 2, name: 'Борис', city: 'Санкт-Петербург', age: 30 },
  { id: 3, name: 'Алексей', city: 'Москва', age: 28 }
];

function groupByCity(users) {
  return users.reduce((acc, user) => {
    if (!acc[user.city]) {
      acc[user.city] = [];
    }
    acc[user.city].push(user);
    return acc;
  }, {});
}

console.log(groupByCity(users));
// { 'Москва': [...], 'Санкт-Петербург': [...] }
```

### Фильтрация и вычисления

```javascript
// Задача: Найти средний возраст пользователей из Москвы
function getAverageAge(users, city) {
  const cityUsers = users.filter(user => user.city === city);
  
  if (cityUsers.length === 0) return 0;
  
  const totalAge = cityUsers.reduce((sum, user) => sum + user.age, 0);
  return totalAge / cityUsers.length;
}

console.log(getAverageAge(users, 'Москва')); // 26.5
```

### Группировка по первой букве

```javascript
function groupByFirstLetter(users) {
  return users.reduce((acc, user) => {
    const firstLetter = user.name[0].toUpperCase();
    if (!acc[firstLetter]) {
      acc[firstLetter] = [];
    }
    acc[firstLetter].push(user);
    return acc;
  }, {});
}

console.log(groupByFirstLetter(users));
// { 'А': [...], 'Б': [...] }
```

### Трансформация данных

```javascript
// Задача: Преобразовать массив объектов в нужный формат
function transformUsers(users) {
  return users.map(user => ({
    fullName: user.name,
    location: user.city,
    isAdult: user.age >= 18
  }));
}
```

## Комплексная задача

```javascript
// Найти средний возраст пользователей из Москвы,
// сгруппировать их по первой букве имени
function complexTransform(users, city) {
  // Фильтруем по городу
  const cityUsers = users.filter(user => user.city === city);
  
  // Вычисляем средний возраст
  const averageAge = cityUsers.length > 0
    ? cityUsers.reduce((sum, user) => sum + user.age, 0) / cityUsers.length
    : 0;
  
  // Группируем по первой букве
  const grouped = cityUsers.reduce((acc, user) => {
    const firstLetter = user.name[0].toUpperCase();
    if (!acc[firstLetter]) {
      acc[firstLetter] = [];
    }
    acc[firstLetter].push({
      name: user.name,
      age: user.age
    });
    return acc;
  }, {});
  
  return {
    averageAge,
    grouped,
    totalUsers: cityUsers.length
  };
}
```

## Полезные методы массивов

- **`map()`** — трансформация элементов
- **`filter()`** — фильтрация по условию
- **`reduce()`** — агрегация данных
- **`find()`** — поиск первого элемента
- **`some()`** / **`every()`** — проверка условий
- **`sort()`** — сортировка (мутирует массив!)
- **`slice()`** — копирование части массива
- **`concat()`** — объединение массивов

## Edge cases

```javascript
function safeGrouping(users, city) {
  // Проверка на null/undefined
  if (!users || !Array.isArray(users)) {
    return { averageAge: 0, grouped: {}, totalUsers: 0 };
  }
  
  // Проверка на пустой массив
  if (users.length === 0) {
    return { averageAge: 0, grouped: {}, totalUsers: 0 };
  }
  
  // Основная логика...
}
```
