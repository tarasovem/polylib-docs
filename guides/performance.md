# Производительность

Оптимизация Web Components для лучшей производительности.

---

## Почему это важно

- Более быстрый First Contentful Paint
- Меньше CPU-usage
- Плавные анимации (60fps)
- Лучший UX на мобильных устройствах

---

## Ленивая загрузка

### Динамический импорт

```js
// Вместо статического импорта
import '@plcmp/components';

// Используйте динамический
async function showModal() {
  const { Modal } = await import('@plcmp/components');
  const modal = new Modal();
  document.body.appendChild(modal);
}
```

### lazy-loading атрибут

```html
<!-- Загрузится только когда станет видимым -->
<heavy-component lazy></heavy-component>
```

---

## Мемоизация

### Computed properties

```js
// ✗ Вычисляется каждый рендер
get formattedItems() {
  return this.items.map(item => ({
    ...item,
    formattedPrice: item.price.toLocaleString(),
  }));
}

// ✓ Кэшируется
get formattedItems() {
  if (this._cache.formattedItems && this._cache.items === this.items) {
    return this._cache.formattedItems;
  }
  this._cache.items = this.items;
  this._cache.formattedItems = this.items.map(item => ({
    ...item,
    formattedPrice: item.price.toLocaleString(),
  }));
  return this._cache.formattedItems;
}
```

### Методы vs Getters

```js
// ✗ Создаёт новый массив каждый раз
renderItems() {
  return this.items.filter(i => i.visible);
}

// ✓ Запоминает результат
get visibleItems() {
  return this._visibleItems;
}

set items(value) {
  this._visibleItems = value.filter(i => i.visible);
}
```

---

## Избегайте лишних обновлений

### Проверяйте изменения

```js
updated(changedProperties) {
  // ✗ Всегда обновляется
  this.shadowRoot.querySelector('.count').textContent = this.count;

  // ✓ Только при изменении count
  if (changedProperties.has('count')) {
    this.shadowRoot.querySelector('.count').textContent = this.count;
  }
}
```

### Debounce обновлений

```js
constructor() {
  super();
  this._updateTimer = null;
}

set searchQuery(value) {
  clearTimeout(this._updateTimer);
  this._updateTimer = setTimeout(() => {
    this.set('searchQuery', value);
  }, 300);
}
```

---

## Оптимизация списков

### Key в repeat

```js
// ✗ Полная перерисовка при изменении
${repeat(this.items, (item) => null, (item) => html`<li>[[item.name]]</li>`)}

// ✓ Эффективные обновления
${repeat(this.items, (item) => item.id, (item) => html`<li>[[item.name]]</li>`)}
```

### Виртуализация

Для больших списков (> 100 элементов):

```js
class VirtualList extends PlElement {
  static properties = {
    items: { type: Array },
    itemHeight: { type: Number },
    visibleCount: { type: Number },
  };

  constructor() {
    super();
    this.itemHeight = 50;
    this.visibleCount = 10;
  }

  get scrollTop() {
    return this._scrollTop || 0;
  }

  set scrollTop(value) {
    this._scrollTop = value;
    const start = Math.floor(value / this.itemHeight);
    this.set('visibleItems', this.items.slice(start, start + this.visibleCount + 2));
  }

  onScroll(event) {
    this.scrollTop = event.target.scrollTop;
  }
}
```

---

## CSS оптимизации

### will-change

```css
/* Для анимируемых элементов */
.animating-element {
  will-change: transform, opacity;
}

/* Но не злоупотребляйте! */
```

### contain

```css
/* Изолирует layout перерасчёты */
.card {
  contain: content;
}

/* Значения: none | strict | content | layout | style | paint */
```

### will-change: contents

```js
// Для часто обновляемого текста
this.shadowRoot.querySelector('.counter').style.willChange = 'contents';
```

---

## Анимации

### CSS transitions вместо JS

```css
/* ✗ JavaScript анимация */
element.style.transform = `translateX(${x}px)`;
requestAnimationFrame(() => { ... });

/* ✓ CSS transition */
element.classList.add('moving');
```

```css
.moving {
  transition: transform 0.3s ease;
  transform: translateX(var(--target-x));
}
```

### transform и opacity

```css
/* ✗ Дорогие свойства */
.animating {
  transition: left 0.3s, top 0.3s, width 0.3s;
}

/* ✓ GPU-ускоряемые */
.animating {
  transition: transform 0.3s, opacity 0.3s;
}
```

---

## Асинхронные данные

### Параллельная загрузка

```js
async loadAll() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments(),
  ]);

  this.set('users', users);
  this.set('posts', posts);
  this.set('comments', comments);
}
```

### Приоритизация

```js
async loadPage() {
  // Критичный контент — синхронно
  this.set('layout', await fetchLayout());

  // Менее важный — отложенно
  requestIdleCallback(() => {
    this.loadRecommendations();
  });
}
```

---

## Профилирование

### Chrome DevTools

1. Откройте Performance вкладку
2. Запустите запись
3. Взаимодействуйте с компонентом
4. Смотрите Long Tasks

### Custom Elements DevTools

Установите [Custom Elements DevTools](https://chrome.google.com/webstore/detail/custom-elements-devtools/focksbofjegppgcjhdfkppmiknodneig) для:
- Инспектирования компонентов
- Мониторинга обновлений
- Отладки Shadow DOM

---

## Метрики

### First Contentful Paint (FCP)

```js
// Измеряйте время до первого рендера
const start = performance.now();
customElements.whenDefined('my-component').then(() => {
  console.log('Ready:', performance.now() - start, 'ms');
});
```

### Time to Interactive (TTI)

```js
// Компонент интерактивен когда...
this.updateComplete.then(() => {
  // Shadow DOM готов
});
```

---

## Чеклист оптимизации

- [ ] Используйте `key` в repeat
- [ ] Мемоизируйте вычисления
- [ ] Debounce частых обновлений
- [ ] CSS transitions вместо JS анимаций
- [ ] contain для изоляции
- [ ] will-change для анимаций
- [ ] Lazy loading для тяжёлых компонентов
- [ ] Виртуализация для больших списков

---

## Ресурсы

- [Web Performance Working Group](https://www.w3.org/webperformance/)
- [Google Web Vitals](https://web.dev/vitals/)
- [Custom Elements Performance](https://developers.google.com/web/fundamentals/web-components/best-practices)

---

## См. также

- [Лучшие практики](best-practices.md)
- [Тестирование](testing.md)
