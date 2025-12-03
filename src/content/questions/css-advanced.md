---
title: "CSS: Продвинутые техники"
description: "Современные возможности CSS: Grid, Flexbox, анимации, кастомные свойства"
category: "CSS"
difficulty: "medium"
tags: ["css", "grid", "flexbox", "animations", "custom-properties"]
order: 22
---

## CSS Grid

Мощная система двумерных раскладок.

### Базовая сетка

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  gap: 20px;
}

.header {
  grid-column: 1 / -1;
}
```

### Repeat и minmax

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
```

## Flexbox

```css
.container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

.item {
  flex: 1;
}
```

## CSS Custom Properties (Переменные)

```css
:root {
  --primary-color: #3498db;
  --spacing: 16px;
  --border-radius: 8px;
}

.button {
  background: var(--primary-color);
  padding: var(--spacing);
  border-radius: var(--border-radius);
}

/* Темная тема */
[data-theme="dark"] {
  --primary-color: #2980b9;
  --bg-color: #1a1a1a;
}
```

## Анимации и Transitions

### Transitions

```css
.button {
  transition: all 0.3s ease;
}

.button:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}
```

### Animations

```css
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.element {
  animation: fadeIn 0.5s ease-out;
}
```

## Container Queries

```css
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

## Вопросы для собеседования

### 1. Когда использовать Grid vs Flexbox?

**Ответ:**
- **Grid**: двумерные раскладки (строки и колонки)
- **Flexbox**: одномерные раскладки (строка или колонка)
- Можно комбинировать

### 2. Что такое CSS Custom Properties?

**Ответ:**
Переменные в CSS с каскадом и возможностью изменения через JavaScript.

### 3. В чём разница между transition и animation?

**Ответ:**
- **Transition**: анимация между двумя состояниями (hover, focus)
- **Animation**: сложная анимация с keyframes, может повторяться
