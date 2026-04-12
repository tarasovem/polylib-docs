# Стилизация

Polylib предоставляет несколько механизмов для стилизации компонентов. Выбор зависит от того, что нужно изолировать, а что — открыть для кастомизации.

---

## static css

Определите стили компонента через статическое свойство `css`:

```js
import { PlElement, html, css } from 'polylib';

class MyButton extends PlElement {
  static css = css`
    :host {
      display: inline-block;
    }

    button {
      padding: 8px 16px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
  `;

  static template = html`
    <button part="button">
      <slot></slot>
    </button>
  `;
}
```

---

## :host

Псевдокласс `:host` стилизует сам элемент-обёртку:

```js
static css = css`
  :host {
    display: block;
    margin: 8px 0;
  }

  /* Состояния хоста */
  :host([disabled]) {
    opacity: 0.5;
    pointer-events: none;
  }

  :host([theme="dark"]) {
    --bg: #333;
    --text: #fff;
  }

  /* Псевдоклассы */
  :host(:focus-within) {
    outline: 2px solid blue;
  }
`;
```

### :host() — условные стили

```css
:host(.primary) button {
  background: blue;
}

:host(.secondary) button {
  background: gray;
}
```

---

## Каскад и наследование

### Наследуемые свойства

Некоторые CSS-свойства наследуются внутрь Shadow DOM:

```js
static css = css`
  /* Наследуется */
  .text {
    color: inherit;
    font-family: inherit;
    font-size: inherit;
  }

  /* Не наследуется (нужно задать явно) */
  .box {
    background: white; /* Не inherit! */
  }
`;
```

### Принудительное наследование

```js
static css = css`
  :host {
    color: var(--text-color, inherit);
    font-family: var(--font-family, inherit);
  }
`;
```

---

## CSS Custom Properties

### Внутренние переменные

```js
static css = css`
  :host {
    --button-bg: #2196f3;
    --button-color: white;
    --button-padding: 8px 16px;
    --button-radius: 4px;
  }

  button {
    background: var(--button-bg);
    color: var(--button-color);
    padding: var(--button-padding);
    border-radius: var(--button-radius);
  }
`;
```

### Темы через CSS-переменные

```html
<!-- Тема по умолчанию -->
<my-button>Кнопка</my-button>

<!-- Кастомная тема -->
<my-button style="
  --button-bg: #e91e63;
  --button-radius: 20px;
">
  Розовая кнопка
</my-button>
```

### Наследование переменных

```js
// Родительский компонент
static css = css`
  :host {
    --theme-primary: blue;
    --theme-secondary: lightblue;
  }
`;

// Дочерний компонент использует те же переменные
static css = css`
  .primary {
    background: var(--theme-primary);
  }
`;
```

---

## Медиа-запросы

### Внутри компонента

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
    .sidebar {
      width: 300px;
    }
  }
`;
```

### Responsive дизайн

```js
static css = css`
  :host {
    --columns: 1;
  }

  @media (min-width: 600px) {
    :host {
      --columns: 2;
    }
  }

  @media (min-width: 900px) {
    :host {
      --columns: 3;
    }
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(var(--columns), 1fr);
  }
`;
```

---

## Псевдоклассы и псевдоэлементы

```js
static css = css`
  button {
    transition: all 0.2s ease;
  }

  button:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  }

  button:active {
    transform: translateY(0);
  }

  button::before {
    content: '';
    display: inline-block;
    width: 8px;
    height: 8px;
    margin-right: 8px;
    border-radius: 50%;
    background: currentColor;
  }
`;
```

---

## Адаптивные стили

### Container Queries (если поддерживается)

```js
static css = css`
  @container (min-width: 400px) {
    :host {
      --button-size: large;
    }
  }

  :host([size="large"]) button {
    padding: 12px 24px;
    font-size: 16px;
  }
`;
```

---

## Динамические стили

### Через свойства

```js
class ThemedBox extends PlElement {
  static properties = {
    bgColor: { type: String },
  };

  static template = html`
    <div
      class="box"
      style$="background: [[bgColor]]"
    ></div>
  `;
}
```

### Через computed стили

```js
get computedStyle() {
  return {
    backgroundColor: this.bgColor || 'white',
    borderColor: this.borderColor || '#ddd',
  };
}

static template = html`
  <div
    class="box"
    style$="[[JSON.stringify(computedStyle)]]"
  ></div>
`;
```

---

## Организация стилей

### Методология

```js
static css = css`
  /* 1. Переменные */
  :host {
    --color-primary: #2196f3;
    --color-text: #333;
    --space-sm: 8px;
    --space-md: 16px;
  }

  /* 2. Базовые стили */
  :host {
    display: block;
    font-family: inherit;
  }

  /* 3. Состояния */
  :host([disabled]) { opacity: 0.5; }
  :host([loading]) { pointer-events: none; }

  /* 4. Компоненты */
  .header { ... }
  .content { ... }
  .footer { ... }
`;
```

---

## Чеклист стилей

- [ ] Все размеры через `var()` для темизации
- [ ] Цвета через CSS Custom Properties
- [ ] Отступы через spacing-переменные
- [ ] Адаптивность через `@media`
- [ ] Состояния через атрибуты хоста

---

## Распространённые паттерны

### Кнопка с иконкой

```js
static css = css`
  :host {
    --icon-size: 20px;
    --gap: 8px;
  }

  .button {
    display: inline-flex;
    align-items: center;
    gap: var(--gap);
  }

  .icon {
    width: var(--icon-size);
    height: var(--icon-size);
  }

  ::slotted(svg) {
    width: var(--icon-size);
    height: var(--icon-size);
  }
`;
```

### Карточка с тенью

```js
static css = css`
  :host {
    --shadow-sm: 0 1px 3px rgba(0,0,0,0.1);
    --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
    --shadow-lg: 0 10px 25px rgba(0,0,0,0.15);
  }

  .card {
    background: white;
    border-radius: 8px;
    box-shadow: var(--shadow-md);
    transition: box-shadow 0.2s;
  }

  :host(:hover) .card {
    box-shadow: var(--shadow-lg);
  }
`;
```

---

## См. также

- [Shadow DOM и ::part](shadow-dom-parts.md)
- [Слоты](slots.md)
- [Лучшие практики](../guides/best-practices.md)
