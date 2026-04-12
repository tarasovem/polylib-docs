# Валидация

Валидация данных в формах — критически важная часть любого приложения. Рассмотрим подходы к валидации в Polylib.

---

## Базовые правила валидации

### Валидация в компоненте

```js
class FormInput extends PlElement {
  static properties = {
    value: { type: String },
    type: { type: String },
    required: { type: Boolean },
    valid: { type: Boolean },
    error: { type: String },
  };

  constructor() {
    super();
    this.valid = true;
    this.error = '';
  }

  validate() {
    // Обязательное поле
    if (this.required && !this.value) {
      this.set('valid', false);
      this.set('error', 'Поле обязательно для заполнения');
      return false;
    }

    // Тип поля
    if (this.value) {
      if (this.type === 'email' && !this.isValidEmail(this.value)) {
        this.set('valid', false);
        this.set('error', 'Неверный формат email');
        return false;
      }

      if (this.type === 'url' && !this.isValidUrl(this.value)) {
        this.set('valid', false);
        this.set('error', 'Неверный формат URL');
        return false;
      }

      if (this.type === 'phone' && !this.isValidPhone(this.value)) {
        this.set('valid', false);
        this.set('error', 'Неверный формат телефона');
        return false;
      }
    }

    this.set('valid', true);
    this.set('error', '');
    return true;
  }

  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  isValidUrl(url) {
    try {
      new URL(url);
      return true;
    } catch {
      return false;
    }
  }

  isValidPhone(phone) {
    return /^\+?[0-9\s\-()]{10,}$/.test(phone);
  }
}
```

---

## Валидация по паттерну

### HTML5 pattern

```js
// Валидация по регулярному выражению
class PatternInput extends PlElement {
  static properties = {
    value: { type: String },
    pattern: { type: String },
    patternMessage: { type: String },
    valid: { type: Boolean },
    error: { type: String },
  };

  validate() {
    if (!this.value || !this.pattern) {
      this.valid = true;
      this.error = '';
      return true;
    }

    const regex = new RegExp(this.pattern);
    this.valid = regex.test(this.value);

    if (!this.valid) {
      this.error = this.patternMessage || 'Неверный формат';
    }

    return this.valid;
  }
}
```

```html
<!-- Использование -->
<pattern-input
  value="{{username}}"
  pattern="^[a-zA-Z0-9_]{3,20}$"
  pattern-message="3-20 символов, только буквы и цифры"
></pattern-input>
```

---

## Валидация форм

### Form Validator

```js
class FormValidator extends PlElement {
  static properties = {
    fields: { type: Object },
    submitted: { type: Boolean },
  };

  constructor() {
    super();
    this.fields = {};
    this.submitted = false;
  }

  registerField(name, validator) {
    this.fields[name] = {
      value: '',
      validator,
      valid: true,
      error: '',
    };
  }

  setFieldValue(name, value) {
    if (this.fields[name]) {
      this.fields[name].value = value;
      if (this.submitted) {
        this.validateField(name);
      }
    }
  }

  validateField(name) {
    const field = this.fields[name];
    if (!field) return true;

    const result = field.validator(field.value);
    field.valid = result.valid;
    field.error = result.error || '';

    return field.valid;
  }

  validateAll() {
    this.submitted = true;
    let allValid = true;

    for (const name in this.fields) {
      if (!this.validateField(name)) {
        allValid = false;
      }
    }

    return allValid;
  }

  getData() {
    const data = {};
    for (const name in this.fields) {
      data[name] = this.fields[name].value;
    }
    return data;
  }
}
```

---

## Валидация в реальном времени

### Debounced валидация

```js
class ValidatedInput extends PlElement {
  static properties = {
    value: { type: String },
    rules: { type: Array },
    valid: { type: Boolean },
    error: { type: String },
  };

  constructor() {
    super();
    this._debounceTimer = null;
  }

  static template = html`
    <div class$="[[valid ? 'valid' : 'invalid']]">
      <input
        type="text"
        value="{{value}}"
        on-input="handleInput"
        on-blur="handleBlur"
      >
      ${when(this.error, () => html`
        <span class="error">[[error]]</span>
      `)}
    </div>
  `;

  handleInput(event) {
    this.value = event.target.value;
    this.debouncedValidate();
  }

  handleBlur() {
    clearTimeout(this._debounceTimer);
    this.validate();
  }

  debouncedValidate() {
    clearTimeout(this._debounceTimer);
    this._debounceTimer = setTimeout(() => {
      this.validate();
    }, 300);
  }

  validate() {
    for (const rule of this.rules) {
      const result = rule(this.value);
      if (!result.valid) {
        this.set('valid', false);
        this.set('error', result.error);
        return false;
      }
    }

    this.set('valid', true);
    this.set('error', '');
    return true;
  }
}
```

---

## Встроенные правила

```js
// Библиотека правил
const rules = {
  required: (value) => ({
    valid: Boolean(value),
    error: 'Поле обязательно',
  }),

  minLength: (min) => (value) => ({
    valid: !value || value.length >= min,
    error: `Минимум ${min} символов`,
  }),

  maxLength: (max) => (value) => ({
    valid: !value || value.length <= max,
    error: `Максимум ${max} символов`,
  }),

  pattern: (regex,message) => (value) => ({
    valid: !value || regex.test(value),
    error: message || 'Неверный формат',
  }),

  email: () => ({
    valid: !value || /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    error: 'Неверный email',
  }),

  numeric: () => ({
    valid: !value || /^\d+$/.test(value),
    error: 'Только цифры',
  }),

  match: (otherField, message) => (value) => ({
    valid: value === otherField,
    error: message || 'Значения не совпадают',
  }),
};
```

---

## Практический пример: Registration Form

```js
class RegistrationForm extends PlElement {
  static properties = {
    email: { type: String },
    password: { type: String },
    confirmPassword: { type: String },
    errors: { type: Object },
  };

  constructor() {
    super();
    this.errors = {};
  }

  static template = html`
    <form on-submit="handleSubmit">
      <validated-input
        label="Email"
        value="{{email}}"
        .rules="[[[rules.required, rules.email]]]"
      ></validated-input>

      <validated-input
        label="Пароль"
        type="password"
        value="{{password}}"
        .rules="[[[rules.required, rules.minLength(8)]]]"
      ></validated-input>

      <validated-input
        label="Подтверждение пароля"
        type="password"
        value="{{confirmPassword}}"
        .rules="[[[rules.required, v => rules.match(password, 'Пароли не совпадают')(v)]]]"
      ></validated-input>

      <button type="submit">Зарегистрироваться</button>
    </form>
  `;

  async handleSubmit(event) {
    event.preventDefault();

    // Валидация всех полей
    const isValid = this.shadowRoot.querySelectorAll('validated-input')
      .every(input => input.validate());

    if (isValid) {
      // Отправка формы
      await this.register();
    }
  }
}
```

---

## Сообщения об ошибках

### Кастомные сообщения

```js
class FormField extends PlElement {
  static properties = {
    label: { type: String },
    value: { type: String },
    error: { type: String },
    hint: { type: String },
  };

  static template = html`
    <div class$="[[error ? 'has-error' : '']]">
      <label>[[label]]</label>
      <slot></slot>
      ${when(hint && !error, () => html`
        <span class="hint">[[hint]]</span>
      `)}
      ${when(error, () => html`
        <span class="error">[[error]]</span>
      `)}
    </div>
  `;
}
```

---

## Чеклист валидации

- [ ] Валидируйте на клиенте и сервере
- [ ] Показывайте понятные сообщения об ошибках
- [ ] Валидируйте в реальном времени (с debounce)
- [ ] Подсвечивайте ошибочные поля
- [ ] Блокируйте отправку невалидной формы

---

## См. также

- [Коллекции](collections.md)
- [События](../managing-data/context-events.md)
- [Лучшие практики](../guides/best-practices.md)
