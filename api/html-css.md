# html / css

Функции `html` и `css` — основа шаблонизации и стилизации в Polylib.

---

## html

Создаёт DOM-шаблон из литеральных строк.

### Базовый синтаксис

```js
import { html } from 'polylib';

const template = html`<div class="container">Hello</div>`;
```

### Вложенные шаблоны

```js
const header = html`<h1>Заголовок</h1>`;
const footer = html`<footer>Подвал</footer>`;

const page = html`
  <div class="page">
    ${header}
    <main>Контент</main>
    ${footer}
  </div>
`;
```

---

## Биндинги в html

### Однонаправленный [[...]]

```js
html`<h1>[[title]]</h1>`;
html`<img src="[[imageUrl]]">`;
html`<div class$="[[className]]">`;
```

### Двунаправленный {{...}}

```js
html`<input value="{{inputValue}}">`;
html`<textarea>{{content}}</textarea>`;
```

### Boolean атрибуты ?

```js
html`<input ?disabled="[[isDisabled]]">`;
html`<div ?hidden="[[isHidden]]">`;
```

---

## css

Создаёт CSS-стили для компонента.

### Базовый синтаксис

```js
import { css } from 'polylib';

const styles = css`
  :host {
    display: block;
  }
  .content {
    color: red;
  }
`;
```

---

## :host

Стилизация самого компонента:

```js
static css = css`
  :host {
    display: block;
  }

  :host([disabled]) {
    opacity: 0.5;
  }

  :host(.theme-dark) {
    --bg-color: #333;
  }
`;
```

---

## Вложенные селекторы

```js
static css = css`
  .card {
    padding: 16px;
  }

  .card .title {
    font-size: 18px;
  }

  .card:hover .title {
    color: blue;
  }
`;
```

---

## CSS Custom Properties

```js
static css = css`
  :host {
    --primary-color: #2196f3;
    --border-radius: 4px;
    --spacing: 8px;
  }

  button {
    background: var(--primary-color);
    border-radius: var(--border-radius);
    padding: var(--spacing);
  }
`;
```

---

## Динамические стили

### Через свойства

```js
static template = html`
  <div
    style$="background: [[bgColor]]; color: [[textColor]];"
  ></div>
`;
```

### Через computed

```js
get computedStyles() {
  return `
    background: ${this.bgColor};
    border: ${this.borderWidth}px solid ${this.borderColor};
  `;
}

static template = html`
  <div style$="[[computedStyles]]"></div>
`;
```

---

## Медиа-запросы

```js
static css = css`
  :host {
    display: block;
  }

  @media (max-width: 600px) {
    :host {
      display: none;
    }
  }

  @media (min-width: 1200px) {
    .container {
      max-width: 1000px;
    }
  }
`;
```

---

## Псевдоклассы

```js
static css = css`
  button {
    transition: all 0.2s;
  }

  button:hover {
    background: blue;
  }

  button:active {
    transform: scale(0.98);
  }

  button:focus-visible {
    outline: 2px solid blue;
  }
`;
```

---

## Псевдоэлементы

```js
static css = css`
  .required::before {
    content: '*';
    color: red;
    margin-right: 4px;
  }

  .truncate {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
`;
```

---

## ::slotted()

Стилизация контента в слотах:

```js
static css = css`
  ::slotted(img) {
    width: 100%;
    border-radius: 8px;
  }

  ::slotted(a) {
    color: var(--link-color, #2196f3);
  }
`;
```

---

## ::part()

Стилизация экспортированных частей:

```js
// В компоненте
static template = html`
  <button part="button">
    <span part="label">Текст</span>
  </button>
`;

// Извне
// my-button::part(button) { ... }
// my-button::part(label) { ... }
```

---

## Антипаттерны

### Не используйте встроенные стили для layout

```js
// ✗ Плохо
html`<div style="display: flex;">`;

// ✓ Хорошо — стили в css
static css = css`
  .container { display: flex; }
`;
html`<div class="container">`;
```

---

## Чеклист стилей

- [ ] Все значения через CSS-переменные
- [ ] Используйте :host для компонента
- [ ] Сбрасывайте стили с :host
- [ ] Комбинируйте ::slotted и ::part

---

## См. также

- [PlElement](pl-element.md)
- [Стилизация](../components/styling.md)
- [Shadow DOM и ::part](../components/shadow-dom-parts.md)
