# Идеология Polylib

Философия библиотеки строится на трёх столпах: **декларативность**, **нативность** и **простота**.

---

## Декларативный подход

Компонент описывает **что** должно отображаться, а не **как** это сделать. Вы определяете связь между данными и UI, а библиотека обеспечивает синхронизацию.

```js
// Вы говорите: "показывай имя пользователя"
static template = html`
  <h1>[[userName]]</h1>
`;

// Не нужно вручную обновлять DOM при изменении
this.userName = 'Новое имя'; // UI обновится автоматически
```

### Почему это важно?

- **Меньше ошибок** — невозможно забыть обновить DOM
- **Проще поддержка** — состояние в одном месте
- **Предсказуемость** — UI всегда отражает данные

---

## Данные через свойства

Единственный источник истины — **свойства компонента**. Не храните состояние в DOM или глобальных переменных.

```js
// ✗ Неправильно — состояние в DOM
const span = this.shadowRoot.querySelector('span');
span.textContent = 'Новое значение';

// ✗ Неправильно — глобальная переменная
window.appState = { count: 42 };

// ✓ Правильно — свойство компонента
this.set('count', 42);
```

### Принцип: свойства = состояние

```js
class Counter extends PlElement {
  static properties = {
    count: { type: Number },  // Это состояние
  };

  onClick() {
    this.count++; // Единственный способ изменить UI
  }
}
```

---

## this.$ — только для UI-операций

Прямой доступ к DOM через `this.$` (или `this.shadowRoot`) допустим только для:

| Допустимо | Не допустимо |
|-----------|--------------|
| Фокус элемента | Чтение данных |
| Анимации | Изменение данных |
| Интеграция с сторонними библиотеками | Манипуляция контентом |
| Измерение размеров | Создание элементов |

```js
// ✓ Допустимо
this.shadowRoot.querySelector('input').focus();

// ✓ Допустимо — для интеграции
const chart = new ThirdPartyChart(this.$['#chart']);
chart.render();

// ✗ Неправильно
this.shadowRoot.querySelector('span').textContent = 'value';
```

---

## Компонент как чёрный ящик

Хороший компонент скрывает внутреннюю реализацию и предоставляет чистый API.

```js
// Внутренняя реализация скрыта
class DataTable extends PlElement {
  // Приватные свойства
  _sortColumn = null;
  _sortDirection = 'asc';

  // Публичный API — только свойства и методы
  static properties = {
    data: { type: Array },
    loading: { type: Boolean },
  };

  sortBy(column) { /* ... */ }
}
```

### Что компонент должен скрывать:
- Внутренние переменные состояния
- Вспомогательные методы
- Детали реализации
- Прямой доступ к DOM

### Что компонент должен предоставлять:
- Публичные свойства
- Понятные методы
- События для коммуникации
- Слоты для расширения

---

## Композиция вместо конфигурации

Создавайте_small_ компоненты и комбинируйте их:

```js
// ✗ Один большой компонент с кучей пропсов
<super-form
  show-title
  show-subtitle
  title-text="Форма"
  subtitle-text="Заполните поля"
  button-text="Отправить"
  button-icon="send"
></super-form>

// ✓ Композиция компонентов
<form-wrapper>
  <form-header>
    <h1 slot="title">Форма</h1>
    <p slot="subtitle">Заполните поля</p>
  </form-header>
  <form-body><!-- контент --></form-body>
  <form-footer>
    <ui-button icon="send">Отправить</ui-button>
  </form-footer>
</form-wrapper>
```

---

## События как контракт

Компонент общается с внешним миром через **события**. Это делает его переиспользуемым.

```js
class SubmitButton extends PlElement {
  onClick() {
    // Событие — это контракт
    this.dispatchEvent(new CustomEvent('submit', {
      bubbles: true,
      composed: true,
      detail: { timestamp: Date.now() }
    }));
  }
}

// Использование — в любом месте
<submit-button on-submit="handleSubmit"></submit-button>
```

---

## Соглашение об именовании

| Что | Формат | Пример |
|-----|--------|--------|
| Тег компонента | kebab-case | `<user-card>`, `<data-table>` |
| Класс компонента | PascalCase | `UserCard`, `DataTable` |
| Свойства | camelCase | `userName`, `isActive` |
| Методы | camelCase | `handleClick`, `sortItems` |
| События | kebab-case | `value-change`, `item-select` |

---

## Резюме

| Принцип | Реализация |
|---------|------------|
| Декларативность | Шаблоны вместо DOM-манипуляций |
| Единый источник | Свойства = состояние |
| Изоляция | Shadow DOM + слоты |
| Коммуникация | События |
| Композиция | Маленькие компоненты |

---

## См. также

- [Свойства и шаблоны](properties-templates.md)
- [События и контекст](../managing-data/context-events.md)
- [Лучшие практики](../guides/best-practices.md)
