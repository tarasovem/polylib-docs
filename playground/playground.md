# Playground

Интерактивные примеры для изучения Polylib. Каждый пример можно редактировать и сразу видеть результат.

> **CodePen** выбран как платформа для интерактивных примеров, так как он работает в России без ограничений.

---

## Базовые примеры

### 1. Минимальный компонент

Создайте свой первый Web Component с нуля:

<p class="codepen" data-height="400" data-theme-id="dark" data-default-tab="result" data-slug-hash="xxjWZvZ" data-preview="true" data-editable="true" style="height: 400px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Минимальный компонент">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/xxjWZvZ">
  Polylib: Минимальный компонент</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- Создание класса компонента
- Наследование от `PlElement`
- Регистрация Custom Element
- Базовый шаблон

---

### 2. Реактивные свойства

Научитесь работать со свойствами:

<p class="codepen" data-height="450" data-theme-id="dark" data-default-tab="result" data-slug-hash="GRwbLvQ" data-preview="true" data-editable="true" style="height: 450px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Реактивные свойства">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/GRwbLvQ">
  Polylib: Реактивные свойства</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- Объявление `static properties`
- Типы данных (String, Number, Boolean)
- Реактивное обновление UI
- Метод `this.set()`

---

### 3. Обработка событий

События и взаимодействие с пользователем:

<p class="codepen" data-height="400" data-theme-id="dark" data-default-tab="result" data-slug-hash="bGMbBvp" data-preview="true" data-editable="true" style="height: 400px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Обработка событий">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/bGMbBvp">
  Polylib: Обработка событий</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- Директива `on-click`
- Методы-обработчики
- `this.dispatchEvent()`
- Передача данных через события

---

### 4. Слоты и композиция

Используйте слоты для создания составных компонентов:

<p class="codepen" data-height="450" data-theme-id="dark" data-default-tab="result" data-slug-hash="XWapLrv" data-preview="true" data-editable="true" style="height: 450px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Слоты">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/XWapLrv">
  Polylib: Слоты</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- Элемент `<slot>`
- Именованные слоты
- Fallback-контент
- Композиция компонентов

---

### 5. Условный рендеринг

Отображение контента по условию:

<p class="codepen" data-height="400" data-theme-id="dark" data-default-tab="result" data-slug-hash="xxjWZvK" data-preview="true" data-editable="true" style="height: 400px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Условный рендеринг">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/xxjWZvK">
  Polylib: Условный рендеринг</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- Директива `?` для атрибутов
- Условия в шаблоне
- Ленивый рендеринг

---

### 6. Списки и repeat

Рендеринг коллекций:

<p class="codepen" data-height="500" data-theme-id="dark" data-default-tab="result" data-slug-hash="poWbQvB" data-preview="true" data-editable="true" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Списки">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/poWbQvB">
  Polylib: Списки</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- Директива `repeat`
- Рендеринг массивов
- Ключи элементов

---

## Продвинутые примеры

### 7. Shadow DOM и стилизация

Инкапсуляция стилей:

<p class="codepen" data-height="450" data-theme-id="dark" data-default-tab="result" data-slug-hash="ZEMywdV" data-preview="true" data-editable="true" style="height: 450px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Shadow DOM стили">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/ZEMywdV">
  Polylib: Shadow DOM стили</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- `static css`
- Псевдоклассы `:host`
- CSS custom properties
- Инкапсуляция стилей

---

### 8. Кастомные события

Обмен данными между компонентами:

<p class="codepen" data-height="400" data-theme-id="dark" data-default-tab="result" data-slug-hash="ExjPQBd" data-preview="true" data-editable="true" style="height: 400px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Кастомные события">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/ExjPQBd">
  Polylib: Кастомные события</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

**Что изучаем:**
- `CustomEvent`
- Опции `{ bubbles, composed }`
- Двусторонняя связь

---

## Свои эксперименты

### Шаблон для старта

Хотите поэкспериментировать сами? Вот готовый шаблон:

<p class="codepen" data-height="500" data-theme-id="dark" data-default-tab="result" data-slug-hash="JjLzbvd" data-preview="true" data-editable="true" style="height: 500px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Polylib: Пустой шаблон">
  <span>See the Pen <a href="https://codepen.io/polylib/pen/JjLzbvd">
  Polylib: Пустой шаблон</a> by Polylib (<a href="https://codepen.io/polylib">@polylib</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

### Идеи для экспериментов

Попробуйте реализовать:

1. **Счётчик с лимитом** — не даёт превысить максимальное значение
2. **Переключатель темы** — светлая/тёмная с CSS variables
3. **Сворачиваемая панель** — accordion с анимацией
4. **Список с удалением** — items с кнопкой delete
5. **Форма валидации** — email, required, pattern

---

## Как пользоваться

1. **Редактируйте код** — переключитесь на вкладку HTML/CSS/JS
2. **Смотрите результат** — вкладка "Result" показывает live preview
3. **Экспериментируйте** — все примеры можно изменять
4. **Сохраняйте копию** — кнопка "Save" создаст вашу версию

---

## Полезные ресурсы

- [CodePen](https://codepen.io/) — платформа для экспериментов
- [Polylib на GitHub](https://github.com/) — исходный код библиотеки
- [MDN Web Components](https://developer.mozilla.org/ru/docs/Web/Web_Components) — документация по Web Components

---

## Следующие шаги

- [Быстрый старт](../getting-started/quick-start.md)
- [Свойства и шаблоны](../components/properties-templates.md)
- [Лучшие практики](../guides/best-practices.md)
- [Тестирование](../guides/testing.md)

<!-- CodePen Embed Script -->
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>
