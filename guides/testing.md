# Тестирование компонентов

Тестирование Web Components имеет свою специфику. Разберём подходы, инструменты и паттерны.

---

## Почему тестировать сложнее

Особенности Polylib-компонентов:

- **Shadow DOM** — изолированное DOM-дерево
- **Реактивность** — обновления асинхронны
- **Custom Elements** — требуют регистрации в браузере

---

## Инструменты

### Рекомендуемый стек

| Задача | Инструмент |
|--------|------------|
| Тестовый раннер | [Vitest](https://vitest.dev/) |
| Тестирование в браузере | [@web/test-runner](https://modern-web.dev/docs/test-runner/overview/) |
| Утилиты | [@open-wc/testing](https://github.com/open-wc/open-wc/tree/master/packages/testing) |
| Моки | [Sinon](https://sinonjs.org/) |

### Быстрая настройка

```bash
npm init -y
npm install --save-dev @web/test-runner vitest @open-wc/testing sinon
```

---

## Базовые тесты

### Настройка тестового окружения

```js
// test/setup.js
import { html, fixture, expect } from '@open-wc/testing';
import '../src/my-component.js';

describe('MyComponent', () => {
  it('should render', async () => {
    const el = await fixture(html`<my-component></my-component>`);
    expect(el).to.exist;
  });
});
```

### Первый тест компонента

```js
// test/my-counter.test.js
import { html, fixture, expect } from '@open-wc/testing';
import '../src/my-counter.js';

describe('MyCounter', () => {
  it('should have initial count of 0', async () => {
    const el = await fixture(html`<my-counter></my-counter>`);
    expect(el.count).to.equal(0);
  });

  it('should increment count on button click', async () => {
    const el = await fixture(html`<my-counter></my-counter>`);
    el.shadowRoot.querySelector('button').click();

    // Ждём обновления реактивного свойства
    await el.updateComplete;
    expect(el.count).to.equal(1);
  });
});
```

---

## Тестирование свойств

### Установка свойств

```js
describe('UserCard', () => {
  it('should display user name', async () => {
    const el = await fixture(html`
      <user-card name="Иван" role="admin"></user-card>
    `);

    // Проверяем через свойство
    expect(el.name).to.equal('Иван');
    expect(el.role).to.equal('admin');
  });

  it('should update when property changes', async () => {
    const el = await fixture(html`<user-card></user-card>`);

    el.name = 'Мария';
    await el.updateComplete;

    expect(el.name).to.equal('Мария');
  });
});
```

### Тестирование валидации

```js
describe('FormInput validation', () => {
  it('should set error state for invalid email', async () => {
    const el = await fixture(html`
      <form-input
        type="email"
        value="not-an-email"
      ></form-input>
    `);

    expect(el.valid).to.be.false;
    expect(el.error).to.equal('Неверный формат email');
  });

  it('should accept valid email', async () => {
    const el = await fixture(html`
      <form-input
        type="email"
        value="user@example.com"
      ></form-input>
    `);

    expect(el.valid).to.be.true;
    expect(el.error).to.be.null;
  });
});
```

---

## Тестирование событий

### Проверка выброса событий

```js
describe('Button events', () => {
  it('should dispatch click event', async () => {
    const el = await fixture(html`<ui-button>Click me</ui-button>`);

    let clicked = false;
    el.addEventListener('click', () => { clicked = true; });

    el.shadowRoot.querySelector('button').click();

    expect(clicked).to.be.true;
  });

  it('should dispatch custom action event', async () => {
    const el = await fixture(html`<action-button></action-button>`);

    const received = await new Promise(resolve => {
      el.addEventListener('action', (e) => resolve(e.detail));
    });

    el.performAction();

    expect(received).to.deep.equal({ type: 'save' });
  });
});
```

### Использование fake-таймеров

```js
describe('DebouncedInput', () => {
  it('should debounce input events', async () => {
    // Важно: fake-таймеры должны использоваться осторожно
    // с Polylib из-за requestAnimationFrame
    const el = await fixture(html`
      <debounced-input></debounced-input>
    `);

    el.value = 'a';
    el.value = 'ab';
    el.value = 'abc';

    // Фейковые таймеры могут конфликтовать с реактивностью
    // Лучше тестируйте финальное состояние
    await el.updateComplete;

    // Проверяем что debounce сработал
    expect(el.pendingUpdates).to.equal(1);
  });
});
```

---

## Тестирование Shadow DOM

### Доступ к shadow root

```js
describe('Styling integration', () => {
  it('should apply shadow DOM styles', async () => {
    const el = await fixture(html`<styled-box></styled-box>`);

    const box = el.shadowRoot.querySelector('.box');
    const styles = getComputedStyle(box);

    expect(styles.display).to.equal('flex');
  });

  it('should expose parts for styling', async () => {
    const el = await fixture(html`
      <themed-card></themed-card>
    `);

    // CodePen пример позже покажет как это работает
    const part = el.shadowRoot.querySelector('[part="title"]');
    expect(part).to.exist;
  });
});
```

### Тестирование через part

```js
it('should allow part styling', async () => {
  const el = await fixture(html`
    <ui-card>
      <span slot="title">Заголовок</span>
    </ui-card>
  `);

  const title = el.shadowRoot.querySelector('[part="title"]');
  expect(title.textContent).to.equal('Заголовок');
});
```

---

## Тестирование слотов

```js
describe('Slot content', () => {
  it('should render slotted content', async () => {
    const el = await fixture(html`
      <card-wrapper>
        <p>Слот контент</p>
      </card-wrapper>
    `);

    const slot = el.shadowRoot.querySelector('slot');
    const distributed = slot.assignedNodes({ flatten: true });

    expect(distributed.length).to.equal(1);
    expect(distributed[0].textContent).to.contain('Слот контент');
  });
});
```

---

## Моки и стабы

### Мокирование fetch

```js
import sinon from 'sinon';

describe('DataLoader', () => {
  let server;
  let fetchStub;

  beforeEach(() => {
    fetchStub = sinon.stub(window, 'fetch');
  });

  afterEach(() => {
    fetchStub.restore();
  });

  it('should load data from API', async () => {
    const mockData = [{ id: 1, name: 'Test' }];
    fetchStub.resolves(new Response(JSON.stringify(mockData)));

    const el = await fixture(html`<data-loader url="/api/items"></data-loader>`);
    await el.updateComplete;

    expect(el.items).to.deep.equal(mockData);
  });

  it('should handle network error', async () => {
    fetchStub.rejects(new Error('Network error'));

    const el = await fixture(html`<data-loader url="/api/items"></data-loader>`);
    await el.updateComplete;

    expect(el.error).to.equal('Ошибка загрузки данных');
  });
});
```

### Мокирование событий компонента

```js
describe('Event propagation', () => {
  it('should bubble custom events', async () => {
    const el = await fixture(html`<event-emitter></event-emitter>`);

    const received = await new Promise(resolve => {
      el.addEventListener('my-event', (e) => resolve(e.detail));
      el.emit();
    });

    expect(received).to.equal('test');
  });
});
```

---

## Интеграционные тесты

### Тестирование взаимодействия компонентов

```js
describe('Form integration', () => {
  it('should validate and submit', async () => {
    const form = await fixture(html`
      <contact-form>
        <input name="email" value="test@example.com">
        <input name="message" value="Привет">
      </contact-form>
    `);

    const submitBtn = form.shadowRoot.querySelector('button[type="submit"]');

    // Заполняем и отправляем
    submitBtn.click();
    await form.updateComplete;

    // Проверяем состояние после отправки
    expect(form.submitted).to.be.true;
    expect(form.submittedData).to.deep.equal({
      email: 'test@example.com',
      message: 'Привет',
    });
  });
});
```

---

## CI/CD

### Пример GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm test
```

### Web Test Runner config

```js
// web-test-runner.config.js
export default {
  files: 'test/**/*.test.js',
  nodeResolver: true,
  playwright: true,
  playwrightConfig: {
    headless: true,
  },
};
```

---

## Паттерны тестирования

### Arrange-Act-Assert (AAA)

```js
it('should increment counter', async () => {
  // Arrange
  const el = await fixture(html`<counter></counter>`);

  // Act
  el.increment();
  await el.updateComplete;

  // Assert
  expect(el.count).to.equal(1);
});
```

### Given-When-Then

```js
describe('UserList filtering', () => {
  it('should show only active users when filter is "active"', async () => {
    // Given
    const list = await fixture(html`
      <user-list
        .users$="[[users]]"
        filter="active"
      ></user-list>
    `);

    // When
    await list.updateComplete;

    // Then
    const items = list.shadowRoot.querySelectorAll('user-item');
    items.forEach(item => {
      expect(item.active).to.be.true;
    });
  });
});
```

---

## Частые проблемы

| Проблема | Решение |
|----------|---------|
| `updateComplete` never resolves | Проверьте, что свойство действительно меняется |
| Shadow root null | Компонент ещё не connected |
| Событие не всплывает | Используйте `{ bubbles: true, composed: true }` |
| Тесты падают в CI | Добавьте `await page.waitForFunction()` |

---

## Ресурсы

- [Open WC Testing Guide](https://open-wc.org/docs/testing/guide/)
- [Web Test Runner](https://modern-web.dev/docs/test-runner/overview/)
- [Testing Playbook](https://github.com/nicktomlin/playwright-testing)

---

## Playground

Попробуйте примеры интерактивно:

- [Playground с примерами тестов](playground.md)

---

## См. также

- [Лучшие практики](best-practices.md)
- [Производительность](performance.md)
