# Signal Forms
> Source: https://angular.dev/guide/signals/forms
> Source: https://angular.dev/essentials/signal-forms
> Source: https://angular.dev/guide/forms/signals/overview

Angular's Signal Forms manage form state using Angular signals to automatically synchronize data models with UI elements. This is an **experimental API** (as of Angular v21) subject to change in future releases.

## Overview

Signal Forms address three primary challenges in web form development:

1. **Automatic State Synchronization** -- Form data models sync automatically with bound UI fields
2. **Type Safety** -- Fully type-safe schemas with bindings between UI controls and data models
3. **Centralized Validation** -- All validation rules defined in a single validation schema

Signal Forms work best for new applications built with signals. The framework provides production-ready solutions, though reactive forms remain viable for existing applications or when production stability is paramount.

## Prerequisites and Setup

### Requirements

- Angular v21 or higher

### Installation

Import necessary functions from the signals submodule:

```typescript
import {form, FormField, required, email} from '@angular/forms/signals';
```

The `FormField` directive must be included in component imports:

```typescript
@Component({
  imports: [FormField],
})
```

## Core Concepts

### 1. Create a Form Model with `signal()`

Begin by creating a signal containing your form's data structure:

```typescript
interface LoginData {
  email: string;
  password: string;
}

const loginModel = signal<LoginData>({
  email: '',
  password: '',
});
```

### 2. Pass Model to `form()` to Create a FieldTree

The `form()` function transforms your model signal into a "field tree" -- a mirrored object structure enabling dot-notation field access:

```typescript
const loginForm = form(loginModel);

// Access fields directly
loginForm.email;
loginForm.password;
```

### 3. Bind Inputs with `[formField]` Directive

Connect HTML inputs to form fields using the `[formField]` directive for two-way binding:

```html
<input type="email" [formField]="loginForm.email" />
<input type="password" [formField]="loginForm.password" />
```

User input automatically updates the form state. The directive also synchronizes attributes like `required`, `disabled`, and `readonly`.

### 4. Access Field Values with `value()`

Call a field as a function to retrieve its `FieldState` object containing reactive signals:

```typescript
loginForm.email(); // Returns FieldState with value(), valid(), touched()

// Access the current value
const currentEmail = loginForm.email().value();
```

### 5. Update Values with `set()`

Programmatically modify field values using `value.set()`, which updates both the field and underlying model:

```typescript
loginForm.email().value.set('alice@wonderland.com');

// Model signal also updates
console.log(loginModel().email); // 'alice@wonderland.com'
```

## Basic Usage by Input Type

### Text Inputs

```html
<input type="text" [formField]="form.name" />
<input type="email" [formField]="form.email" />
```

### Number Inputs

Automatically converts between strings and numbers:

```html
<input type="number" [formField]="form.age" />
```

### Date and Time

Stores as ISO format strings (`YYYY-MM-DD` for dates, `HH:mm` for time):

```html
<input type="date" [formField]="form.eventDate" />
<input type="time" [formField]="form.eventTime" />
```

Convert to Date objects when needed: `new Date(form.eventDate().value())`

### Textareas

```html
<textarea [formField]="form.message" rows="4"></textarea>
```

### Checkboxes

Single checkboxes bind to boolean values:

```html
<label>
  <input type="checkbox" [formField]="form.agreeToTerms" />
  I agree to the terms
</label>
```

Multiple checkboxes require separate boolean fields for each option.

### Radio Buttons

Radio buttons using the same `[formField]` automatically share a `name` attribute:

```html
<label>
  <input type="radio" value="free" [formField]="form.plan" />
  Free
</label>
<label>
  <input type="radio" value="premium" [formField]="form.plan" />
  Premium
</label>
```

Selecting an option sets the field's value to that radio button's `value` attribute.

### Select Dropdowns

Support both static and dynamic options:

```html
<!-- Static -->
<select [formField]="form.country">
  <option value="">Select a country</option>
  <option value="us">United States</option>
  <option value="ca">Canada</option>
</select>

<!-- Dynamic with @for -->
<select [formField]="form.productId">
  <option value="">Select a product</option>
  @for (product of products; track product.id) {
    <option [value]="product.id">{{ product.name }}</option>
  }
</select>
```

**Note:** Multiple select is not currently supported.

## Validation and State Management

### Adding Validators

Pass a schema function as the second argument to `form()` to configure validation:

```typescript
const loginForm = form(loginModel, (schemaPath) => {
  debounce(schemaPath.email, 500);
  required(schemaPath.email);
  email(schemaPath.email);
});
```

### Common Validators

- `required()` -- Field must have a value
- `email()` -- Validates email format
- `min()` / `max()` -- Number range validation
- `minLength()` / `maxLength()` -- String or collection length
- `pattern()` -- Regex pattern validation

### Custom Error Messages

```typescript
required(schemaPath.email, {message: 'Email is required'});
email(schemaPath.email, {message: 'Please enter a valid email address'});
```

### Field State Signals

Each field exposes reactive state signals:

| Signal | Description |
|--------|-------------|
| `valid()` | Returns true if all validations pass |
| `touched()` | Returns true after user focus and blur |
| `dirty()` | Returns true if user changed the value |
| `disabled()` | Returns true if field is disabled |
| `readonly()` | Returns true if field is readonly |
| `pending()` | Returns true during async validation |
| `errors()` | Array of validation errors with `kind` and `message` |

## Complete Example: Login Form

### Component

```typescript
// app.ts
import {ChangeDetectionStrategy, Component, signal} from '@angular/core';
import {email, form, FormField, required} from '@angular/forms/signals';

interface LoginData {
  email: string;
  password: string;
}

@Component({
  selector: 'app-root',
  templateUrl: 'app.html',
  styleUrl: 'app.css',
  imports: [FormField],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class App {
  loginModel = signal<LoginData>({
    email: '',
    password: '',
  });

  loginForm = form(this.loginModel, (schemaPath) => {
    required(schemaPath.email, {message: 'Email is required'});
    email(schemaPath.email, {message: 'Enter a valid email address'});
    required(schemaPath.password, {message: 'Password is required'});
  });

  onSubmit(event: Event) {
    event.preventDefault();
    const credentials = this.loginModel();
    console.log('Logging in with:', credentials);
  }
}
```

### Template

```html
<!-- app.html -->
<form (submit)="onSubmit($event)">
  <div>
    <label>
      Email:
      <input type="email" [formField]="loginForm.email" />
    </label>
    @if (loginForm.email().touched() && loginForm.email().invalid()) {
      <ul class="error-list">
        @for (error of loginForm.email().errors(); track error) {
          <li>{{ error.message }}</li>
        }
      </ul>
    }
  </div>

  <div>
    <label>
      Password:
      <input type="password" [formField]="loginForm.password" />
    </label>
    @if (loginForm.password().touched() && loginForm.password().invalid()) {
      <div class="error-list">
        @for (error of loginForm.password().errors(); track error) {
          <p>{{ error.message }}</p>
        }
      </div>
    }
  </div>

  <button type="submit">Log In</button>
</form>
```

## Learning Path

The full Signal Forms documentation provides these sequential guides:

- Signal forms essentials
- Form models
- Form model design
- Field state management
- Validation
- Custom controls
- Comparison with other form systems
- Migration guidance from legacy forms

## Key Takeaways

Signal Forms provide a streamlined approach to form management through:

- **Automatic synchronization** between model signals and UI
- **Reactive validation** with built-in and custom validators
- **State tracking** via signals for user interactions
- **Type-safe field access** through mirrored object structures
- **Simplified binding** using the `[formField]` directive

For additional details, consult the in-depth guides on form models, field state management, and validation strategies at the Angular documentation site.
