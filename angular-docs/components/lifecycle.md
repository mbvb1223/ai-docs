# Component Lifecycle
> Source: https://angular.dev/guide/components/lifecycle

## Overview

A component's **lifecycle** encompasses the sequence of steps from creation through destruction. Angular provides **lifecycle hooks** -- methods and functions you can implement to execute code at specific stages of this process.

## Lifecycle Phases

### 1. Creation Phase

- **`constructor`**: Standard JavaScript class constructor that runs when Angular instantiates the component.

### 2. Change Detection Phase

Multiple hooks execute during Angular's change detection traversal:

- **`ngOnChanges`**: Executes after any component inputs change. During initialization, it runs before `ngOnInit`. Accepts a `SimpleChanges` argument containing previous and current values for each input.

- **`ngOnInit`**: Runs exactly once after Angular initializes all component inputs. Executes before the component's template is initialized, allowing you to update state based on initial input values.

- **`ngDoCheck`**: Runs before every change detection cycle. Use for manually detecting state changes outside Angular's normal detection mechanism. Avoid when possible due to performance impact.

- **`ngAfterContentInit`**: Executes once after all child content nested in the component (its "content") has been initialized. Useful for reading content query results.

- **`ngAfterContentChecked`**: Runs every time content has been checked for changes. Avoid defining when possible due to frequency of execution.

- **`ngAfterViewInit`**: Executes once after the component's template children (its "view") have been initialized. Ideal for accessing view query results.

- **`ngAfterViewChecked`**: Runs after every view check cycle. Use sparingly due to performance implications.

### 3. Rendering Phase

These are standalone functions rather than class methods:

- **`afterNextRender`**: Callback executes once after all components render to the DOM. Must be called within an injection context (typically in a constructor).

- **`afterEveryRender`**: Callback runs every time all components complete rendering to the DOM. Also requires an injection context.

Both support **phases** for sequencing DOM operations:
- `earlyRead`: Read layout-affecting properties strictly necessary for calculations
- `write`: Write layout-affecting properties and styles
- `mixedReadWrite`: Default phase for operations requiring both reads and writes
- `read`: Read layout-affecting properties after writes complete

### 4. Destruction Phase

- **`ngOnDestroy`**: Executes once before the component is destroyed (removed from the DOM).

- **`DestroyRef`**: Alternative injection-based approach. Call `inject(DestroyRef).onDestroy()` to register cleanup callbacks. The `destroyed` property prevents operations on destroyed components.

## Key Implementation Details

### SimpleChanges Usage

```typescript
@Component({/*...*/})
export class UserProfile {
  name = input('');
  ngOnChanges(changes: SimpleChanges<UserProfile>) {
    if (changes.name) {
      console.log(`Previous: ${changes.name.previousValue}`);
      console.log(`Current: ${changes.name.currentValue}`);
      console.log(`Is first ${changes.name.firstChange}`);
    }
  }
}
```

### DestroyRef Pattern

```typescript
@Component({/*...*/})
export class UserProfile {
  constructor() {
    inject(DestroyRef).onDestroy(() => {
      console.log('UserProfile destruction');
    });
  }
}
```

### Render Callbacks with Phases

```typescript
import {Component, ElementRef, afterNextRender} from '@angular/core';

@Component({/*...*/})
export class UserProfile {
  private prevPadding = 0;
  private elementHeight = 0;

  constructor() {
    const elementRef = inject(ElementRef);
    const nativeElement = elementRef.nativeElement;

    afterNextRender({
      write: () => {
        const padding = computePadding();
        const changed = padding !== this.prevPadding;
        if (changed) {
          nativeElement.style.padding = padding;
        }
        return changed;
      },
      read: (didWrite) => {
        if (didWrite) {
          this.elementHeight = nativeElement.getBoundingClientRect().height;
        }
      },
    });
  }
}
```

## Lifecycle Interfaces

TypeScript interfaces exist for each lifecycle method, named without the `ng` prefix:

```typescript
@Component({/*...*/})
export class UserProfile implements OnInit {
  ngOnInit() {
    /* ... */
  }
}
```

Available interfaces: `OnInit`, `OnChanges`, `OnDestroy`, `DoCheck`, `AfterContentInit`, `AfterContentChecked`, `AfterViewInit`, `AfterViewChecked`.

## Execution Order

### During Initialization

1. `constructor`
2. `ngOnChanges`
3. `ngOnInit`
4. `ngDoCheck`
5. `ngAfterContentInit`
6. `ngAfterContentChecked`
7. `ngAfterViewInit`
8. `ngAfterViewChecked`
9. `afterNextRender` / `afterEveryRender`

### Subsequent Updates

1. `ngOnChanges`
2. `ngDoCheck`
3. `ngAfterContentChecked`
4. `ngAfterViewChecked`
5. `afterEveryRender`

## Important Notes

- Angular traverses the application tree top-to-bottom, visiting each component exactly once per cycle
- Avoid making state changes during lifecycle hook execution (except in specific hooks like `ngOnInit`)
- Many "After" hooks cannot safely modify state; doing so triggers `ExpressionChangedAfterItHasBeenCheckedError`
- Render callbacks don't execute during server-side rendering or build-time pre-rendering
- Framework doesn't guarantee ordering between component and directive lifecycle hooks on the same element
