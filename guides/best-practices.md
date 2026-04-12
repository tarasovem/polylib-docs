# Лучшие практики и паттерны

Рекомендации, выработанные на основе реальных проектов. Следуйте им — и ваши компоненты будут предсказуемыми, поддерживаемыми и переиспользуемыми.

---

## Архитектура компонентов

### Принцип единственной ответственности

Каждый компонент решает одну задачу. Если компонент делает слишком много — разбейте его на части.

**Хорошо:**
```js
class UserCard extends PlElement {
  // Только отображение карточки пользователя
}

class UserList extends PlElement {
  // Только список карточек
}
```

**Плохо:**
```js
class UserDashboard extends PlElement {
  // Загружает данные, рендерит список, фильтры, пагинацию...
}
```

### Композиция вместо наследования

Создавайте_small_ специализированные компоненты и собирайте из них большие.

```js
class IconButton extends PlElement { }
class Badge extends PlElement { }
class Avatar extends PlElement { }

// Затем используйте композицию
class UserCard extends PlElement {
  static template = html`
    <avatar src="[[avatarUrl]]"></avatar>
    <badge>[[role]]</badge>
    <icon-button icon="edit" on-click="onEdit"></icon-button>
  `;
}
```

---

## Работа со свойствами

### Всегда объявляйте тип свойства

```js
static properties = {
  // ✗ Неявные типы
  name: {},
  active: {},

  // ✓ Явные типы
  name: { type: String },
  active: { type: Boolean },
  count: { type: Number },
  items: { type: Array },
  config: { type: Object },
};
```

### Значения по умолчанию через конструктор

```js
constructor() {
  super();
  this.count = 0;
  this.items = [];
  this.active = false;
}
```

### Группируйте связанные свойства

```js
static properties = {
  // Состояние UI
  loading: { type: Boolean },
  error: { type: String },

  // Данные
  items: { type: Array },
  selectedId: { type: String },

  // Конфигурация
  compact: { type: Boolean },
  searchable: { type: Boolean },
};
```

---

## Шаблоны

### Читаемость шаблона

Используйте переносы строк и отступы:

```js
static template = html`
  <div class="container">
    <header>
      <h1>[[title]]</h1>
    </header>
    <main>
      [[content]]
    </main>
    <footer>
      <button on-click="onCancel">Отмена</button>
      <button on-click="onSubmit">Подтвердить</button>
    </footer>
  </div>
`;
```

### Избегайте сложной логики в шаблоне

**Хорошо:**
```js
computedClass() {
  return this.isActive ? 'active' : 'inactive';
}

static template = html`
  <div class$="[[computedClass]]">...</div>
`;
```

**Плохо:**
```js
static template = html`
  <div class$="[[this.isActive ? 'active' : 'inactive']]">...</div>
`;
```

### Предпочитайте события, а не колбеки

```js
// ✓ События — компонент сам решает, когда их выбрасывать
<button on-click="onSave">Сохранить</button>

// onSave вызывает this.dispatchEvent(...)

// ✗ Колбеки — связывают компонент с конкретным использованием
<my-button on-save="handleSave"></my-button>
```

---

## Именование

### Теги компонентов

Используйте префикс, отделяя слова дефисом:

```js
// ✓ Хорошие имена
customElements.define('ui-button', UiButton);
customElements.define('data-table', DataTable);
customElements.define('form-input', FormInput);

// ✗ Плохие имена
customElements.define('button', Button);         // слишком общее
customElements.define('mycomp', MyComp);        // непонятно
customElements.define('UserListItem', MyComp);  // не kebab-case
```

### Классы компонентов

```js
// ✓ PascalCase
class UiButton extends PlElement { }
class DataTable extends PlElement { }

// ✗ camelCase или kebab-case
class uiButton extends PlElement { }
class data-table extends PlElement { }
```

---

## Обработка ошибок

### Всегда валидируйте входные данные

```js
static properties = {
  items: { type: Array },
};

set items(value) {
  if (!Array.isArray(value)) {
    console.warn('items should be an array');
    return;
  }
  this._setProperty('items', value);
}
```

### Graceful degradation

```js
class UserAvatar extends PlElement {
  static properties = {
    src: { type: String },
  };

  static template = html`
    <img
      src="[[src]]"
      alt="Аватар пользователя"
      on-error="onImageError"
    >
  `;

  onImageError() {
    // Показываем placeholder вместо битой картинки
    this.shadowRoot.querySelector('img').src = '/placeholder.svg';
  }
}
```

---

## Производительность

### Избегайте лишних перерисовок

```js
// ✗ Не создавайте объекты в render
static template = html`
  <div style$="[[{ color: 'red' }]]">...</div>
`;

// ✓ Кешируйте вычисления
get computedStyle() {
  if (!this._styleCache) {
    this._styleCache = { color: 'red', fontSize: '14px' };
  }
  return this._styleCache;
}
```

### Мемоизация дорогих вычислений

```js
get filteredItems() {
  if (this._lastFilter !== this.filter) {
    this._lastFilter = this.filter;
    this._cachedFiltered = this.items.filter(...);
  }
  return this._cachedFiltered;
}
```

---

## Тестирование

### Избегайте зависимости от DOM-структуры

```js
// ✗ Привязка к внутренней структуре
const span = component.shadowRoot.querySelector('div > span');
expect(span.textContent).toBe('42');

// ✓ Проверка через публичный API
expect(component.count).toBe(42);
```

### Тестируйте поведение, а не реализацию

```js
// ✗ Тестируем как реализовано
component.set('count', 5);
const button = component.shadowRoot.querySelector('button');
expect(button.classList.contains('active')).toBe(true);

// ✓ Тестируем что должно произойти
component.set('count', 5);
expect(component.isHighValue).toBe(true);
```

---

## Документирование

### JSDoc для свойств

```js
static properties = {
  /**
   * URL изображения аватара
   * @type {string}
   */
  avatarUrl: { type: String },

  /**
   * Имя пользователя
   * @type {string}
   */
  name: { type: String },

  /**
   * Роль пользователя в системе
   * @type {'admin' | 'user' | 'guest'}
   */
  role: { type: String },
};
```

---

## Чеклист перед релизом

- [ ] Все свойства имеют типы
- [ ] Значения по умолчанию заданы
- [ ] Обработка граничных случаев (empty, null, undefined)
- [ ] Нет утечек памяти (event listeners сняты)
- [ ] Доступны a11y-атрибуты (aria-*)
- [ ] Написаны базовые тесты
- [ ] JSDoc комментарии добавлены

---

## См. также

- [Тестирование компонентов](testing.md)
- [Производительность](performance.md)
- [API PlElement](../api/pl-element.md)
