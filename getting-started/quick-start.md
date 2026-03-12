# Быстрый старт

Эта страница помогает получить первый рабочий результат с Polylib за 10-15 минут.

## 1) Подготовка окружения

Требования:

- современный браузер с поддержкой Web Components;
- обычный `index.html` и ES modules.

## 2) Создайте файл компонента

Создайте `my-counter.js`:

```js
import { PlElement, html, css } from "https://esm.sh/polylib";

export class MyCounter extends PlElement {
  static properties = {
    count: { type: Number },
  };

  static css = css`
    :host {
      display: inline-flex;
      gap: 8px;
      align-items: center;
      font-family: sans-serif;
    }
  `;

  constructor() {
    super();
    this.count = 0;
  }

  onIncrement() {
    this.set("count", this.count + 1);
  }

  render() {
    return html`
      <button on-click="onIncrement">+</button>
      <span>Count: [[count]]</span>
    `;
  }
}

customElements.define("my-counter", MyCounter);
```

## 3) Подключите компонент на странице

```html
<!doctype html>
<html lang="ru">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Polylib Quick Start</title>
  </head>
  <body>
    <my-counter></my-counter>

    <script type="module" src="./my-counter.js"></script>
  </body>
</html>
```

## 4) Что проверить после запуска

- компонент регистрируется без ошибок;
- кнопка обновляет значение счетчика;
- стили из `static css` применяются к компоненту.

## Следующие шаги

- [Что такое Polylib](../components/what-is-polylib.md)
- [Свойства и шаблоны](../components/properties-templates.md)
- [Контекст и события](../components/context-events.md)
