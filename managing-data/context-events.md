# Контекст и события

Система контекста позволяет передавать данные через дерево компонентов без пропс-дриллинга. События обеспечивают коммуникацию между компонентами.

---

## Контекст (провайдеры)

### Провайдер контекста

```js
import { provide } from 'polylib';

class ThemeProvider extends PlElement {
  static properties = {
    theme: { type: String },
  };

  static css = css`
    :host {
      display: contents;
    }
  `;

  constructor() {
    super();
    this._contextId = 'theme-context';
  }

  connectedCallback() {
    super.connectedCallback();
    provide(this, this._contextId, {
      theme: this.theme,
      setTheme: (theme) => {
        this.theme = theme;
      },
    });
  }
}
```

### Потребитель контекста

```js
import { consume } from 'polylib';

class ThemedButton extends PlElement {
  static css = css`
    :host {
      display: inline-block;
    }
    button {
      background: var(--button-bg, #2196f3);
      color: var(--button-color, white);
    }
  `;

  constructor() {
    super();
    consume(this, 'theme-context');
  }

  contextCallback(name, value) {
    if (name === 'theme') {
      this.style.setProperty('--button-bg', value.theme === 'dark' ? '#333' : '#2196f3');
    }
  }
}
```

---

## Паттерн Context API

### Множественные значения

```js
// Провайдер
provide(this, 'app-context', {
  user: this.user,
  settings: this.settings,
  actions: {
    logout: () => this.logout(),
    updateSettings: (settings) => this.updateSettings(settings),
  },
});

// Потребитель
consume(this, 'app-context');

contextCallback(name, value) {
  if (name === 'app-context') {
    this.user = value.user;
    this.actions = value.actions;
  }
}
```

---

## Встроенный контекст

Polylib предоставляет встроенные контексты:

```js
// Контекст перевода (если настроен)
consume(this, 'i18n');

// Контекст темы
consume(this, 'theme');
```

---

## Кастомные события

### Базовое событие

```js
class MyButton extends PlElement {
  onClick() {
    this.dispatchEvent(new Event('my-click'));
  }
}
```

### Событие с данными

```js
this.dispatchEvent(new CustomEvent('action', {
  detail: { action: 'save', timestamp: Date.now() },
  bubbles: true,
  composed: true,
}));
```

### Параметры события

| Параметр | Описание |
|----------|----------|
| `type` | Тип события |
| `detail` | Данные события |
| `bubbles` | Всплытие по DOM |
| `composed` | Проход через Shadow DOM |
| `cancelable` | Возможность отмены |

---

## Обработка событий

### В шаблоне

```js
static template = html`
  <my-button on-my-click="handleClick"></my-button>
`;

handleClick() {
  console.log('Кнопка нажата!');
}
```

### Через addEventListener

```js
connectedCallback() {
  super.connectedCallback();
  this.addEventListener('my-click', this.handleClick);
}

disconnectedCallback() {
  super.disconnectedCallback();
  this.removeEventListener('my-click', this.handleClick);
}
```

---

## Делегирование событий

### Один обработчик для многих элементов

```js
static template = html`
  <div class="toolbar" on-click="handleToolbar">
    <button data-action="bold">B</button>
    <button data-action="italic">I</button>
    <button data-action="underline">U</button>
  </div>
`;

handleToolbar(event) {
  const action = event.target.dataset.action;
  if (!action) return;

  this.dispatchEvent(new CustomEvent('format', {
    detail: { action },
    bubbles: true,
    composed: true,
  }));
}
```

---

## Изолированные события

### Без всплытия

```js
// Событие останется внутри компонента
this.dispatchEvent(new CustomEvent('internal'));
```

### Со всплытием

```js
// Всплывает по DOM, но не выходит из Shadow DOM
this.dispatchEvent(new CustomEvent('action', {
  bubbles: true,
}));
```

### Через Shadow DOM

```js
// Выйдет за пределы Shadow DOM
this.dispatchEvent(new CustomEvent('action', {
  bubbles: true,
  composed: true,
}));
```

---

## Практический пример: Form Builder

```js
// Field Component
class FormField extends PlElement {
  static properties = {
    name: { type: String },
    value: { type: String },
    error: { type: String },
  };

  static template = html`
    <label>
      <slot name="label"></slot>
      <input
        name$="[[name]]"
        value="{{value}}"
        on-input="handleInput"
        on-blur="handleBlur"
      >
      ${when(this.error, () => html`
        <span class="error">[[error]]</span>
      `)}
    </label>
  `;

  handleInput(event) {
    this.value = event.target.value;
    this.dispatchEvent(new CustomEvent('field-change', {
      detail: { name: this.name, value: this.value },
      bubbles: true,
      composed: true,
    }));
  }
}
```

---

## Event Bus

### Глобальные события

```js
// Простой event bus
class EventBus {
  constructor() {
    this.listeners = new Map();
  }

  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }

  off(event, callback) {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      const index = callbacks.indexOf(callback);
      if (index > -1) callbacks.splice(index, 1);
    }
  }

  emit(event, data) {
    const callbacks = this.listeners.get(event);
    if (callbacks) {
      callbacks.forEach(cb => cb(data));
    }
  }
}

export const bus = new EventBus();
```

---

## Чеклист событий

-[ ] Используйте `composed: true` для событий из Shadow DOM
- [ ] Очищайте слушатели в `disconnectedCallback`
- [ ] Документируйте события компонента
- [ ] Используйте понятные имена (`item-select` вместо `sel`)

---

## Чеклист контекста

- [ ] Не злоупотребляйте контекстом
- [ ] Один контекст — одна ответственность
- [ ] Предоставляйте значения по умолчанию

---

## См. также

- [Манипуляции с данными](data-manipulation.md)
- [Слоты](../components/slots.md)
- [Лучшие практики](../guides/best-practices.md)
