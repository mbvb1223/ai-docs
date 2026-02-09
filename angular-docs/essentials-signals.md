# Reactivity with Signals
> Source: https://angular.dev/essentials/signals

## Overview

Angular employs **signals** as a mechanism for creating and managing state. A signal functions as a lightweight wrapper encapsulating a value.

## Creating Signals

Use the `signal()` function to establish a signal for managing local state:

```typescript
import { signal } from '@angular/core';

// Create a signal with the `signal` function.
const firstName = signal('Morgan');

// Read a signal value by calling it -- signals are functions.
console.log(firstName());

// Change the value of this signal by calling its `set` method with a new value.
firstName.set('Jaime');

// You can also use the `update` method to change the value
// based on the previous value.
firstName.update((name) => name.toUpperCase());
```

## Reactivity

Angular monitors signal read locations and update occurrences. The framework leverages this information to perform additional operations, including DOM updates reflecting new state. This capacity to respond to shifting signal values across time is termed **reactivity**.

## Computed Expressions

A `computed` signal generates its value based on other signals:

```typescript
import { signal, computed } from '@angular/core';

const firstName = signal('Morgan');
const firstNameCapitalized = computed(() => firstName().toUpperCase());
console.log(firstNameCapitalized()); // MORGAN
```

Computed signals are **read-only** -- they lack `set` or `update` methods. The computed signal's value automatically updates when any of its dependency signals change:

```typescript
import { signal, computed } from '@angular/core';

const firstName = signal('Morgan');
const firstNameCapitalized = computed(() => firstName().toUpperCase());
console.log(firstNameCapitalized()); // MORGAN
firstName.set('Jaime');
console.log(firstNameCapitalized()); // JAIME
```

## Using Signals in Components

Incorporate `signal` and `computed` within components to establish and control state:

```typescript
@Component({
  /* ... */
})
export class UserProfile {
  isTrial = signal(false);
  isTrialExpired = signal(false);
  showTrialDuration = computed(() => this.isTrial() && !this.isTrialExpired());

  activateTrial() {
    this.isTrial.set(true);
  }
}
```

**Tip:** For comprehensive coverage of Angular Signals, consult the in-depth Signals guide.

## Next Steps

With knowledge of declaring and managing reactive data, progress to learning how to apply that data within templates.
