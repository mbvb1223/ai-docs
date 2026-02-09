# Side Effects with effect()

> Source: https://angular.dev/guide/signals/effect

## Overview

This guide covers Angular's `effect` function and related APIs for handling side effects when working with signals and non-reactive code.

## Effects Fundamentals

An effect is an operation that executes whenever one or more signal values change. Create effects using the `effect()` function:

```typescript
effect(() => {
  console.log(`The current count is: ${count()}`);
});
```

### Key Characteristics

- **Runs at least once** automatically
- **Tracks dependencies dynamically** -- only monitors signals read in the most recent execution
- **Executes asynchronously** during the change detection process

## Appropriate Use Cases

Effects should be your last resort. Prefer `computed()` for derived values and `linkedSignal()` for values that can be both derived and manually set.

Valid use cases include:

- Logging signal values for analytics or debugging
- Synchronizing state with storage (localStorage, sessionStorage, cookies)
- Adding custom DOM behavior beyond template syntax
- Custom rendering to canvas elements or third-party UI libraries

### When NOT to Use Effects

Avoid effects for state propagation, as this can cause `ExpressionChangedAfterItHasBeenChecked` errors, infinite loops, or excessive change detection cycles. Use `computed` signals instead for dependent state.

## Injection Context Requirements

By default, effects require an injection context. Create them within component, directive, or service constructors:

```typescript
@Component({/*...*/})
export class EffectiveCounter {
  readonly count = signal(0);

  constructor() {
    effect(() => {
      console.log(`The count is: ${this.count()}`);
    });
  }
}
```

To create effects outside the constructor, pass an `Injector` via options:

```typescript
@Component({/*...*/})
export class EffectiveCounter {
  readonly count = signal(0);
  private injector = inject(Injector);

  initializeLogging(): void {
    effect(
      () => {
        console.log(`The count is: ${this.count()}`);
      },
      {injector: this.injector},
    );
  }
}
```

## Effect Execution Behavior

Angular distinguishes between two effect types based on creation context:

**View Effects** -- created during component instantiation (including effects in component-scoped services)
- Execute before their component is checked by change detection

**Root Effects** -- created in root-provided services
- Execute before any components are checked

In both cases, if effect dependencies change during execution, the effect reruns before proceeding with change detection.

## Destroying Effects

Angular automatically cleans up effects when components or directives are destroyed.

Effects return an `EffectRef` with a `destroy()` method for manual disposal. Combine this with the `manualCleanup` option to prevent automatic cleanup, but ensure you manually destroy such effects.

## Effect Cleanup Functions

Long-running operations should be cancelled if the effect is destroyed or reruns. Accept an `onCleanup` function as the first parameter:

```typescript
effect((onCleanup) => {
  const user = currentUser();
  const timer = setTimeout(() => {
    console.log(`1 second ago, the user became ${user}`);
  }, 1000);

  onCleanup(() => {
    clearTimeout(timer);
  });
});
```

The cleanup callback runs before the next effect execution or when the effect is destroyed.

## Side Effects on DOM Elements

The `effect()` function runs *before* Angular updates the DOM. For direct DOM manipulation or third-party library integration requiring DOM access, use `afterRenderEffect()`:

```typescript
@Component({/*...*/})
export class MyFancyChart {
  chartData = input.required<ChartData>();
  canvas = viewChild.required<ElementRef<HTMLCanvasElement>>('canvas');
  chart: ChartInstance;

  constructor() {
    afterNextRender({
      write: () => {
        this.chart = initializeChart(
          this.canvas().nativeElement(),
          this.chartData()
        );
      },
    });

    afterRenderEffect(() => {
      this.chart.updateData(this.chartData());
    });
  }
}
```

### Render Phases

Optimize DOM operations using `afterRenderEffect` phases to prevent layout thrashing:

| Phase | Purpose |
|-------|---------|
| `earlyRead` | Read DOM before write operations |
| `write` | Write to DOM only (never read) |
| `mixedReadWrite` | Read and write simultaneously (use as fallback) |
| `read` | Read DOM only (never write) |

```typescript
afterRenderEffect({
  earlyRead: (cleanupFn) => { /* read operations */ },
  write: (previousPhaseValue, cleanupFn) => { /* write operations */ },
  mixedReadWrite: (previousPhaseValue, cleanupFn) => { /* if necessary */ },
  read: (previousPhaseValue, cleanupFn) => { /* read operations */ },
});
```

Phases execute in order (earlyRead -> write -> mixedReadWrite -> read). If a phase modifies a tracked signal, affected phases rerun.

### Important Note

If no phase is specified, `afterRenderEffect` defaults to `mixedReadWrite`, which may harm performance through unnecessary reflows. Always specify appropriate phases.

## Server-Side Rendering Considerations

`afterRenderEffect` only executes on the client. Components may not be hydrated before callbacks run -- use caution when directly accessing or modifying the DOM and layout.

## Preference Alternatives

Prefer `ResizeObserver`, `MutationObserver`, and `IntersectionObserver` APIs when possible instead of effects for DOM change detection.
