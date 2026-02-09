# Component Host Elements
> Source: https://angular.dev/guide/components/host-elements

## Overview

Angular automatically instantiates components for every HTML element matching the component's selector. The matching DOM element is called the **host element**, and the component's template content renders inside it.

### Example Structure

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
})
export class ProfilePhoto {}
```

Usage:
```html
<h3>Your profile photo</h3>
<profile-photo></profile-photo>
<button>Upload a new profile photo</button>
```

Rendered result:
```html
<h3>Your profile photo</h3>
<profile-photo>
  <img src="profile-photo.jpg" alt="Your profile photo" />
</profile-photo>
<button>Upload a new profile photo</button>
```

In this example, `<profile-photo>` serves as the host element.

---

## Binding to the Host Element

Components can bind properties, attributes, styles, and events to their host elements using the `host` property in the `@Component` decorator:

```typescript
@Component({
  ...,
  host: {
    'role': 'slider',
    '[attr.aria-valuenow]': 'value',
    '[class.active]': 'isActive()',
    '[style.background]': `hasError() ? 'red' : 'green'`,
    '[tabIndex]': 'disabled ? -1 : 0',
    '(keydown)': 'updateValue($event)',
  },
})
export class CustomSlider {
  value: number = 0;
  disabled: boolean = false;
  isActive = signal(false);
  hasError = signal(false);
  updateValue(event: KeyboardEvent) { /* ... */ }
}
```

**Note:** Global target names for event prefixes include `document:`, `window:`, and `body:`.

---

## @HostBinding and @HostListener Decorators

### @HostBinding

Binds host properties and attributes to class members:

```typescript
@Component({
  /* ... */
})
export class CustomSlider {
  @HostBinding('attr.aria-valuenow')
  value: number = 0;

  @HostBinding('tabIndex')
  get tabIndex() {
    return this.disabled ? -1 : 0;
  }
  /* ... */
}
```

### @HostListener

Binds event listeners to the host element:

```typescript
export class CustomSlider {
  @HostListener('keydown', ['$event'])
  updateValue(event: KeyboardEvent) {
    /* ... */
  }
}
```

**Best Practice:** Always prefer using the `host` property over these decorators, which exist only for backward compatibility.

---

## Binding Collisions

When both template bindings and host bindings target the same property, these rules apply:

- Static bindings on the component instance override static host bindings
- Dynamic bindings override static ones
- When both are dynamic, the component's host binding takes precedence

Example:
```typescript
@Component({
  ...,
  host: {
    'role': 'presentation',
    '[id]': 'id',
  }
})
export class ProfilePhoto { /* ... */ }
```

Usage:
```html
<profile-photo role="group" [id]="otherId" />
```

---

## CSS Custom Properties

Use style bindings to set CSS custom properties on host elements:

```typescript
@Component({
  /* ... */
  host: {
    '[style.--my-background]': 'color()',
  },
})
export class MyComponent {
  color = signal('lightgreen');
}
```

The custom property updates automatically when the signal changes, affecting the component and its children.

### Setting Custom Properties on Child Components

Alternatively, apply style bindings to child component host elements:

```typescript
@Component({
  selector: 'my-component',
  template: `<my-child [style.--my-background]="color()" />`,
})
export class MyComponent {
  color = signal('lightgreen');
}
```

---

## Injecting Host Element Attributes

Components and directives can read static attributes from their host element using `HostAttributeToken` with the `inject` function:

```typescript
import { Component, HostAttributeToken, inject } from '@angular/core';

@Component({
  selector: 'app-button',
  ...,
})
export class Button {
  variation = inject(new HostAttributeToken('variation'));
}
```

Usage:
```html
<app-button variation="primary">Click me</app-button>
```

**Note:** `HostAttributeToken` throws an error if the attribute is missing unless marked as optional injection.
