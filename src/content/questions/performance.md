---
title: "Производительность и оптимизация"
description: "Рефлоу, репайнт, ленивая загрузка, оптимизация загрузки страницы"
category: "Браузерные API"
difficulty: "medium"
tags: ["performance", "optimization", "reflow", "repaint"]
order: 13
---

## Рефлоу и Репайнт

### Reflow (Рефлоу)

**Пересчет геометрии элементов** — самая дорогая операция.

Вызывают рефлоу:
- Изменение размеров (width, height, padding, margin, border)
- Изменение позиции (top, left, right, bottom)
- Изменение шрифта (font-size, font-family)
- Добавление/удаление элементов DOM
- Изменение классов
- Изменение содержимого (textContent, innerHTML)
- Вычисление размеров (offsetWidth, clientHeight, scrollTop)
- Изменение размера окна

```javascript
// ❌ Плохо — 3 рефлоу
element.style.width = '100px';
element.style.height = '100px';
element.style.margin = '10px';

// ✅ Хорошо — 1 рефлоу
element.style.cssText = 'width: 100px; height: 100px; margin: 10px;';

// ✅ Или через класс
element.classList.add('styled');
```

### Repaint (Репайнт)

**Перерисовка без изменения геометрии** — дешевле рефлоу.

Вызывают репайнт:
- Изменение цвета (color, background-color)
- Изменение видимости (visibility)
- Изменение outline

```javascript
// Только репайнт (без рефлоу)
element.style.color = 'red';
element.style.backgroundColor = 'blue';
```

## Оптимизация DOM операций

### Пакетные изменения

```javascript
// ❌ Плохо — много рефлоу
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = i;
  container.appendChild(div); // Рефлоу на каждой итерации
}

// ✅ Хорошо — DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = i;
  fragment.appendChild(div);
}
container.appendChild(fragment); // Один рефлоу

// ✅ Или через innerHTML (если нет обработчиков)
const html = Array.from({ length: 1000 }, (_, i) => 
  `<div>${i}</div>`
).join('');
container.innerHTML = html;
```

### Скрытие элемента при изменениях

```javascript
// ❌ Плохо
element.style.width = '100px';
element.style.height = '100px';
element.style.padding = '10px';

// ✅ Хорошо — скрываем, меняем, показываем
element.style.display = 'none'; // 1 рефлоу
element.style.width = '100px';
element.style.height = '100px';
element.style.padding = '10px';
element.style.display = 'block'; // 1 рефлоу
// Итого: 2 рефлоу вместо 3
```

### Клонирование для массовых изменений

```javascript
const clone = element.cloneNode(true);
// Вносим изменения в клон
clone.style.width = '100px';
clone.style.height = '100px';
// Заменяем оригинал
element.parentNode.replaceChild(clone, element);
```

### Кэширование размеров

```javascript
// ❌ Плохо — вызывает рефлоу на каждой итерации
for (let i = 0; i < elements.length; i++) {
  elements[i].style.top = container.offsetHeight + 'px';
}

// ✅ Хорошо — кэшируем размер
const containerHeight = container.offsetHeight;
for (let i = 0; i < elements.length; i++) {
  elements[i].style.top = containerHeight + 'px';
}
```

## Ленивая загрузка (Lazy Loading)

### Изображения

```html
<!-- Встроенная ленивая загрузка -->
<img src="image.jpg" loading="lazy" alt="Описание">

<!-- С Intersection Observer -->
<img data-src="image.jpg" class="lazy" alt="Описание">
```

```javascript
// Intersection Observer для изображений
const imageObserver = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.classList.remove('lazy');
      observer.unobserve(img);
    }
  });
});

document.querySelectorAll('img.lazy').forEach(img => {
  imageObserver.observe(img);
});
```

### Компоненты (React)

```javascript
import React, { lazy, Suspense } from 'react';

// Ленивая загрузка компонента
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Загрузка...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Модули (JavaScript)

```javascript
// Динамический импорт
button.addEventListener('click', async () => {
  const module = await import('./heavy-module.js');
  module.doSomething();
});
```

## Оптимизация загрузки страницы

### Critical CSS

```html
<!-- Критичные стили inline -->
<style>
  /* Стили для первого экрана */
  .header { ... }
  .hero { ... }
</style>

<!-- Остальные стили асинхронно -->
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
```

### Preload, Prefetch, Preconnect

```html
<!-- Preload — загрузить ресурс заранее -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="hero.jpg" as="image">

<!-- Prefetch — загрузить для следующей страницы -->
<link rel="prefetch" href="next-page.html">

<!-- Preconnect — установить соединение заранее -->
<link rel="preconnect" href="https://api.example.com">

<!-- DNS-prefetch — только DNS lookup -->
<link rel="dns-prefetch" href="https://fonts.googleapis.com">
```

### Defer и Async для скриптов

```html
<!-- Async — загрузить и выполнить асинхронно -->
<script src="analytics.js" async></script>

<!-- Defer — загрузить асинхронно, выполнить после парсинга HTML -->
<script src="app.js" defer></script>

<!-- Обычный — блокирует парсинг -->
<script src="blocking.js"></script>
```

**Разница:**
- **async**: загружается параллельно, выполняется сразу после загрузки
- **defer**: загружается параллельно, выполняется после парсинга HTML
- **без атрибутов**: блокирует парсинг HTML

### Code Splitting

```javascript
// Webpack — автоматическое разделение
import(/* webpackChunkName: "chart" */ './chart.js')
  .then(module => {
    module.renderChart();
  });

// Vite — аналогично
const module = await import('./heavy-module.js');
```

## Виртуализация списков

Для больших списков — рендерить только видимые элементы.

```javascript
// Пример с react-window
import { FixedSizeList } from 'react-window';

function VirtualList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index]}
    </div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

## Debounce и Throttle для событий

```javascript
// Debounce для поиска
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((value) => {
  fetch(`/api/search?q=${value}`)
    .then(r => r.json())
    .then(results => displayResults(results));
}, 300);

searchInput.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});

// Throttle для скролла
const handleScroll = throttle(() => {
  const scrollPosition = window.scrollY;
  // Обновить UI
}, 100);

window.addEventListener('scroll', handleScroll);
```

## Web Workers

Для тяжелых вычислений — использовать отдельный поток.

```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ data: largeArray });

worker.onmessage = (e) => {
  console.log('Результат:', e.data);
};

// worker.js
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};
```

## Мемоизация

```javascript
// Кэширование результатов функции
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Использование
const expensiveFunction = memoize((n) => {
  console.log('Вычисляем...');
  return n * 2;
});

expensiveFunction(5); // Вычисляем... 10
expensiveFunction(5); // 10 (из кэша)
```

## Инструменты для анализа производительности

### Chrome DevTools

```javascript
// Performance API
performance.mark('start');

// Код для измерения
doSomething();

performance.mark('end');
performance.measure('doSomething', 'start', 'end');

const measure = performance.getEntriesByName('doSomething')[0];
console.log('Время выполнения:', measure.duration);

// Очистка
performance.clearMarks();
performance.clearMeasures();
```

### Lighthouse

```bash
# Установка
npm install -g lighthouse

# Запуск
lighthouse https://example.com --view
```

### Web Vitals

```javascript
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

getCLS(console.log); // Cumulative Layout Shift
getFID(console.log); // First Input Delay
getFCP(console.log); // First Contentful Paint
getLCP(console.log); // Largest Contentful Paint
getTTFB(console.log); // Time to First Byte
```

## Чек-лист оптимизации

### HTML
- ✅ Минимизировать DOM дерево
- ✅ Использовать семантические теги
- ✅ Defer/async для скриптов

### CSS
- ✅ Минифицировать CSS
- ✅ Удалить неиспользуемые стили
- ✅ Critical CSS inline
- ✅ Избегать сложных селекторов

### JavaScript
- ✅ Минифицировать и сжимать (gzip/brotli)
- ✅ Code splitting
- ✅ Tree shaking
- ✅ Ленивая загрузка модулей

### Изображения
- ✅ Оптимизировать размер и формат (WebP, AVIF)
- ✅ Responsive images (`srcset`)
- ✅ Lazy loading
- ✅ CDN для статики

### Сеть
- ✅ HTTP/2 или HTTP/3
- ✅ Кэширование (Cache-Control)
- ✅ Compression (gzip, brotli)
- ✅ CDN

## Советы для собеседования

1. **Рефлоу дороже репайнта** — минимизируйте изменения геометрии
2. **Пакетные DOM операции** — используйте DocumentFragment
3. **Ленивая загрузка** — для изображений и компонентов
4. **Code splitting** — разделяйте код на чанки
5. **Debounce/Throttle** — для частых событий (scroll, resize, input)
6. **Web Workers** — для тяжелых вычислений
7. **Измеряйте производительность** — используйте Performance API и Lighthouse
