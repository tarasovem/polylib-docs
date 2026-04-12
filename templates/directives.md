# Директивы

Директивы — это специальные команды в шаблоне, которые управляют рендерингом элементов. Polylib предоставляет встроенные директивы для условного отображения и работы со списками.

---

## Директива repeat

Повторяет элемент для каждого item в массиве.

### Базовый синтаксис

```js
static template = html`
  <ul>
    ${repeat(
      this.items,
      (item) => item.id,
      (item, index) => html`
        <li>[[index]]: [[item.name]]</li>
      `
    )}
  </ul>
`;
```

### Параметры

| Параметр | Описание |
|----------|---------|
| Массив | Итерируемые данные |
| Key-функция | Уникальный идентификатор для each элемента |
| Render-функция | Что рендерить для каждого элемента |

### Примеры

```js
// Простой список
${repeat(this.users, (user) => user.id, (user) => html`
  <user-item name="[[user.name]]"></user-item>
`)}

// С индексом
${repeat(this.items, (item) => item.id, (item, i) => html`
  <div>[[i + 1]]. [[item.title]]</div>
`)}

// Вложенные списки
${repeat(this.categories, (cat) => cat.id, (cat) => html`
  <li>
    [[cat.name]]
    <ul>
      ${repeat(cat.products, (p) => p.id, (p) => html`
        <li>[[p.name]]</li>
      `)}
    </ul>
  </li>
`)}
```

---

## Директива if

Условный рендеринг.

### when / else

```js
static template = html`
  ${when(
    this.isLoggedIn,
    () => html`<div>Добро пожаловать!</div>`,
    () => html`<div>Войдите в систему</div>`
  )}
`;
```

### Вложенные условия

```js
static template = html`
  ${when(
    this.user,
    () => html`
      <div class="user-info">
        <p>[[user.name]]</p>
        ${when(
          user.isAdmin,
          () => html`<span class="badge">Админ</span>`,
          () => html`<span class="badge">Пользователь</span>`
        )}
      </div>
    `,
    () => html`<p>Загрузка...</p>`
  )}
`;
```

---

## Директива choose

Множественные условия (аналог switch-case).

```js
static template = html`
  ${choose(
    this.status,
    [
      ['loading', () => html`<div>Загрузка...</div>`],
      ['error', () => html`<div>Ошибка: [[errorMessage]]</div>`],
      ['success', () => html`<div>Успех!</div>`],
    ],
    () => html`<div>Неизвестный статус</div>`
  )}
`;
```

---

## Комбинирование директив

### repeat + when

```js
static template = html`
  ${repeat(
    this.items.filter(item => item.visible),
    (item) => item.id,
    (item) => html`
      ${when(
        item.inStock,
        () => html`<product-card .item="[[item]]"></product-card>`,
        () => html`<div class="sold-out">[[item.name]] — Распродан</div>`
      )}
    `
  )}
`;
```

### repeat с группировкой

```js
// Группировка по категориям
const grouped = Object.entries(
  this.products.reduce((acc, p) => {
    (acc[p.category] = acc[p.category] || []).push(p);
    return acc;
  }, {})
);

static template = html`
  ${repeat(
    grouped,
    ([category]) => category,
    ([category, items]) => html`
      <h2>[[category]]</h2>
      ${repeat(items, (item) => item.id, (item) => html`
        <product-card .item="[[item]]"></product-card>
      `)}
    `
  )}
`;
```

---

## key — оптимизация

Всегда используйте уникальный key в repeat:

```js
// ✗ Без key — полная перерисовка при изменении
${repeat(this.items, (item) => html`
  <li>[[item.name]]</li>
`)}

// ✓ С key — эффективные обновления
${repeat(this.items, (item) => item.id, (item) => html`
  <li>[[item.name]]</li>
`)}
```

### Типы keys

| Тип | Пример | Когда использовать |
|-----|--------|-------------------|
| ID | `item.id` | Уникальный числовой/строковый ID |
| Index | `(item, i) => i` | Простые списки без изменений |
| Composite | `item.id + '-' + item.type` | Составные ключи |

---

## Практические примеры

### Дерево файлов

```js
renderNode(node) {
  return html`
    <div class="tree-node">
      ${when(node.type === 'folder', () => html`
        <details open>
          <summary>[[node.name]]</summary>
          ${repeat(node.children, (c) => c.path, (c) => this.renderNode(c))}
        </details>
      `, () => html`
        <span>[[node.name]]</span>
      `)}
    </div>
  `;
}

static template = html`
  <div class="file-tree">
    ${repeat(this.files, (f) => f.path, (f) => this.renderNode(f))}
  </div>
`;
```

### Таблица с фильтрацией и сортировкой

```js
get sortedData() {
  return [...this.data].sort((a, b) => {
    if (this.sortBy === 'name') {
      return a.name.localeCompare(b.name);
    }
    return a.value - b.value;
  });
}

static template = html`
  <table>
    <thead>
      <tr>
        <th on-click="() => this.sortBy = 'name'">Имя</th>
        <th on-click="() => this.sortBy = 'value'">Значение</th>
      </tr>
    </thead>
    <tbody>
      ${repeat(
        this.sortedData,
        (row) => row.id,
        (row) => html`
          <tr>
            <td>[[row.name]]</td>
            <td>[[row.value]]</td>
          </tr>
        `
      )}
    </tbody>
  </table>
`;
```

---

## Производительность

### Избегайте создания функций в шаблоне

```js
// ✗ Создаёт новую функцию каждый рендер
${this.items.map(item => html`<li>[[item.name]]</li>`)}

// ✓ Переиспользуйте функции
_renderItem = (item) => html`<li>[[item.name]]</li>`;

static template = html`
  ${this.items.map(this._renderItem)}
`;
```

### Виртуализация для больших списков

Для списков > 100 элементов рассмотрите виртуализацию:

```js
// Простая виртуализация
get visibleItems() {
  const start = this.scrollPosition;
  const end = start + this.visibleCount;
  return this.allItems.slice(start, end);
}
```

---

## Директива unsafeHTML

Для вставки HTML из строки:

```js
import { unsafeHTML } from 'polylib';

// ⚠️ Опасно! Используйте только для доверенного контента
static template = html`
  <div>${unsafeHTML(this.rawHtml)}</div>
`;
```

---

## Чеклист директив

- [ ] Всегда используйте key в repeat
- [ ] Выносите render-функции в методы
- [ ] Комбинируйте repeat + when для фильтрации
- [ ] Проверяйте пустые массивы

---

## См. также

- [Биндинги](data-binding.md)
- [Манипуляции с данными](../managing-data/data-manipulation.md)
- [Производительность](../guides/performance.md)
