# Dependent State with linkedSignal

> Source: https://angular.dev/guide/signals/linked-signal

## Overview

The `linkedSignal` function enables creation of signals whose state is intrinsically linked to other state. Unlike regular signals initialized with default values, `linkedSignal` uses a computation function that automatically updates when its source changes.

## Core Concept

The `linkedSignal` function lets you create a signal to hold some state that is intrinsically linked to some other state.

### Basic Usage Pattern

```typescript
@Component({ /* ... */ })
export class ShippingMethodPicker {
  shippingOptions: Signal<ShippingMethod[]> = getShippingOptions();

  // Initialize selectedOption to the first shipping option
  selectedOption = linkedSignal(() => this.shippingOptions()[0]);

  changeShipping(index: number) {
    this.selectedOption.set(this.shippingOptions()[index]);
  }
}
```

## How It Works

`linkedSignal` operates like `signal` with one key distinction: instead of passing a default value, you provide a computation function (similar to `computed`). When the computation value changes, the `linkedSignal` updates automatically, ensuring the signal always maintains a valid value.

### Reactive Example

```typescript
const shippingOptions = signal(['Ground', 'Air', 'Sea']);
const selectedOption = linkedSignal(() => shippingOptions()[0]);

console.log(selectedOption()); // 'Ground'
selectedOption.set(shippingOptions()[2]);
console.log(selectedOption()); // 'Sea'
shippingOptions.set(['Email', 'Will Call', 'Postal service']);
console.log(selectedOption()); // 'Email'
```

When `shippingOptions` changes, `selectedOption` automatically recomputes to reflect the first item in the updated array.

## Accounting for Previous State

For scenarios requiring knowledge of prior values, `linkedSignal` supports separate `source` and `computation` properties:

```typescript
interface ShippingMethod {
  id: number;
  name: string;
}

@Component({ /* ... */ })
export class ShippingMethodPicker {
  constructor() {
    this.changeShipping(2);
    this.changeShippingOptions();
    console.log(this.selectedOption());
    // {"id":2,"name":"Postal Service"}
  }

  shippingOptions = signal<ShippingMethod[]>([
    {id: 0, name: 'Ground'},
    {id: 1, name: 'Air'},
    {id: 2, name: 'Sea'},
  ]);

  selectedOption = linkedSignal<ShippingMethod[], ShippingMethod>({
    source: this.shippingOptions,
    computation: (newOptions, previous) => {
      // Preserve selection if available, otherwise default to first
      return newOptions.find((opt) => opt.id === previous?.value.id)
        ?? newOptions[0];
    },
  });

  changeShipping(index: number) {
    this.selectedOption.set(this.shippingOptions()[index]);
  }

  changeShippingOptions() {
    this.shippingOptions.set([
      {id: 0, name: 'Email'},
      {id: 1, name: 'Sea'},
      {id: 2, name: 'Postal Service'},
    ]);
  }
}
```

### Source and Computation Parameters

- **`source`**: Any signal (computed, input, or standard signal) that triggers updates
- **`computation`**: Function receiving:
  - New source value
  - `previous` object with `previous.source` and `previous.value` properties

**Important**: Explicit generic type arguments are required when using the `previous` parameter. The first type corresponds to the source, the second to the computation output.

## Custom Equality Comparison

Like all signals, `linkedSignal` accepts custom equality functions for downstream dependency tracking:

```typescript
const activeUser = signal({id: 123, name: 'Morgan', isAdmin: true});

const activeUserEditCopy = linkedSignal(() => activeUser(), {
  equal: (a, b) => a.id === b.id,
});

// Or with separate source and computation:
const activeUserEditCopy = linkedSignal({
  source: activeUser,
  computation: (user) => user,
  equal: (a, b) => a.id === b.id,
});
```

The equality function determines whether downstream dependencies recognize value changes based on custom logic rather than reference equality.
