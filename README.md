# Что такое Polylib

Polylib - библиотека для разработки Web Components с декларативными шаблонами, реактивными свойствами и нативной моделью компонентов платформы.

## Что можно строить с Polylib

С Polylib удобно создавать:

- переиспользуемые UI-компоненты;
- библиотеки компонентов и дизайн-системы;
- встраиваемые виджеты для проектов с разным стеком;
- интерфейсы, где важна нативная совместимость Web Components.

## Почему выбирают Polylib

- **Нативный фундамент**: каждый компонент - стандартный Custom Element.
- **Декларативный рендер**: UI описывается через `html`-шаблоны.
- **Реактивные свойства**: обновления интерфейса зависят от состояния компонента.
- **Изолированные стили**: `css`, `:host`, `::part`, `::slotted`.
- **Без обязательной сборки**: можно начать с `index.html` и ES modules.

## Минимальный пример

```js
import { PlElement, html } from "polylib";

class HelloCard extends PlElement {
  static properties = {
    name: {
      type: String,
      value: "Polylib",
    },
  };

  static template = html`
    <h2>Hello, [[name]]</h2>
  `;
}

customElements.define("hello-card", HelloCard);
```

```html
<hello-card></hello-card>
```

## Требования

- базовое знание HTML, CSS, JavaScript;
- понимание Web Components на уровне общих принципов.

## Путь изучения

### Быстрый путь

1. [Быстрый старт](getting-started/quick-start.md)
2. [Свойства и шаблоны](components/properties-templates.md)
3. [Стилизация](components/styling.md)
4. [Контекст и события](components/context-events.md)

### Полный путь

1. [Компоненты](components/what-is-polylib.md)
2. [Шаблоны](templates/data-binding.md)
3. [Работа с данными](managing-data/data-manipulation.md)
4. [Формы](forms/collections.md)
5. [Справочник API](api/pl-element.md)
6. [Руководства](guides/performance.md)
7. [Экосистема](ecosystem/integration.md)

## Ключевые концепции

- **Свойства**: публичный API и состояние компонента.
- **Шаблоны**: декларативное описание DOM в зависимости от состояния.
- **События**: обмен действиями между компонентами и окружением.
- **Slots и Shadow DOM**: композиция и контроль инкапсуляции.
- **Parts и CSS custom properties**: управляемая стилизация извне.

## Далее

- [Открыть быстрый старт](getting-started/quick-start.md)
- [Изучить компонентную модель](components/what-is-polylib.md)
- [Перейти к справочнику API](api/pl-element.md)
