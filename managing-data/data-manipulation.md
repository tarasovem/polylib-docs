# Манипуляции с данными

Polylib предоставляет удобные методы для работы с данными в компонентах. Следуйте этим паттернам для предсказуемого управления состоянием.

---

## this.set()

Установка значения свойства с уведомлением об изменении:

```js
// Базовое использование
this.set('count', 42);
this.set('user', { name: 'Иван' });
this.set('items', [...this.items, newItem]);
```

### Почему не просто присваивание?

```js
// ✗ Может не вызвать обновление
this.count = 42;

// ✓ Всегда вызывает обновление
this.set('count', 42);
```

---

## this.push()

Добавление элемента в массив:

```js
// Добавить один элемент
this.push('items', newItem);

// Добавить несколько
this.push('items', item1, item2, item3);

// Эквивалент:
// this.items = [...this.items, newItem];
```

### Пример: Добавление задачи

```js
addTask(title) {
  this.push('tasks', {
    id: Date.now(),
    title,
    done: false,
    createdAt: new Date(),
  });
}
```

---

## this.splice()

Удаление и замена элементов массива:

```js
// Удалить 1 элемент начиная с индекса 2
this.splice('items', 2, 1);

// Вставить элементы
this.splice('items', 2, 0, newItem1, newItem2);

// Заменить 1 элемент
this.splice('items', 2, 1, replacementItem);
```

### Примеры

```js
// Удалить первый элемент
this.splice('items', 0, 1);

// Удалить последний элемент
this.splice('items', this.items.length - 1, 1);

// Удалить все completed
this.splice(
  'tasks',
  this.tasks.findIndex(t => t.done),
  this.tasks.filter(t => t.done).length
);
```

---

## Иммутабельные операции

Всегда создавайте новые объекты/массивы:

```js
// ✗ Мутация
this.user.name = 'Новое имя';
this.items.push(item);
this.data[key] = value;

// ✓ Иммутабельность
this.set('user', { ...this.user, name: 'Новое имя' });
this.set('items', [...this.items, item]);
this.set('data', { ...this.data, [key]: value });
```

### Глубокая иммутабельность

```js
// Для вложенных объектов
this.set('user', {
  ...this.user,
  profile: {
    ...this.user.profile,
    avatar: newUrl,
  }
});
```

---

## Computed properties

Вычисляемые свойства кэшируют результат:

```js
get fullName() {
  return `${this.user.firstName} ${this.user.lastName}`;
}

get sortedItems() {
  return [...this.items].sort((a, b) => a.name.localeCompare(b.name));
}

get completedCount() {
  return this.tasks.filter(t => t.done).length;
}
```

### В шаблоне

```js
static template = html`
  <p>Выполнено: [[completedCount]] из [[tasks.length]]</p>
`;
```

---

## Паттерны данных

### Список с фильтрацией

```js
static properties = {
  items: { type: Array },
  filter: { type: String },
};

get filteredItems() {
  if (!this.filter || this.filter === 'all') {
    return this.items;
  }
  return this.items.filter(item =>
    item.category === this.filter
  );
}

static template = html`
  ${repeat(
    this.filteredItems,
    (item) => item.id,
    (item) => html`<item-card .item="[[item]]"></item-card>`
  )}
`;
```

### Дерево данных

```js
getFlattenedTree(node, depth = 0) {
  return [
    { ...node, depth },
    ...node.children.flatMap(child =>
      this.getFlattenedTree(child, depth + 1)
    )
  ];
}
```

---

## Async данные

### Загрузка с состоянием

```js
static properties = {
  data: { type: Array },
  loading: { type: Boolean },
  error: { type: String },
};

async connectedCallback() {
  super.connectedCallback();
  await this.loadData();
}

async loadData() {
  this.loading = true;
  this.error = null;

  try {
    const response = await fetch(this.url);
    this.set('data', await response.json());
  } catch (e) {
    this.set('error', e.message);
  } finally {
    this.set('loading', false);
  }
}
```

### Debounce

```js
constructor() {
  super();
  this._debounceTimer = null;
}

search(query) {
  clearTimeout(this._debounceTimer);
  this._debounceTimer = setTimeout(() => {
    this.performSearch(query);
  }, 300);
}
```

---

## Связанные данные

### Parent → Child

```js
// Родитель передаёт данные
class Parent extends PlElement {
  static template = html`
    <child-component .data="[[items]]"></child-component>
  `;
}

// Дочерний получает через свойство
class Child extends PlElement {
  static properties = {
    data: { type: Array },
  };
}
```

### Child → Parent

```js
// Дочерний выбрасывает событие
class Child extends PlElement {
  handleSelect(item) {
    this.dispatchEvent(new CustomEvent('select', {
      detail: item,
      bubbles: true,
      composed: true,
    }));
  }
}

// Родитель слушает
class Parent extends PlElement {
  static template = html`
    <child-component on-select="handleSelect"></child-component>
  `;

  handleSelect(event) {
    console.log('Выбран:', event.detail);
  }
}
```

---

## Нормализация данных

```js
// Приведение типов в setter
set value(val) {
  this._value = Number(val) || 0;
}

get value() {
  return this._value;
}

// Или через свойство
static properties = {
  value: {
    type: Number,
    set(value) {
      return Number(value) || 0;
    }
  }
};
```

---

## Чеклист

- [ ] Используйте `this.set()` для обновления свойств
- [ ] Никогда не мутируйте объекты напрямую
- [ ] Всегда создавайте новые массивы при изменении
- [ ] Очищайте таймеры в `disconnectedCallback`
- [ ] Обрабатывайте loading и error состояния

---

## См. также

- [Контекст и события](context-events.md)
- [Биндинги](../templates/data-binding.md)
- [Лучшие практики](../guides/best-practices.md)
