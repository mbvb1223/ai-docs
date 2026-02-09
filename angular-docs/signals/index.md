# Angular Signals

> Source: https://angular.dev/guide/signals

## Overview

A **signal** wraps a value and notifies consumers when changes occur. Signals track usage throughout applications, enabling framework optimization of rendering updates.

## Key Concepts

### Writable Signals

Create writable signals using the `signal()` function:

```typescript
const count = signal(0);
console.log('The count is: ' + count());
```

Update values with `.set()` or `.update()`:

```typescript
count.set(3);
count.update((value) => value + 1);
```

**Type:** `WritableSignal<T>`

### Converting to Read-Only

Use `asReadonly()` to expose signals without modification capability:

```typescript
@Injectable({providedIn: 'root'})
export class CounterState {
  private readonly _count = signal(0);
  readonly count = this._count.asReadonly();

  increment() {
    this._count.update((v) => v + 1);
  }
}
```

**Important:** Read-only signals don't prevent deep mutations of their values.

### Computed Signals

Derive read-only signals from other signals using `computed()`:

```typescript
const count = signal(0);
const doubleCount = computed(() => count() * 2);
```

**Characteristics:**
- Lazily evaluated -- computation runs only on first read
- Memoized -- cached values returned on subsequent reads
- Not writable -- cannot use `.set()` or `.update()`
- Dynamic dependencies -- only tracked signals actually read are monitored

#### Dynamic Dependencies Example

```typescript
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
  if (showCount()) {
    return `The count is ${count()}.`;
  } else {
    return 'Nothing to see here!';
  }
});
```

When `showCount` is false, `count()` isn't read, so changes to `count` won't trigger recomputation.

## Reactive Contexts

A **reactive context** monitors signal reads to establish dependencies. Angular automatically enters reactive contexts during:

- `effect()` or `afterRenderEffect()` callbacks
- `computed()` evaluation
- `linkedSignal` evaluation
- `resource()` params or loader functions
- Component template rendering (including host bindings)

### Asserting Non-Reactive Context

Use `assertNotInReactiveContext()` to ensure code isn't executing within a reactive context:

```typescript
import {assertNotInReactiveContext} from '@angular/core';

function subscribeToEvents() {
  assertNotInReactiveContext(subscribeToEvents);
  // Safe to proceed
}
```

### Reading Without Tracking

Prevent signal reads from creating dependencies using `untracked()`:

```typescript
effect(() => {
  console.log(`User: ${currentUser()} and counter: ${untracked(counter)}`);
});
```

Also useful for external code that shouldn't become a dependency:

```typescript
effect(() => {
  const user = currentUser();
  untracked(() => {
    this.loggingService.log(`User set to ${user}`);
  });
});
```

## Advanced Topics

### Signal Equality Functions

Provide custom equality checking:

```typescript
import _ from 'lodash';

const data = signal(['test'], {equal: _.isEqual});
data.set(['test']); // No update triggered due to deep equality
```

**Default:** `Object.is()` referential equality

### Type Checking

Check if a value is a signal:

```typescript
const count = signal(0);
const doubled = computed(() => count() * 2);

isSignal(count);      // true
isSignal(doubled);    // true
isSignal(42);         // false

isWritableSignal(count);    // true
isWritableSignal(doubled);  // false
```

## Related Topics

- **Dependent state:** See `linkedSignal` guide for writable state derived from signals
- **Async operations:** Use `resource()` for incorporating async data while maintaining synchronous access
- **Side effects:** Use `effect()` or `afterRenderEffect()` for non-reactive API reactions
- **OnPush components:** Signals automatically mark components for updates when changed

## OnPush Component Integration

Reading signals in `OnPush` components automatically tracks them as dependencies. Angular marks components for update when signal values change.

## RxJS Interoperability

See dedicated documentation on RxJS interop with Angular signals for stream integration patterns.
