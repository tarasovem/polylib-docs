# Жизненный цикл компонента

Понимание жизненного цикла поможет правильно инициализировать компонент, работать с DOM и избегать утечек памяти.

---

## Обзор цикла

```
┌─────────────────────────────────────────────────────┐
│                    Жизненный цикл                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  constructor()     ──►  Создание объекта             │
│        │                                                │
│        ▼                                                │
│  connectedCallback() ──►  Добавлен в DOM             │
│        │                                                │
│        ▼                                                │
│  (repeat)  ◄──────────────────────────┐               │
│        │                               │               │
│        ▼                               │               │
│  attributeChangedCallback()  ◄── Изменился атрибут    │
│        │                               │               │
│        ▼                               │               │
│  requestUpdate() ──►  Запрос обновления ──┘           │
│        │                                                │
│        ▼                                                │
│  updateComplete ──►  Рендер завершён                  │
│        │                                                │
│        ▼                                                │
│  disconnectedCallback() ──►  Удалён из DOM           │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## constructor()

Вызывается при создании элемента. Используйте для:

- Инициализации свойств по умолчанию
- Настройки Shadow DOM (Polylib делает это автоматически)
- Привязки методов

```js
constructor() {
  super(); // Обязательно!

  // Инициализация свойств
  this.items = [];
  this.loading = false;

  // Привязка методов
  this.handleScroll = this.handleScroll.bind(this);
}
```

### Не делайте в constructor()

- Не обращайтесь к DOM (shadowRoot ещё не существует)
- Не запускайте асинхронные операции
- Не добавляйте event listeners к внешним элементам

---

## connectedCallback()

Вызывается при добавлении элемента в DOM. Используйте для:

- Добавления event listeners
- Запуска инициализации
- Запроса данных

```js
connectedCallback() {
  super.connectedCallback();

  // Добавление слушателей
  window.addEventListener('resize', this.handleResize);
  document.addEventListener('visibilitychange', this.handleVisibility);

  // Инициализация данных
  this.loadInitialData();
}
```

---

## disconnectedCallback()

Вызывается при удалении элемента из DOM. **Критически важно** для предотвращения утечек памяти.

```js
disconnectedCallback() {
  super.disconnectedCallback();

  // Удаление всех слушателей!
  window.removeEventListener('resize', this.handleResize);
  document.removeEventListener('visibilitychange', this.handleVisibility);

  // Очистка таймеров
  clearTimeout(this._debounceTimer);
  clearInterval(this._pollTimer);

  // Отмена запросов
  this._pendingRequest?.abort();
}
```

### Чеклист очистки

- [ ] removeEventListener для всех addEventListener
- [ ] clearTimeout для всех setTimeout
- [ ] clearInterval для всех setInterval
- [ ] abort() для fetch/XHR запросов
- [ ] close() для WebSocket
- [ ] disconnect() для MutationObserver

---

## attributeChangedCallback()

Вызывается при изменении атрибутов. Работает только для атрибутов, перечисленных в `observedAttributes`.

```js
static properties = {
  name: { type: String, attribute: 'name' },
  value: { type: Number, attribute: 'value' },
};

// Какие атрибуты отслеживать
static observedAttributes = ['disabled', 'loading'];

attributeChangedCallback(name, oldValue, newValue) {
  if (oldValue === newValue) return;

  switch (name) {
    case 'disabled':
      this._handleDisabled(newValue !== null);
      break;
    case 'loading':
      this._handleLoading(newValue !== null);
      break;
  }
}
```

### Типичное использование

```js
static properties = {
  // Атрибут "theme" связывается с свойством "theme"
  theme: { type: String, attribute: 'theme' },
};

attributeChangedCallback(name, oldValue, newValue) {
  if (name === 'theme' && oldValue !== null) {
    this._applyTheme(newValue);
  }
}
```

---

## requestUpdate()

Вызывает перерисовку компонента. Polylib делает это автоматически при изменении свойств.

```js
// Принудительное обновление
this.requestUpdate();

// С задержкой до следующего фрейма
await this.updateComplete;
this.requestUpdate();
```

---

## updateComplete

Promise, который резолвится после завершения рендера. Полезен для:

- Тестирования
- Измерения размеров элемента
- Фокуса после обновления

```js
async afterUpdate() {
  await this.updateComplete;

  // Теперь можно измерить элемент
  const height = this.offsetHeight;
}

// Альтернатива
this.updateComplete.then(() => {
  console.log('Рендер завершён');
});
```

### Ожидание обновления

```js
async refresh() {
  this.items = await fetchItems();

  // Ждём завершения рендера
  await this.updateComplete;

  // Теперь элементы существуют в DOM
  const firstItem = this.shadowRoot.querySelector('.item');
  firstItem.scrollIntoView();
}
```

---

## updated()

Переопределите для выполнения действий после каждого обновления:

```js
updated(changedProperties) {
  if (changedProperties.has('items')) {
    console.log('Items изменились:', this.items);
  }

  if (changedProperties.has('loading')) {
    this._updateLoadingState();
  }
}
```

---

## Полный пример

```js
class DataFetcher extends PlElement {
  static properties = {
    url: { type: String },
    loading: { type: Boolean },
    data: { type: Object },
    error: { type: String },
  };

  constructor() {
    super();
    this.loading = false;
    this.data = null;
    this.error = null;
    this._abortController = null;
  }

  connectedCallback() {
    super.connectedCallback();
    this._fetch();
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    this._abortController?.abort();
  }

  updated(changedProperties) {
    if (changedProperties.has('url')) {
      this._fetch();
    }
  }

  async _fetch() {
    this._abortController?.abort();
    this._abortController = new AbortController();

    this.loading = true;
    this.error = null;

    try {
      const response = await fetch(this.url, {
        signal: this._abortController.signal,
      });
      this.data = await response.json();
    } catch (e) {
      if (e.name !== 'AbortError') {
        this.error = e.message;
      }
    } finally {
      this.loading = false;
    }
  }
}
```

---

## График вызовов

| Действие | constructor | connected | updated | updateComplete |
|----------|-------------|-----------|--------|----------------|
| Создание | ✓ | | | |
| Добавление в DOM | | ✓ | | |
| Изменение свойства | | | ✓ | ✓ |
| Удаление из DOM | | | | ✓ |

---

## Типичные ошибки

| Ошибка | Решение |
|--------|---------|
| Утечка памяти | Всегда очищайте в disconnectedCallback |
| Race conditions | Используйте AbortController |
| Доступ к null | Проверяйте this.shadowRoot |
| Зацикливание | Не меняйте свойства в updated() |

---

## См. также

- [API PlElement](../api/pl-element.md)
- [Работа с данными](../managing-data/data-manipulation.md)
- [Тестирование](../guides/testing.md)
