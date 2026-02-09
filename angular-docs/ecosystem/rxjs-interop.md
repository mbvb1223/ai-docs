# RxJS Interop with Angular Signals
> Source: https://angular.dev/ecosystem/rxjs-interop

## Overview

The `@angular/core/rxjs-interop` package provides APIs for integrating RxJS and Angular signals, enabling developers to bridge reactive paradigms within Angular applications.

## Creating Signals from Observables with `toSignal`

The `toSignal` function transforms an Observable into a signal that tracks its value. It functions similarly to Angular's `async` pipe but offers greater flexibility for use anywhere in an application.

### Basic Usage

```typescript
import {Component} from '@angular/core';
import {AsyncPipe} from '@angular/common';
import {interval} from 'rxjs';
import {toSignal} from '@angular/core/rxjs-interop';

@Component({
  template: `{{ counter() }}`,
})
export class Ticker {
  counterObservable = interval(1000);
  counter = toSignal(this.counterObservable, {initialValue: 0});
}
```

### Key Characteristics

**Immediate Subscription**: Like the `async` pipe, `toSignal` subscribes to the Observable immediately, which may trigger side effects.

**Automatic Cleanup**: The subscription automatically unsubscribes when the component or service is destroyed.

**Important Note**: `toSignal` creates a subscription. You should avoid calling it repeatedly for the same Observable, and instead reuse the signal it returns.

### Injection Context Requirement

`toSignal` operates within an injection context by default (during component/service construction). Manual injector specification is available if needed.

### Managing Initial Values

#### Using `initialValue`

Specify a default value the signal returns before the Observable emits:

```typescript
toSignal(observable, {initialValue: 0})
```

#### Undefined Values

Without an `initialValue`, the signal returns `undefined` until emission, mirroring the `async` pipe's behavior.

#### The `requireSync` Option

For synchronous Observables like `BehaviorSubject`, use `requireSync: true`:

```typescript
toSignal(behaviorSubject, {requireSync: true})
```

This ensures the signal always has a value without requiring an initial value or `undefined` typing.

### Manual Cleanup

Override automatic unsubscription with the `manualCleanup` option for Observables that complete naturally.

### Custom Equality Comparison

Define custom equality logic to prevent redundant updates:

```typescript
@Component()
export class EqualExample {
  temperature$ = interval(1000).pipe(
    map(() => ({temperature: Math.floor(Math.random() * 3) + 20})),
  );

  temperature = toSignal(this.temperature$, {
    initialValue: {temperature: 20},
    equal: (prev, curr) => prev.temperature === curr.temperature,
  });
}
```

When consecutive values are considered equal, the resulting signal does not update. This prevents redundant computations, DOM updates, or effects from re-running unnecessarily.

### Error and Completion Handling

- **Errors**: Thrown when reading the signal
- **Completion**: The signal continues returning the last emitted value

## Creating Observables from Signals with `toObservable`

The `toObservable` utility creates an Observable tracking signal value changes via an internal effect.

### Example Usage

```typescript
import {Component, signal} from '@angular/core';
import {toObservable} from '@angular/core/rxjs-interop';

@Component()
export class SearchResults {
  query: Signal<string> = inject(QueryService).query;
  query$ = toObservable(this.query);
  results$ = this.query$.pipe(
    switchMap((query) => this.http.get('/search?q=' + query))
  );
}
```

### Injection Context

Like `toSignal`, `toObservable` requires an injection context, with manual injector specification available.

### Timing Behavior

`toObservable` uses an effect to track the value of the signal in a `ReplaySubject`. On subscription, the first value (if available) may be emitted synchronously, and all subsequent values will be asynchronous.

Example demonstrating batched updates:

```typescript
const obs$ = toObservable(mySignal);
obs$.subscribe((value) => console.log(value));
mySignal.set(1);
mySignal.set(2);
mySignal.set(3);
```

Only the final value (3) logs, as signals stabilize before emission.

## Using `rxResource` for Async Data

**Status**: Experimental API subject to change before stabilization.

`rxResource` builds on Angular's `resource` function, accepting a `stream` factory returning RxJS Observables instead of standard loaders.

### Implementation

```typescript
import {Component, inject} from '@angular/core';
import {rxResource} from '@angular/core/rxjs-interop';

@Component()
export class UserProfile {
  private userData = inject(MyUserDataClient);
  protected userId = input<string>();

  private userResource = rxResource({
    params: () => ({userId: this.userId()}),
    stream: ({params}) => this.userData.load(params.userId),
  });
}
```

The `stream` property accepts a factory function returning Observables, invoked whenever `params` changes. Otherwise, `rxResource` provides identical APIs and behavior to `resource` for parameters, value access, loading states, and error examination.
