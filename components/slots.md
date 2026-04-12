# Слоты

Слоты позволяют встраивать произвольный контент в компонент — ключевой механизм для создания гибких, переиспользуемых компонентов.

---

## Базовый слот

```js
class CardWrapper extends PlElement {
  static template = html`
    <div class="card">
      <header>Заголовок карточки</header>
      <slot></slot>
    </div>
  `;
}
```

```html
<card-wrapper>
  <p>Этот контент попадёт в слот</p>
</card-wrapper>
```

### Как это работает

```
┌─────────────────────────────┐
│  <card-wrapper>             │
│  ┌───────────────────────┐  │
│  │ <header>...</header>  │  │
│  │ <slot></slot> ◄──────┼──┼── <p>Этот контент...</p>
│  └───────────────────────┘  │
└─────────────────────────────┘
```

---

## Именованные слоты

Для нескольких мест вставки используйте атрибут `name`:

```js
class PageLayout extends PlElement {
  static template = html`
    <header><slot name="header"></slot></header>
    <main><slot></slot></main>
    <footer><slot name="footer"></slot></footer>
  `;
}
```

```html
<page-layout>
  <h1 slot="header">Мой сайт</h1>
  
  <p>Основной контент страницы</p>
  
  <span slot="footer">© 2024</span>
</page-layout>
```

### Правила именованных слотов

| Правило | Пример |
|---------|--------|
| Один безымянный слот | `<slot></slot>` |
| Несколько именованных | `<slot name="header">`, `<slot name="footer">` |
| Fallback-контент | `<slot name="empty">Нет данных</slot>` |

---

## Fallback-контент

Слот показывает fallback, если контент не передан:

```js
static template = html`
  <div class="avatar">
    <slot name="icon">🧑</slot>
  </div>
  <div class="name">
    <slot>Аноним</slot>
  </div>
`;
```

```html
<!-- Будет показан fallback -->
<user-badge></user-badge>

<!-- Будет показан переданный контент -->
<user-badge>
  <span slot="icon">👨‍💻</span>
  <span slot="default">Иван</span>
</user-badge>
```

---

## ::slotted() — стилизация контента

CSS-селектор `::slotted()` позволяет стилизовать контент внутри слота:

```js
static css = css`
  ::slotted(img) {
    width: 48px;
    height: 48px;
    border-radius: 50%;
  }

  ::slotted(a) {
    color: #2196f3;
    text-decoration: none;
  }
`;
```

### Ограничения ::slotted()

| Можно | Нельзя |
|-------|--------|
| Стилизовать | Использовать комбинаторы (`::slotted(img: hover)`) |
| Менять layout | Скрывать элементы (`display: none`) |
| Задавать размеры | Анимировать |

---

## Слоты и события

Слоты — это часть Shadow DOM, но события из Light DOM проходят сквозь:

```js
class InteractiveCard extends PlElement {
  static template = html`
    <div class="card">
      <slot></slot>
    </div>
  `;
}
```

```html
<!-- Клик по кнопке всплывёт -->
<interactive-card>
  <button on-click="handleClick">Нажми меня</button>
</interactive-card>
```

---

## Практический пример: Карточка товара

```js
class ProductCard extends PlElement {
  static css = css`
    :host {
      display: block;
    }
    .card {
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 16px;
    }
    .card-image {
      width: 100%;
    }
    ::slotted(h3) {
      margin: 12px 0 8px;
      font-size: 18px;
    }
    ::slotted(p) {
      color: #666;
      margin: 0;
    }
  `;

  static template = html`
    <div class="card">
      <div class="card-image">
        <slot name="image">
          <img src="/placeholder.png" alt="Placeholder">
        </slot>
      </div>
      <slot></slot>
    </div>
  `;
}
```

```html
<product-card>
  <img slot="image" src="/product.jpg" alt="Товар">
  <h3>Ноутбук MacBook Pro</h3>
  <p>14-дюймовый дисплей, M3 Pro</p>
</product-card>
```

---

## Слоты и условный рендеринг

Для условного отображения слотов используйте CSS:

```js
static css = css`
  :host([empty]) .empty-state {
    display: block;
  }
  :host(:not([empty])) .empty-state {
    display: none;
  }
  slot:not([name]) {
    display: none;
  }
  :host([empty]) slot:not([name]) {
    display: block;
  }
`;

static template = html`
  <div class="content">
    <slot></slot>
    <div class="empty-state">
      <slot name="empty">Пусто</slot>
    </div>
  </div>
`;
```

---

## Доступ к слотам в JavaScript

```js
class SlotComponent extends PlElement {
  connectedCallback() {
    super.connectedCallback();
    
    // Получить все слотты
    const slots = this.shadowRoot.querySelectorAll('slot');
    
    // Слушать изменения
    slots.forEach(slot => {
      slot.addEventListener('slotchange', (e) => {
        const nodes = e.target.assignedNodes({ flatten: true });
        console.log('Слот изменился:', nodes);
      });
    });
  }
}
```

---

## Типичные паттерны

### Обёртка с действием

```js
class ActionCard extends PlElement {
  static template = html`
    <div class="card">
      <slot></slot>
      <div class="actions">
        <slot name="actions"></slot>
      </div>
    </div>
  `;
}
```

### Табы

```js
class TabContainer extends PlElement {
  static template = html`
    <div class="tabs">
      <slot name="tabs"></slot>
    </div>
    <div class="content">
      <slot></slot>
    </div>
  `;
}
```

---

## Чеклист

- [ ] Используйте fallback-контент для всех слотов
- [ ] Группируйте связанный контент через именованные слоты
- [ ] Стилизуйте через `::slotted()` для инкапсуляции
- [ ] Не злоупотребляйте — слишком много слотов усложняет компонент

---

## См. также

- [Shadow DOM и ::part](shadow-dom-parts.md)
- [Стилизация](styling.md)
- [Композиция компонентов](../guides/best-practices.md)
