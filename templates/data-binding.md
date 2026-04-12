# Биндинги

Биндинги связывают данные из свойств компонента с элементами в шаблоне. Polylib поддерживает однонаправленное и двунаправленное связывание.

---

## Однонаправленный биндинг [[...]]

Данные передаются из свойства в шаблон. Изменение свойства вызывает обновление UI.

### Синтаксис

```js
[[propertyName]]
[[nested.property]]
[[array[0]]]
```

### Примеры

```js
static template = html`
  <!-- Простое свойство -->
  <h1>[[title]]</h1>

  <!-- Вложенное свойство -->
  <p>[[user.name]]</p>
  <p>[[user.address.city]]</p>

  <!-- Элемент массива -->
  <span>[[items[0]]</span>

  <!-- Вызов метода -->
  <span>[[formatDate(date)]]</span>
  <span>[[getGreeting(name)]]</span>
`;
```

---

## Двунаправленный биндинг {{...}}

Данные синхронизируются в обе стороны: свойство → UI и UI → свойство.

### Синтаксис

```js
<input value="{{userName}}">
<textarea>{{description}}</textarea>
<select value="{{selectedOption}}">
```

### Пример

```js
static properties = {
  userName: { type: String },
  email: { type: String },
};

static template = html`
  <form>
    <input
      type="text"
      value="{{userName}}"
      placeholder="Имя"
    >
    <input
      type="email"
      value="{{email}}"
      placeholder="Email"
    >
  </form>
  <p>Привет, [[userName]]!</p>
`;
```

### Когда использовать

| Однонаправленный | Двунаправленный |
|-----------------|----------------|
| Отображение данных | Формы ввода |
| Read-only поля | Пользовательский ввод |
| Disabled элементы | Editing состояние |

---

## Атрибутные биндинги

### bind:$attr

Для атрибутов HTML используйте `bind:$attr`:

```js
static template = html`
  <!-- Синтаксис bind:$атрибут -->
  <a bind:$href="url" bind:$target="target">Ссылка</a>

  <!-- class как объект -->
  <div class$="[[getClassName()]]">Див</div>

  <!-- class как строка -->
  <div class$="active">Див</div>
`;
```

### Boolean атрибуты

```js
// Для boolean используйте ?
static template = html`
  <input type="checkbox" ?checked="[[isChecked]]">
  <button ?disabled="[[isLoading]]">Загрузка...</button>
`;
```

### Стили

```js
static template = html`
  <div style$="[[getStyles()]]">Стилизованный</div>

  <!-- Или объект -->
  <div
    style$="width: [[width]]px; height: [[height]]px;"
  ></div>
`;
```

---

## Отрицание

```js
// Отрицание в шаблоне
static template = html`
  <div ?hidden="[[!isLoggedIn]]">
    Пожалуйста, войдите
  </div>
`;

// Эквивалент в JavaScript
get isVisible() {
  return !this.isLoggedIn;
}
```

---

## Условные биндинги

### Директива ?

Для условного добавления атрибутов:

```js
static template = html`
  <!-- Атрибут добавится только если true -->
  <input ?required="[[isRequired]]">
  <input ?disabled="[[isDisabled]]">

  <!-- class добавится если условие true -->
  <div class$="box [[isActive ? 'active' : '']]">
    Контент
  </div>
`;
```

---

## Вложенные шаблоны

```js
// Можно использовать результат метода
renderHeader() {
  return html`<h1>[[title]]</h1>`;
}

static template = html`
  ${this.renderHeader()}
  <main>${this.renderContent()}</main>
`;
```

### Фрагменты

```js
static template = html`
  ${this.isLoggedIn ? this._renderUserPanel() : this._renderLoginForm()}
`;

_renderUserPanel() {
  return html`<div class="user">Привет, [[userName]]</div>`;
}

_renderLoginForm() {
  return html`<form>...</form>`;
}
```

---

## Комплексные примеры

### Профиль пользователя

```js
static properties = {
  user: { type: Object },
  isEditing: { type: Boolean },
};

static template = html`
  <div class="profile">
    <img
      src="[[user.avatar]]"
      alt$="[[user.name]]"
      class$="[[isEditing ? 'editing' : '']]"
    >
    <h2>[[user.name]]</h2>
    <p>[[user.email]]</p>

    <button ?disabled="[[!user.verified]]">
      [[user.verified ? 'Подтверждён' : 'Не подтверждён']]
    </button>
  </div>
`;
```

### Список задач

```js
static properties = {
  tasks: { type: Array },
  filter: { type: String },
};

get filteredTasks() {
  if (this.filter === 'completed') {
    return this.tasks.filter(t => t.done);
  }
  if (this.filter === 'active') {
    return this.tasks.filter(t => !t.done);
  }
  return this.tasks;
}

static template = html`
  <ul>
    ${this.filteredTasks.map(task => html`
      <li class$="[[task.done ? 'done' : '']]">
        <input
          type="checkbox"
          ?checked="[[task.done]]"
        >
        <span>[[task.title]]</span>
      </li>
    `)}
  </ul>
`;
```

---

## Ограничения биндингов

| Можно | Нельзя |
|-------|--------|
| `[[property]]` | `[[a + b]]` — выражения |
| `[[obj.prop]]` | `[[a ? b : c]]` — тернарные |
| `[[arr[0]]]` | `[[for...]]` — циклы |
| `[[method()]]` | `[[if...]]` — условия |

Для логики используйте computed properties или методы:

```js
// ✗ Не работает
[[items.filter(i => i.active).length]]

// ✓ Работает
getActiveCount() {
  return this.items.filter(i => i.active).length;
}

static template = html`[[getActiveCount()]]`;
```

---

## Чеклист

- [ ] Используйте `[[...]]` для отображения данных
- [ ] Используйте `{{...}}` только для форм
- [ ] Выносите логику в методы
- [ ] Проверяйте undefined/null в шаблоне

---

## См. также

- [Директивы](directives.md)
- [Свойства и шаблоны](../components/properties-templates.md)
- [Манипуляции с данными](../managing-data/data-manipulation.md)
