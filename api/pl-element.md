# PlElement

Базовый класс для всех компонентов Polylib. Наследуйтесь от него для создания Web Components.

---

## Импорт

```js
import { PlElement } from 'polylib';
```

---

## Статические свойства

### properties

Объявление реактивных свойств:

```js
class MyComponent extends PlElement {
  static properties = {
    // Базовое объявление
    name: { type: String },

    // С значением по умолчанию
    count: { type: Number, value: 0 },

    // Только для атрибутов
    variant: { type: String, attribute: 'variant' },

    // Кастомный converter
    items: {
      type: Array,
      value: () => [],
    },
  };
}
```

### css

Стили компонента:

```js
class MyComponent extends PlElement {
  static css = css`
    :host {
      display: block;
    }
    .content {
      color: var(--text-color, inherit);
    }
  `;
}
```

### template

Шаблон компонента:

```js
class MyComponent extends PlElement {
  static template = html`
    <div class="content">
      <h1>[[title]]</h1>
      <slot></slot>
    </div>
  `;
}
```

---

## Свойства экземпляра

### shadowRoot

Доступ к Shadow DOM:

```js
connectedCallback() {
  super.connectedCallback();
  const button = this.shadowRoot.querySelector('button');
}
```

### $

Быстрый доступ к элементам по ID:

```js
// При наличии <div id="content">
this.$.content.textContent = 'Hello';

// Эквивалент
this.shadowRoot.getElementById('content');
```

---

## Методы

### set(name, value)

Установка свойства с уведомлением:

```js
this.set('count', 42);
this.set('user', { name: 'Иван' });
```

### get(name)

Получение значения свойства:

```js
const count = this.get('count');
```

### requestUpdate()

Принудительный запрос обновления:

```js
this.requestUpdate();
```

---

## Методы для массивов

### push(name, ...items)

Добавить элементы:

```js
this.push('items', newItem);
this.push('items', item1, item2, item3);
```

### splice(name, start, deleteCount, ...items)

Удалить/заменить элементы:

```js
// Удалить 1 элемент
this.splice('items', 2, 1);

// Заменить элемент
this.splice('items', 2, 1, newItem);
```

---

## Жизненный цикл

### constructor()

```js
constructor() {
  super();
  this._privateData = [];
}
```

### connectedCallback()

```js
connectedCallback() {
  super.connectedCallback();
  // Компонент добавлен в DOM
}
```

### disconnectedCallback()

```js
disconnectedCallback() {
  super.disconnectedCallback();
  // Компонент удалён из DOM
  // Очистка: removeEventListener, clearInterval и т.д.
}
```

### updated(changedProperties)

```js
updated(changedProperties) {
  if (changedProperties.has('count')) {
    console.log('Count изменился:', this.count);
  }
}
```

---

## updateComplete

Promise, резолвящийся после рендера:

```js
async someMethod() {
  this.data = await fetchData();
  await this.updateComplete;
  // DOM обновлён
}
```

---

## Пример использования

```js
import { PlElement, html, css } from 'polylib';

class UserCard extends PlElement {
  // Объявление свойств
  static properties = {
    user: { type: Object },
    compact: { type: Boolean },
  };

  // Стили
  static css = css`
    :host {
      display: block;
    }
    .card {
      padding: 16px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
    :host([compact]) .card {
      padding: 8px;
    }
    .avatar {
      width: 48px;
      height: 48px;
      border-radius: 50%;
    }
  `;

  // Шаблон
  static template = html`
    <div class="card" part="card">
      <img class="avatar" src="[[user.avatar]]" alt="[[user.name]]">
      <h3>[[user.name]]</h3>
      ${when(this.user.role, () => html`
        <span class="role">[[user.role]]</span>
      `)}
    </div>
  `;
}

customElements.define('user-card', UserCard);
```

---

## Расширение PlElement

### От другого компонента

```js
import { PlElement, html, css } from 'polylib';

class ThemedElement extends PlElement {
  static css = css`
    :host {
      --theme-bg: white;
      --theme-color: black;
    }
  `;
}

class SpecialButton extends ThemedElement {
  static css = css`
    :host {
      --theme-bg: #2196f3;
      --theme-color: white;
    }
    button {
      background: var(--theme-bg);
      color: var(--theme-color);
    }
  `;
}
```

---

## Типизация (TypeScript)

```typescript
import { PlElement, html, css } from 'polylib';

interface User {
  name: string;
  avatar?: string;
  role?: string;
}

class UserCard extends PlElement {
  static properties = {
    user: { type: Object as { type: User } },
    compact: { type: Boolean },
  };

  user!: User;
  compact = false;
}
```

---

## См. также

- [html / css](html-css.md)
- [Жизненный цикл](lifecycle.md)
- [Свойства и шаблоны](../components/properties-templates.md)
