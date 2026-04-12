# Контекст и события

События — основной способ коммуникации между компонентами и внешним миром. Polylib предоставляет удобный синтаксис для их обработки.

---

## Обработка событий в шаблоне

### Директива on-*

```js
static template = html`
  <button on-click="handleClick">Нажми</button>
  <input on-input="handleInput" on-blur="handleBlur">
  <div on-mouseenter="handleMouseEnter"></div>
`;
```

### Имя метода

Имя обработчика указывается как строка — Polylib найдёт метод в классе:

```js
class MyComponent extends PlElement {
  // Этот метод будет вызван при клике
  handleClick(event) {
    console.log('Клик!');
  }

  // Можно использовать стрелочную функцию для привязки
  handleClick = (event) => {
    console.log('Клик!', this);
  };
}
```

---

## Параметры обработчика

### event и scope

```js
handleClick(event, scope) {
  // event — стандартный DOM Event
  console.log('Тип:', event.type);

  // scope — привязанный контекст (подробнее ниже)
  console.log('Scope:', scope);
}
```

### Привязка данных к событию

```js
static template = html`
  <button on-click="handleClick" data-id="1">Кнопка 1</button>
  <button on-click="handleClick" data-id="2">Кнопка 2</button>
`;

handleClick(event) {
  const id = event.currentTarget.dataset.id;
  console.log('ID:', id);
}
```

---

## Генерация событий

### dispatchEvent

```js
class SubmitButton extends PlElement {
  onClick() {
    // Простое событие
    this.dispatchEvent(new Event('click'));

    // Кастомное событие с данными
    this.dispatchEvent(new CustomEvent('submit', {
      detail: { timestamp: Date.now() },
      bubbles: true,
      composed: true,
    }));
  }
}
```

### Опции события

| Опция | Описание | По умолчанию |
|-------|----------|--------------|
| `bubbles` | Всплывать по DOM | `false` |
| `composed` | Проходить через Shadow DOM | `false` |
| `cancelable` | Можно отменить | `false` |
| `detail` | Данные события | `null` |

---

## События и Shadow DOM

### composed: true

Без `composed` событие не выйдет за пределы Shadow DOM:

```js
// ✗ Событие останется внутри
this.dispatchEvent(new CustomEvent('action'));

// ✓ Событие выйдет наружу
this.dispatchEvent(new CustomEvent('action', {
  composed: true,
}));
```

### Пример

```js
class InnerButton extends PlElement {
  static template = html`
    <button on-click="handleClick">Кликни</button>
  `;

  handleClick() {
    this.dispatchEvent(new CustomEvent('my-action', {
      bubbles: true,
      composed: true,
      detail: { source: 'inner' }
    }));
  }
}
```

```html
<!-- Работает! -->
<inner-button on-my-action="handleAction"></inner-button>
```

---

## Контекст в обработчиках

### scope — привязанные данные

```js
static template = html`
  <ul>
    ${this.items.map(item => html`
      <li on-click="handleItemClick" data-item$="[[item.id]]">
        [[item.name]]
      </li>
    `)}
  </ul>
`;

handleItemClick(event, scope) {
  // scope содержит привязанные данные
  console.log(scope.item);
  console.log(scope.index);
}
```

---

## Распространённые события

| Событие | Когда | Пример |
|---------|-------|--------|
| `input` | Ввод в поле | `<input on-input="...">` |
| `change` | Изменение значения | `<select on-change="...">` |
| `submit` | Отправка формы | `<form on-submit="...">` |
| `toggle` | Переключение | `<details on-toggle="...">` |

### Формы

```js
static template = html`
  <form on-submit="handleSubmit">
    <input name="email" type="email">
    <button type="submit">Отправить</button>
  </form>
`;

handleSubmit(event) {
  event.preventDefault();
  const form = event.currentTarget;
  const data = new FormData(form);
  console.log(data.get('email'));
}
```

---

## Делегирование событий

### Один обработчик для многих элементов

```js
static template = html`
  <div class="list" on-click="handleListClick">
    <button data-action="add">Добавить</button>
    <button data-action="delete">Удалить</button>
    <button data-action="edit">Редактировать</button>
  </div>
`;

handleListClick(event) {
  const action = event.target.dataset.action;
  if (!action) return;

  switch (action) {
    case 'add': this.addItem(); break;
    case 'delete': this.deleteItem(); break;
    case 'edit': this.editItem(); break;
  }
}
```

---

## Удаление обработчиков

### cleanup в disconnectedCallback

```js
constructor() {
  super();
  this.boundHandler = this.handleClick.bind(this);
}

connectedCallback() {
  super.connectedCallback();
  window.addEventListener('resize', this.boundHandler);
}

disconnectedCallback() {
  super.disconnectedCallback();
  window.removeEventListener('resize', this.boundHandler);
}
```

---

## Практический пример

```js
class SearchInput extends PlElement {
  static properties = {
    value: { type: String },
    debounceMs: { type: Number },
  };

  constructor() {
    super();
    this.debounceMs = 300;
    this._debounceTimer = null;
  }

  static template = html`
    <input
      type="search"
      value="[[value]]"
      placeholder="Поиск..."
      on-input="handleInput"
    >
  `;

  handleInput(event) {
    this.value = event.target.value;
    this.debouncedSearch();
  }

  debouncedSearch() {
    clearTimeout(this._debounceTimer);
    this._debounceTimer = setTimeout(() => {
      this.dispatchEvent(new CustomEvent('search', {
        detail: { query: this.value },
        bubbles: true,
        composed: true,
      }));
    }, this.debounceMs);
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    clearTimeout(this._debounceTimer);
  }
}
```

```html
<search-input
  placeholder="Введите запрос..."
  debounce-ms="500"
  on-search="handleSearch"
></search-input>
```

---

## Чеклист событий

- [ ] Используйте `composed: true` для событий из Shadow DOM
- [ ] Используйте `bubbles: true` для всплытия
- [ ] Удаляйте обработчики в `disconnectedCallback`
- [ ] Привязывайте обработчики через `bind()` или стрелки
- [ ] Проверяйте `event.target` при делегировании

---

## См. также

- [Шаблоны и биндинги](../templates/data-binding.md)
- [Манипуляции с данными](../managing-data/data-manipulation.md)
- [Лучшие практики](../guides/best-practices.md)
