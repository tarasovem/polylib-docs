# Компоненты @plcmp

Экосистема готовых компонентов для быстрой разработки.

---

## Установка

```bash
npm install @plcmp/components
```

### Через CDN

```html
<script type="module" src="https://esm.sh/@plcmp/components"></script>
```

---

## Быстрый старт

```js
import '@plcmp/components';

// Теперь доступны все компоненты
// <ui-button>, <ui-input>, <ui-card> и т.д.
```

---

## UI компоненты

### ui-button

Кнопка с вариантами:

```html
<ui-button>Простая кнопка</ui-button>
<ui-button variant="primary">Primary</ui-button>
<ui-button variant="secondary">Secondary</ui-button>
<ui-button variant="danger">Danger</ui-button>
<ui-button disabled>Disabled</ui-button>
<ui-button loading>Загрузка...</ui-button>
```

```js
// События
document.querySelector('ui-button').addEventListener('click', () => {
  console.log('Clicked!');
});
```

### ui-input

Поле ввода:

```html
<ui-input
  label="Имя"
  placeholder="Введите имя"
  value="{{name}}"
></ui-input>

<ui-input
  type="email"
  label="Email"
  error="Неверный email"
></ui-input>

<ui-input
  type="password"
  label="Пароль"
  hint="Минимум 8 символов"
></ui-input>
```

### ui-card

Карточка:

```html
<ui-card>
  <h3 slot="header">Заголовок</h3>
  <p>Контент карточки</p>
  <ui-button slot="footer">Действие</ui-button>
</ui-card>
```

---

## Компоненты форм

### ui-form

Контейнер формы:

```html
<ui-form on-submit="handleSubmit">
  <ui-input name="email" label="Email" required></ui-input>
  <ui-input name="password" type="password" label="Пароль" required></ui-input>
  <ui-button type="submit">Войти</ui-button>
</ui-form>
```

### ui-select

Выпадающий список:

```html
<ui-select
  label="Выберите цвет"
  .options="[[colorOptions]]"
  value="{{selectedColor}}"
></ui-select>
```

```js
// colorOptions = [
//   { value: 'red', label: 'Красный' },
//   { value: 'blue', label: 'Синий' },
//   { value: 'green', label: 'Зелёный' },
// ]
```

### ui-checkbox

Чекбокс:

```html
<ui-checkbox
  label="Согласен с условиями"
  checked="{{agreed}}"
></ui-checkbox>
```

### ui-switch

Переключатель:

```html
<ui-switch
  label="Тёмная тема"
  checked="{{darkMode}}"
></ui-switch>
```

---

## Компоненты навигации

### ui-tabs

Вкладки:

```html
<ui-tabs selected="{{activeTab}}">
  <ui-tab name="tab1">Вкладка 1</ui-tab>
  <ui-tab name="tab2">Вкладка 2</ui-tab>
  <ui-tab name="tab3">Вкладка 3</ui-tab>
</ui-tabs>

<div>Контент для [[activeTab]]</div>
```

### ui-breadcrumbs

Хлебные крошки:

```html
<ui-breadcrumbs>
  <a href="/">Главная</a>
  <a href="/products">Товары</a>
  <span>Текущая страница</span>
</ui-breadcrumbs>
```

---

## Компоненты отображения

### ui-badge

Бейдж:

```html
<ui-badge>Новый</ui-badge>
<ui-badge variant="success">Успех</ui-badge>
<ui-badge variant="warning">Внимание</ui-badge>
<ui-badge variant="error">Ошибка</ui-badge>
```

### ui-spinner

Спиннер загрузки:

```html
<ui-spinner></ui-spinner>
<ui-spinner size="large"></ui-spinner>
```

### ui-empty

Пустое состояние:

```html
<ui-empty
  icon="📭"
  title="Нет данных"
  description="Добавьте первый элемент"
>
  <ui-button>Добавить</ui-button>
</ui-empty>
```

### ui-skeleton

Скелетон загрузки:

```html
<ui-skeleton type="text"></ui-skeleton>
<ui-skeleton type="avatar"></ui-skeleton>
<ui-skeleton type="card"></ui-skeleton>
```

---

## Компоненты данных

### ui-table

Таблица:

```html
<ui-table
  .columns="[[columns]]"
  .data="[[tableData]]"
  selectable
></ui-table>
```

```js
// columns = [
//   { key: 'name', label: 'Имя', sortable: true },
//   { key: 'email', label: 'Email' },
//   { key: 'status', label: 'Статус', render: (val) => html`<ui-badge>${val}</ui-badge>` },
// ]
```

### ui-pagination

Пагинация:

```html
<ui-pagination
  total="100"
  page-size="10"
  current="{{currentPage}}"
></ui-pagination>
```

---

## Утилиты

### useTheme

Хук для работы с темой:

```js
import { useTheme } from '@plcmp/components';

class MyComponent extends PlElement {
  constructor() {
    super();
    const { theme, setTheme, toggleTheme } = useTheme(this);
    this.theme = theme;
    this.setTheme = setTheme;
  }
}
```

### useI18n

Хук для интернационализации:

```js
import { useI18n } from '@plcmp/components';

class MyComponent extends PlElement {
  constructor() {
    super();
    const { t, setLocale } = useI18n(this, 'ru');
    this.t = t;
  }
}

// В шаблоне
html`<p>${this.t('hello', { name: 'Мир' })}</p>`;
```

---

## Theming

### CSS Custom Properties

Все компоненты поддерживают кастомизацию через CSS-переменные:

```css
:root {
  /* Кнопки */
  --ui-button-bg: #2196f3;
  --ui-button-color: white;
  --ui-button-radius: 4px;

  /* Ввод */
  --ui-input-border: #ddd;
  --ui-input-focus: #2196f3;

  /* Карточки */
  --ui-card-bg: white;
  --ui-card-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
```

---

## Вклад в экосистему

Хотите добавить компонент?

1. Fork репозитория
2. Создайте компонент в `src/components/`
3. Добавьте тесты
4. Откройте PR

---

## Ресурсы

- [GitHub репозиторий](https://github.com/plcmp/components)
- [Storybook](https://storybook.plcmp.dev)
- [npm пакет](https://www.npmjs.com/package/@plcmp/components)

---

## См. также

- [Интеграция](integration.md)
- [Лучшие практики](../guides/best-practices.md)
