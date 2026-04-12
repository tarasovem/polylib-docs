# Жизненный цикл

Методы жизненного цикла вызываются автоматически в определённые моменты существования компонента.

---

## Порядок вызова

```
┌──────────────────────────────────────────────────────────────┐
│                     constructor()                           │
│                          │                                   │
│              ┌───────────┴───────────┐                       │
│              ▼                       ▼                       │
│    connectedCallback()      disconnectedCallback()           │
│              │                       │                       │
│              ▼                       │                       │
│    attributeChangedCallback() ◄─────┘                       │
│              │                                               │
│              ▼                                               │
│         requestUpdate()                                      │
│              │                                               │
│              ▼                                               │
│         updated()                                            │
│              │                                               │
│              ▼                                               │
│       updateComplete ✓                                       │
└──────────────────────────────────────────────────────────────┘
```

---

## constructor()

Вызывается при создании экземпляра.

```js
constructor() {
  super(); // Обязательно!

  // Инициализация
  this.items = [];
  this.loading = false;
}
```

**Что можно делать:**
- Инициализация свойств
- Привязка методов
- Настройка internal state

**Чего нельзя делать:**
- Доступ к DOM (shadowRoot ещё null)
- addEventListener на внешние элементы
- Асинхронные операции

---

## connectedCallback()

Вызывается при добавлении в DOM.

```js
connectedCallback() {
  super.connectedCallback();

  // Компонент теперь в DOM
  this.addEventListener('click', this.handleClick);
}
```

**Что делать:**
- Добавление event listeners
- Запуск инициализации
- Запрос данных

---

## disconnectedCallback()

Вызывается при удалении из DOM.

```js
disconnectedCallback() {
  super.disconnectedCallback();

  // Очистка!
  this.removeEventListener('click', this.handleClick);
  clearTimeout(this._timer);
  clearInterval(this._interval);
}
```

**Обязательно очищайте:**
- addEventListener → removeEventListener
- setTimeout → clearTimeout
- setInterval → clearInterval
- fetch/XHR → abort()
- MutationObserver → disconnect()
- WebSocket → close()

---

## attributeChangedCallback()

Вызывается при изменении атрибутов.

```js
// Какие атрибуты отслеживать
static observedAttributes = ['disabled', 'loading', 'theme'];

attributeChangedCallback(name, oldValue, newValue) {
  if (oldValue === newValue) return;

  switch (name) {
    case 'disabled':
      this._handleDisabled(newValue !== null);
      break;
    case 'loading':
      this._handleLoading(newValue !== null);
      break;
    case 'theme':
      this._applyTheme(newValue);
      break;
  }
}
```

### observedAttributes

```js
// Автоматически для всех
static get observedAttributes() {
  return ['disabled', 'variant'];
}

// Или через свойства
static properties = {
  disabled: { type: Boolean, reflect: true },
};
// disabled автоматически в observedAttributes
```

---

## updated(changedProperties)

Вызывается после каждого обновления.

```js
updated(changedProperties) {
  // changedProperties — Map с изменившимися свойствами
  if (changedProperties.has('items')) {
    this._onItemsChange(this.items);
  }

  if (changedProperties.has('loading')) {
    this._updateLoadingState();
  }
}
```

**Важно:** Не изменяйте свойства здесь — вызовет бесконечный цикл!

---

## requestUpdate()

Запрашивает обновление рендера.

```js
// Автоматически вызывается при this.set()
this.set('value', 42); // → requestUpdate()

// Принудительно
this.requestUpdate();
```

---

## updateComplete

Promise, резолвится после рендера.

```js
async refresh() {
  this.items = await fetchItems();

  // Ждём рендера
  await this.updateComplete;

  // DOM готов
  const first = this.shadowRoot.querySelector('.item');
  first.scrollIntoView();
}
```

### Проверка готовности

```js
if (this.updateComplete) {
  this.updateComplete.then(() => {
    // Рендер завершён
  });
}
```

---

## Практический пример

```js
class DataList extends PlElement {
  static properties = {
    url: { type: String },
    items: { type: Array },
    loading: { type: Boolean },
    error: { type: String },
  };

  constructor() {
    super();
    this.items = [];
    this.loading = false;
    this._abortController = null;
  }

  connectedCallback() {
    super.connectedCallback();
    this._loadData();
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    // Очистка
    this._abortController?.abort();
  }

  updated(changedProperties) {
    if (changedProperties.has('url')) {
      this._loadData();
    }
  }

  async _loadData() {
    this._abortController?.abort();
    this._abortController = new AbortController();

    this.loading = true;
    this.error = null;

    try {
      const response = await fetch(this.url, {
        signal: this._abortController.signal,
      });
      this.set('items', await response.json());
    } catch (e) {
      if (e.name !== 'AbortError') {
        this.set('error', e.message);
      }
    } finally {
      this.set('loading', false);
}
  }
}
```

---

## Таблица методов

| Метод | Когда | Для чего |
|-------|-------|----------|
| constructor() | Создание | Инициализация |
| connectedCallback() | В DOM | Слушатели, загрузка |
| disconnectedCallback() | Из DOM | Очистка |
| attributeChangedCallback() | Атрибут | Синхронизация |
| updated() | После рендера | Побочные эффекты |
| updateComplete | После рендера | Ожидание DOM |

---

## Типичные ошибки

| Ошибка | Решение |
|--------|---------|
| Утечка памяти | Всегда очищайте в disconnectedCallback |
| Race conditions | Используйте AbortController |
| Бесконечный цикл | Не меняйте свойства в updated() |
| Доступ к null | Проверяйте this.shadowRoot |

---

## См. также

- [PlElement](pl-element.md)
- [Жизненный цикл компонента](../components/lifecycle.md)
- [Лучшие практики](../guides/best-practices.md)
