# Two-way Binding
> Source: https://angular.dev/guide/templates/two-way-binding

## Overview

Two-way binding enables simultaneous value binding into an element while allowing that element to propagate changes back through the binding. This creates synchronized state between a component and its template or between parent and child components.

## Syntax

The two-way binding syntax combines square brackets and parentheses: `[()]`. This merges property binding syntax `[]` with event binding syntax `()`. The Angular community informally calls this "banana-in-a-box" syntax.

## Two-way Binding with Form Controls

Developers commonly use two-way binding to keep component data synchronized with form controls as users interact with them. When a user fills out a text input, the component state updates automatically.

### Example: Text Input Binding

```typescript
import {Component} from '@angular/core';
import {FormsModule} from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `
    <main>
      <h2>Hello {{ firstName }}!</h2>
      <input type="text" [(ngModel)]="firstName" />
    </main>
  `,
})
export class App {
  firstName = 'Ada';
}
```

### Setup Requirements

To use two-way binding with native form controls:

1. Import `FormsModule` from `@angular/forms`
2. Use the `ngModel` directive with two-way binding syntax (e.g., `[(ngModel)]`)
3. Assign the state property you want updated (e.g., `firstName`)

Angular automatically ensures text input updates reflect correctly in component state.

## Two-way Binding Between Components

Component-to-component two-way binding requires more configuration than form elements.

### Complete Example

**Parent Component (app.ts):**
```typescript
import {Component} from '@angular/core';
import {Counter} from './counter';

@Component({
  selector: 'app-root',
  imports: [Counter],
  template: `
    <main>
      <h1>Counter: {{ initialCount }}</h1>
      <app-counter [(count)]="initialCount"></app-counter>
    </main>
  `,
})
export class App {
  initialCount = 18;
}
```

**Child Component (counter.ts):**
```typescript
import {Component, model} from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <button (click)="updateCount(-1)">-</button>
    <span>{{ count() }}</span>
    <button (click)="updateCount(+1)">+</button>
  `,
})
export class Counter {
  count = model<number>(0);
  updateCount(amount: number): void {
    this.count.update((currentCount) => currentCount + amount);
  }
}
```

## Enabling Two-way Binding Between Components

### Child Component Requirements

The child component must contain a `model` property:

```typescript
import {Component, model} from '@angular/core';

@Component({
  /* Omitted for brevity */
})
export class Counter {
  count = model<number>(0);
  updateCount(amount: number): void {
    this.count.update((currentCount) => currentCount + amount);
  }
}
```

### Parent Component Requirements

The parent component must:

1. Wrap the `model` property name in two-way binding syntax
2. Assign a property or signal to the `model` property

```typescript
import {Component} from '@angular/core';
import {Counter} from './counter';

@Component({
  selector: 'app-root',
  imports: [Counter],
  template: `
    <main>
      <app-counter [(count)]="initialCount"></app-counter>
    </main>
  `,
})
export class App {
  initialCount = 18;
}
```

## Key Concepts

- **Model Property**: Created using the `model()` function from `@angular/core`, allowing child components to expose writable signals
- **Update Method**: The `model().update()` method allows child components to modify values while propagating changes to parent components
- **Signal Syntax**: In templates, `model` properties are accessed as functions (e.g., `count()`)
