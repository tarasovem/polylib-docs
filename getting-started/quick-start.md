# Быстрый старт

За 5-10 минут вы соберете первый компонент на Polylib и проверите, что реактивность и стили работают.

## Что вы получите

- зарегистрированный `Custom Element`;
- реактивное свойство `count` с обновлением по клику;
- стили компонента через `static css`.

## Требования

- современный браузер с поддержкой Web Components;
- обычный `index.html` и ES modules.

## Вариант запуска

Эта документация использует сценарий без сборки, через CDN-импорт:

```js
import { PlElement, html, css } from "https://esm.sh/polylib";
```

## Шаг 1. Создайте компонент

Создайте файл `my-counter.js`:

```js
import { PlElement, html, css } from "https://esm.sh/polylib";

export class MyCounter extends PlElement {
  static properties = {
    count: {
      type: Number,
      value: 0,
    },
  };

  static css = css`
    :host {
      display: inline-flex;
      gap: 8px;
      align-items: center;
      font-family: sans-serif;
    }
  `;

  onIncrement() {
    this.set("count", this.count + 1);
  }

  static template = html`
    <button on-click="onIncrement">+</button>
    <span>Count: [[count]]</span>
  `;
}

customElements.define("my-counter", MyCounter);
```

## Шаг 2. Подключите компонент на странице

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

## Шаг 3. Проверьте результат

- компонент рендерится без ошибок в консоли;
- по клику на `+` значение `count` увеличивается;
- стили из `static css` применяются к кнопке и тексту.

## Типичные ошибки на старте

- неверный путь импорта (`https://esm.sh/polylib`);
- забыли `customElements.define(...)`;
- имя тега в HTML не совпадает с зарегистрированным (`my-counter`).

## Следующие шаги

- [Что такое Polylib](../README.md)
- [Свойства и шаблоны](../components/properties-templates.md)
- [Контекст и события](../components/context-events.md)
