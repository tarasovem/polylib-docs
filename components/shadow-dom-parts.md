# Shadow DOM и ::part

Shadow DOM обеспечивает инкапсуляцию — стили и структура компонента не влияют на остальную страницу и наоборот. Часть `::part()` позволяет внешней стороне стилизовать внутренние элементы.

---

## Что такое Shadow DOM

Shadow DOM создаёт изолированное DOM-дерево для компонента:

```
┌─────────────────────────────────────────┐
│  <my-component> (Light DOM)             │
│                                         │
│  ┌─ Shadow DOM (инкапсулирован) ──────┐ │
│  │  #shadow-root                       │ │
│  │    <div class="wrapper">           │ │
│  │      <button>Нажми</button>        │ │
│  │    </div>                          │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Без Shadow DOM

```js
// Открытый Shadow DOM (по умолчанию)
class OpenShadow extends PlElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
  }
}

// Закрытый Shadow DOM
class ClosedShadow extends PlElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'closed' });
  }
}
```

---

## Атрибут part

Атрибут `part` экспортирует элемент для внешней стилизации:

```js
class IconButton extends PlElement {
  static template = html`
    <button part="button">
      <span part="icon">🔔</span>
      <span part="label">Уведомления</span>
    </button>
  `;
}
```

### Именование частей

```html
<!-- Несколько частей одного типа -->
<div part="item item-first">Первый</div>
<div part="item">Второй</div>
<div part="item item-last">Третий</div>
```

---

## ::part() — стилизация экспортированных элементов

CSS-селектор `::part()` стилизует элементы с атрибутом `part`:

```css
/* Базовые стили */
icon-button::part(button) {
  background: #2196f3;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
}

/* Псевдоклассы */
icon-button::part(button):hover {
  background: #1976d2;
}

icon-button::part(button):active {
  transform: scale(0.98);
}

/* Вложенные элементы */
icon-button::part(label) {
  margin-left: 8px;
}
```

---

## Разница ::slotted и ::part

| ::slotted() | ::part() |
|-------------|----------|
| Стилизация контента из Light DOM | Стилизация внутренних элементов |
| Ограниченные возможности | Полный контроль |
| Для слотов | Для экспортированных элементов |

```js
static template = html`
  <!-- ::slotted — для контента снаружи -->
  <slot></slot>

  <!-- ::part — для внутренних элементов -->
  <div part="wrapper">
    <button part="submit-button">Отправить</button>
  </div>
`;
```

---

## Псевдоэлемент ::part()

### Basic usage

```css
my-card::part(card) {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
}
```

### With pseudo-classes

```css
my-card::part(card):hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

my-card::part(submit):disabled {
  opacity: 0.5;
}
```

### With pseudo-elements

```css
my-card::part(title)::before {
  content: '📌 ';
}
```

---

## Части и состояния

Комбинируйте `part` с data-атрибутами для состояний:

```js
class ToggleSwitch extends PlElement {
  static properties = {
    checked: { type: Boolean },
  };

  static template = html`
    <button
      part="track"
      class$="[[checked ? 'checked' : '']]"
      on-click="toggle"
    >
      <span part="thumb"></span>
    </button>
  `;

  toggle() {
    this.checked = !this.checked;
  }
}
```

```css
toggle-switch::part(track) {
  width: 48px;
  height: 24px;
  border-radius: 12px;
  background: #ccc;
  transition: background 0.2s;
}

toggle-switch::part(track).checked,
toggle-switch::part(track)[aria-checked="true"] {
  background: #4caf50;
}

toggle-switch::part(thumb) {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: white;
  transition: transform 0.2s;
}

toggle-switch::part(track).checked + part[thumb],
toggle-switch[checked]::part(thumb) {
  transform: translateX(24px);
}
```

---

## CSS Custom Properties + ::part

Комбинируйте для максимальной гибкости:

```js
static css = css`
  :host {
    --button-bg: #2196f3;
    --button-color: white;
    --button-radius: 4px;
  }

  button::part(button) {
    background: var(--button-bg);
    color: var(--button-color);
    border-radius: var(--button-radius);
  }
`;
```

```html
<!-- Тема через CSS переменные -->
<themed-button style="
  --button-bg: #e91e63;
  --button-radius: 20px;
">
  Розовая кнопка
</themed-button>
```

---

## Практический пример: Тематизируемая карточка

```js
class ThemedCard extends PlElement {
  static css = css`
    :host {
      --card-bg: white;
      --card-border: #e0e0e0;
      --card-radius: 8px;
    }

    .card {
      background: var(--card-bg);
      border: 1px solid var(--card-border);
      border-radius: var(--card-radius);
    }

    ::slotted(img) {
      border-radius: var(--card-radius) var(--card-radius) 0 0;
    }
  `;

  static template = html`
    <div class="card" part="card">
      <slot></slot>
    </div>
  `;
}
```

```html
<!-- Темы через CSS Custom Properties -->
<style>
  .dark-theme themed-card {
    --card-bg: #333;
    --card-border: #555;
  }
  .blue-theme themed-card {
    --card-bg: #e3f2fd;
    --card-border: #2196f3;
  }
</style>

<themed-card class="dark-theme">
  <img src="photo.jpg" alt="Фото">
  <p>Контент карточки</p>
</themed-card>
```

---

## Именование частей — соглашения

| Название | Использование |
|----------|---------------|
| `container`, `wrapper` | Внешняя обёртка |
| `header`, `body`, `footer` | Секции |
| `title`, `subtitle` | Текстовые заголовки |
| `icon`, `image` | Графика |
| `button` | Интерактивные элементы |
| `input` | Поля ввода |

---

## Ограничения ::part()

- Работает только с прямыми потомками shadow root
- Нельзя стилизовать псевдо-элементы (`::before`, `::after`)
- Нельзя использовать `::slotted()` внутри `::part()`
- Части не наследуют стили хоста

---

## Чеклист

- [ ] Экспортируйте части, которые нужно стилизовать извне
- [ ] Используйте понятные имена (`title` вместо `t1`)
- [ ] Предоставляйте CSS Custom Properties для цветов и размеров
- [ ] Комбинируйте с fallback-стилями

---

## См. также

- [Слоты](slots.md)
- [Стилизация](styling.md)
- [CSS Custom Properties](../guides/best-practices.md)
