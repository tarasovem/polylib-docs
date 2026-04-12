# Коллекции

Работа с коллекциями — списками, таблицами, деревьями. Полезные паттерны для типичных UI-задач.

---

## Списки

### Базовый список

```js
static template = html`
  <ul class="list">
    ${repeat(
      this.items,
      (item) => item.id,
      (item) => html`
        <li>[[item.name]]</li>
      `
    )}
  </ul>
`;
```

### Список с действиями

```js
static template = html`
  <ul class="list">
    ${repeat(
      this.items,
      (item) => item.id,
      (item) => html`
        <li>
          <span>[[item.name]]</span>
          <button on-click="() => this.editItem(item)">Ред.</button>
          <button on-click="() => this.deleteItem(item.id)">Удалить</button>
        </li>
      `
    )}
  </ul>
`;

deleteItem(id) {
  const index = this.items.findIndex(i => i.id === id);
  if (index > -1) {
    this.splice('items', index, 1);
  }
}
```

---

## CRUD операции

### Create — создание

```js
addItem(item) {
  this.push('items', {
    id: Date.now(),
    createdAt: new Date(),
    ...item,
  });
}
```

### Read — чтение

```js
getItem(id) {
  return this.items.find(i => i.id === id);
}

hasItem(id) {
  return this.items.some(i => i.id === id);
}
```

### Update — обновление

```js
updateItem(id, changes) {
  const index = this.items.findIndex(i => i.id === id);
  if (index > -1) {
    this.splice('items', index, 1, {
      ...this.items[index],
      ...changes,
      updatedAt: new Date(),
    });
  }
}
```

### Delete — удаление

```js
deleteItem(id) {
  const index = this.items.findIndex(i => i.id === id);
  if (index > -1) {
    this.splice('items', index, 1);
  }
}

clearCompleted() {
  const toDelete = this.items.filter(i => i.done);
  toDelete.forEach(item => {
    const index = this.items.findIndex(i => i.id === item.id);
    if (index > -1) {
      this.splice('items', index, 1);
    }
  });
}
```

---

## Сортировка

### Одно поле

```js
get sortedItems() {
  return [...this.items].sort((a, b) => {
    if (a.name < b.name) return -1;
    if (a.name > b.name) return 1;
    return 0;
  });
}

// Или компактнее:
get sortedItems() {
  return [...this.items].sort((a, b) => a.name.localeCompare(b.name));
}
```

### Множественная сортировка

```js
get sortedItems() {
  return [...this.items].sort((a, b) => {
    // Сначала по статусу
    if (a.status !== b.status) {
      const order = { urgent: 0, normal: 1, low: 2 };
      return order[a.status] - order[b.status];
    }
    // Затем по дате
    return new Date(b.createdAt) - new Date(a.createdAt);
  });
}
```

---

## Фильтрация

### Простая фильтрация

```js
static properties = {
  filter: { type: String },
};

get filteredItems() {
  if (!this.filter) return this.items;
  return this.items.filter(item =>
    item.name.toLowerCase().includes(this.filter.toLowerCase())
  );
}
```

### Множественные фильтры

```js
static properties = {
  searchQuery: { type: String },
  categoryFilter: { type: String },
  statusFilter: { type: String },
};

get filteredItems() {
  return this.items.filter(item => {
    // Поиск
    if (this.searchQuery) {
      const query = this.searchQuery.toLowerCase();
      if (!item.name.toLowerCase().includes(query)) {
        return false;
      }
    }

    // Категория
    if (this.categoryFilter && item.category !== this.categoryFilter) {
      return false;
    }

    // Статус
    if (this.statusFilter && item.status !== this.statusFilter) {
      return false;
    }

    return true;
  });
}
```

---

## Пагинация

```js
static properties = {
  items: { type: Array },
  pageSize: { type: Number },
  currentPage: { type: Number },
};

constructor() {
  super();
  this.pageSize = 10;
  this.currentPage = 1;
}

get paginatedItems() {
  const start = (this.currentPage - 1) * this.pageSize;
  return this.items.slice(start, start + this.pageSize);
}

get totalPages() {
  return Math.ceil(this.items.length / this.pageSize);
}

nextPage() {
  if (this.currentPage < this.totalPages) {
    this.currentPage++;
  }
}

prevPage() {
  if (this.currentPage > 1) {
    this.currentPage--;
  }
}
```

---

## Drag & Drop

```js
static template = html`
  <ul
    class="sortable-list"
    on-dragover="handleDragOver"
    on-drop="handleDrop"
  >
    ${repeat(
      this.items,
      (item) => item.id,
      (item) => html`
        <li
          draggable="true"
          on-dragstart="handleDragStart"
          on-dragend="handleDragEnd"
          data-id$="[[item.id]]"
        >
          [[item.name]]
        </li>
      `
    )}
  </ul>
`;

_handleDragStart(event) {
  event.dataTransfer.setData('text/plain', event.target.dataset.id);
}

_handleDragOver(event) {
  event.preventDefault();
}

_handleDrop(event) {
  event.preventDefault();
  const draggedId = event.dataTransfer.getData('text/plain');
  // Реализуйте логику перемещения
}
```

---

## Выбор элементов

### Одиночный выбор

```js
static properties = {
  selectedId: { type: String },
};

selectItem(id) {
  this.selectedId = id;
}

static template = html`
  ${repeat(
    this.items,
    (item) => item.id,
    (item) => html`
      <div
        class$="[[item.id === selectedId ? 'selected' : '']]"
        on-click="() => this.selectItem(item.id)"
      >
        [[item.name]]
      </div>
    `
  )}
`;
```

### Множественный выбор

```js
toggleSelection(id) {
  const selected = new Set(this.selectedIds);
  if (selected.has(id)) {
    selected.delete(id);
  } else {
    selected.add(id);
  }
  this.set('selectedIds', [...selected]);
}

selectAll() {
  this.set('selectedIds', this.items.map(i => i.id));
}

clearSelection() {
  this.selectedIds = [];
}
```

---

## Практический пример: Task List

```js
class TaskList extends PlElement {
  static properties = {
    tasks: { type: Array },
    filter: { type: String },
    sortBy: { type: String },
  };

  get filteredTasks() {
    let tasks = this.tasks;

    // Фильтрация
    if (this.filter === 'active') {
      tasks = tasks.filter(t => !t.done);
    } else if (this.filter === 'completed') {
      tasks = tasks.filter(t => t.done);
    }

    // Сортировка
    tasks = [...tasks].sort((a, b) => {
      if (this.sortBy === 'date') {
        return new Date(b.createdAt) - new Date(a.createdAt);
      }
      return a.name.localeCompare(b.name);
    });

    return tasks;
  }

  static template = html`
    <div class="task-list">
      <div class="controls">
        <select on-change="e => this.filter = e.target.value">
          <option value="all">Все</option>
          <option value="active">Активные</option>
          <option value="completed">Завершённые</option>
        </select>
      </div>

      ${repeat(
        this.filteredTasks,
        (task) => task.id,
        (task) => html`
          <task-item
            .task="[[task]]"
            on-toggle="() => this.toggleTask(task.id)"
            on-delete="() => this.deleteTask(task.id)"
          ></task-item>
        `
      )}
    </div>
  `;

  toggleTask(id) {
    this.updateItem(id, { done: !this.getItem(id).done });
  }
}
```

---

## Чеклист коллекций

- [ ] Всегда используйте уникальный key в repeat
- [ ] Реализуйте фильтрацию и сортировку
- [ ] Показывайте пустое состояние
- [ ] Пагинируйте большие списки

---

## См. также

- [Директивы](../templates/directives.md)
- [Валидация](validation.md)
- [Манипуляции с данными](../managing-data/data-manipulation.md)
