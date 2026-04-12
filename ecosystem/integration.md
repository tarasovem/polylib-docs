# Интеграция

Polylib компоненты можно интегрировать с различными фреймворками и инструментами.

---

## React

### Использование Web Components в React

```jsx
import React from 'react';

// Используйте lowercase для web components
function MyComponent() {
  const handleEvent = (e) => {
    console.log('Event received:', e.detail);
  };

  return (
    <my-component
      on-my-event={handleEvent}
      prop-value="test"
    />
  );
}
```

### Проблема: передача объектов

React не поддерживает передачу объектов через свойства напрямую:

```jsx
// Это не будет работать
<my-component .items={[{id: 1}]} />

// Решение: используйте JSON
<my-component items={JSON.stringify([{id: 1}])} />
```

### Typed React Wrapper

```tsx
// MyComponent.tsx
import React, { createElement } from 'react';

type MyComponentProps = {
  title?: string;
  items?: Item[];
  onSelect?: (item: Item) => void;
};

export const MyComponent: React.FC<MyComponentProps> = ({
  title,
  items,
  onSelect,
}) => {
  const handleSelect = (e: CustomEvent) => {
    onSelect?.(e.detail);
  };

  return createElement('my-component', {
    'on-select': handleSelect,
    title,
    items: JSON.stringify(items),
  });
};
```

---

## Vue

### Vue 3

```vue
<template>
  <my-component
    :title="title"
    :items="items"
    @select="handleSelect"
  />
</template>

<script setup>
import { ref } from 'vue';

const title = ref('Hello');
const items = ref([{ id: 1, name: 'Item 1' }]);

const handleSelect = (event) => {
  console.log('Selected:', event.detail);
};
</script>
```

### Vue 2

```vue
<template>
  <my-component
    :title="title"
    :items="items"
    @select="handleSelect"
  />
</template>

<script>
export default {
  data() {
    return {
      title: 'Hello',
      items: [{ id: 1, name: 'Item 1' }],
    };
  },
  methods: {
    handleSelect(event) {
      console.log('Selected:', event.detail);
    },
  },
};
</script>
```

---

## Angular

### module.ts

```ts
import { CUSTOM_ELEMENTS_SCHEMA, NgModule } from '@angular/core';

@NgModule({
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  // ...
})
export class MyModule {}
```

### component.ts

```ts
import { Component, ElementRef, ViewChild, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-wrapper',
  template: `<my-component #myComp></my-component>`,
})
export class WrapperComponent implements AfterViewInit {
  @ViewChild('myComp') myComp!: ElementRef;

  ngAfterViewInit() {
    // Angular понимает Web Components
    const comp = this.myComp.nativeElement;
    comp.addEventListener('select', (e) => console.log(e.detail));
  }
}
```

---

## Svelte

```svelte
<script>
  import { onMount } from 'svelte';

  let myComponent;

  onMount(() => {
    myComponent.addEventListener('select', (e) => {
      console.log('Selected:', e.detail);
    });
  });
</script>

<my-component bind:this={myComponent} title="Hello" />
```

---

## Сборка

### Vite

```js
// vite.config.js
export default {
  build: {
    target: 'esnext',
  },
  resolve: {
    alias: {
      polylib: '/node_modules/polylib/src/index.js',
    },
  },
};
```

### Rollup

```js
// rollup.config.js
export default {
  output: {
    format: 'esm',
  },
  plugins: [
    resolve(),
    terser(),
  ],
};
```

### Webpack

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
    ],
  },
};
```

---

## TypeScript

### Declaration file

```ts
// polylib.d.ts
declare module 'polylib' {
  export class PlElement extends HTMLElement {
    static properties: Record<string, PropertyDeclaration>;
    static css: TemplateStringsArray;
    static template: ReturnType<typeof html>;

    constructor();
    connectedCallback(): void;
    disconnectedCallback(): void;
    updated(changedProperties: Map<string, unknown>): void;
    updateComplete: Promise<boolean>;

    set<K extends keyof this>(name: K, value: this[K]): void;
    get<K extends keyof this>(name: K): this[K];
    push<K extends keyof this>(name: K, ...items: any[]): void;
    splice<K extends keyof this>(name: K, start: number, deleteCount?: number, ...items: any[]): void;
  }

  export function html(strings: TemplateStringsArray, ...values: any[]): any;
  export function css(strings: TemplateStringsArray, ...values: any[]): any;
  // ... другие экспорты
}
```

---

## SSR (Server-Side Rendering)

### Базовый SSR

Web Components требуют браузерного окружения. Для SSR:

```js
// Серверная обёртка
async function renderPolylib(html) {
  // На сервере: создаём placeholder
  return `
    <div class="web-component-placeholder" data-component="${html}">
      <noscript>${html}</noscript>
    </div>
  `;
}
```

### Next.js

```jsx
// components/MyComponent.tsx
'use client';

import { useEffect, useRef } from 'react';

export function MyComponent(props) {
  const ref = useRef();

  useEffect(() => {
    // Компонент инициализируется на клиенте
  }, []);

  return <my-component ref={ref} {...props} />;
}
```

---

## Тестирование с Jest

```js
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/setupTests.js'],
  transform: {
    '^.+\\.js$': 'babel-jest',
  },
};
```

```js
// setupTests.js
import '@testing-library/jest-dom';

// Мок для Polylib
jest.mock('polylib', () => ({
  html: (strings) => strings,
  css: (strings) => strings,
}));
```

---

## Линтинг

### ESLint

```json
{
  "extends": ["eslint:recommended"],
  "rules": {
    "no-unused-vars": "warn",
    "prefer-const": "error"
  }
}
```

---

## Чеклист интеграции

- [ ] Настройте сборку на ESNext
- [ ] Добавьте TypeScript declarations
- [ ] Протестируйте в целевом фреймворке
- [ ] Документируйте ограничения

---

## Ресурсы

- [Web Components in React](https://custom-elements-everywhere.com/)
- [StencilJS React Integration](https://stenciljs.com/docs/react)
- [Lit React Integration](https://lit.dev/docs/frameworks/react/)

---

## См. также

- [Компоненты @plcmp](components.md)
- [Лучшие практики](../guides/best-practices.md)
