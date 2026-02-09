# Animation Transitions and Triggers
> Source: https://angular.dev/guide/animations/transition-and-triggers
> (Redirects to legacy animations: https://angular.dev/guide/legacy-animations/transition-and-triggers)

> **DEPRECATION NOTICE:** The `@angular/animations` package is now deprecated. The Angular team recommends using native CSS with `animate.enter` and `animate.leave` for animations for all new code.

## Overview

This guide covers special transition states like the `*` wildcard and `void` for Angular animations.

## Predefined States and Wildcard Matching

### Wildcard State (`*`)

An asterisk matches any animation state, enabling transitions regardless of the element's start or end state. For example, `open => *` applies when state changes from open to anything else.

Instead of defining each state pair separately, you can use wildcards:

```typescript
animations: [
  trigger('openClose', [
    state('open', style({
      height: '200px',
      opacity: 1,
      backgroundColor: 'yellow',
    })),
    state('closed', style({
      height: '100px',
      opacity: 0.8,
      backgroundColor: 'blue',
    })),
    transition('* => closed', [animate('1s')]),
    transition('* => open', [animate('0.5s')]),
  ]),
],
```

**Bidirectional transitions** use double-arrow syntax:

```typescript
transition('open <=> closed', [animate('0.5s')])
```

### Wildcard with Styles

Use `*` in style definitions to animate using the current computed value:

```typescript
transition('* => open', [animate('1s', style({opacity: '*'}))])
```

### Void State

The `void` state handles elements entering or leaving the page. Combine it with wildcards for complete enter/leave handling:

- `* => void`: Element leaving a view
- `void => *`: Element entering a view
- `* => *`: Matches any state change

## Animating Elements Entering and Leaving

### Example: Flying Elements

```typescript
animations: [
  trigger('flyInOut', [
    state('in', style({transform: 'translateX(0)'})),
    transition('void => *', [
      style({transform: 'translateX(-100%)'}),
      animate(100)
    ]),
    transition('* => void', [
      animate(100, style({transform: 'translateX(100%)'}))
    ]),
  ]),
],
```

### Aliases: `:enter` and `:leave`

These are shorthand for `void => *` and `* => void`:

```typescript
transition(':enter', [...])  // void => *
transition(':leave', [...])   // * => void
```

### Using with `*ngIf` and `*ngFor`

The `:enter` transition runs when views are inserted; `:leave` runs when removed.

**HTML template:**

```html
@if (isShown) {
  <div @myInsertRemoveTrigger class="insert-remove-container">
    <p>The box is inserted</p>
  </div>
}
```

**Component animation:**

```typescript
trigger('myInsertRemoveTrigger', [
  transition(':enter', [
    style({opacity: 0}),
    animate('100ms', style({opacity: 1}))
  ]),
  transition(':leave', [
    animate('100ms', style({opacity: 0}))
  ]),
])
```

## Transition Selectors: `:increment` and `:decrement`

These selectors trigger transitions when numeric values increase or decrease.

```typescript
trigger('filterAnimation', [
  transition(':enter, * => 0, * => -1', []),
  transition(':increment', [
    query(':enter', [
      style({opacity: 0, width: 0}),
      stagger(50, [animate('300ms ease-out', style({opacity: 1, width: '*'}))])
    ], {optional: true})
  ]),
  transition(':decrement', [
    query(':leave', [
      stagger(50, [animate('300ms ease-out', style({opacity: 0, width: 0}))])
    ])
  ])
])
```

## Boolean Values in Transitions

Trigger transitions using boolean bindings:

**HTML:**

```html
<div [@openClose]="isOpen ? true : false" class="open-close-container"></div>
```

**Component:**

```typescript
animations: [
  trigger('openClose', [
    state('true', style({height: '*'})),
    state('false', style({height: '0px'})),
    transition('false <=> true', animate(500)),
  ]),
],
```

## Multiple Animation Triggers

### Parent-Child Animations

Parent animations take priority. Child animations require the parent to query and use `animateChild()`.

### Disabling Animations

Use the `@.disabled` binding to turn off animations on an element and nested children:

**HTML:**

```html
<div [@.disabled]="isDisabled">
  <div [@childAnimation]="isOpen ? 'open' : 'closed'">
    <p>The box is now {{ isOpen ? 'Open' : 'Closed' }}!</p>
  </div>
</div>
```

**Component:**

```typescript
@Component({
  animations: [
    trigger('childAnimation', [...])
  ],
})
export class OpenCloseChild {
  isDisabled = false;
  isOpen = false;
}
```

**Global disable** via host binding:

```typescript
@Component({
  selector: 'app-root',
  templateUrl: 'app.html',
  animations: [slideInAnimation],
})
export class AppComponent {
  @HostBinding('@.disabled')
  public animationsDisabled = false;
}
```

## Animation Callbacks

Animations emit callbacks at start and completion via `$event`:

**HTML:**

```html
<div
  [@openClose]="isOpen ? 'open' : 'closed'"
  (@openClose.start)="onAnimationEvent($event)"
  (@openClose.done)="onAnimationEvent($event)"
  class="open-close-container">
</div>
```

**Component:**

```typescript
@Component({
  selector: 'app-open-close',
  animations: [trigger('openClose', [...])],
  templateUrl: 'open-close.html',
})
export class OpenClose {
  onAnimationEvent(event: AnimationEvent) {
    console.warn(`Animation Trigger: ${event.triggerName}`);
    console.warn(`Phase: ${event.phaseName}`);  // "start" or "done"
    console.warn(`Total time: ${event.totalTime}`);
    console.warn(`From: ${event.fromState}`);
    console.warn(`To: ${event.toState}`);
    console.warn(`Element: ${event.element}`);
  }
}
```

## Keyframes

Create multi-step animations within a single timing segment:

```typescript
transition('* => active', [
  animate('2s', keyframes([
    style({backgroundColor: 'blue'}),
    style({backgroundColor: 'red'}),
    style({backgroundColor: 'orange'}),
  ]))
])
```

### Offset

Define where each style change occurs (0 to 1 range):

```typescript
transition('* => active', [
  animate('2s', keyframes([
    style({backgroundColor: 'blue', offset: 0}),
    style({backgroundColor: 'red', offset: 0.8}),
    style({backgroundColor: '#754600', offset: 1.0}),
  ]))
]),
transition('* => inactive', [
  animate('2s', keyframes([
    style({backgroundColor: '#754600', offset: 0}),
    style({backgroundColor: 'red', offset: 0.2}),
    style({backgroundColor: 'blue', offset: 1.0}),
  ]))
])
```

Without explicit offsets, frames are evenly spaced.

### Pulsation Effect

```typescript
trigger('openClose', [
  state('open', style({
    height: '200px',
    opacity: 1,
    backgroundColor: 'yellow',
  })),
  state('close', style({
    height: '100px',
    opacity: 0.5,
    backgroundColor: 'green',
  })),
  transition('* => *', [
    animate('1s', keyframes([
      style({opacity: 0.1, offset: 0.1}),
      style({opacity: 0.6, offset: 0.2}),
      style({opacity: 1, offset: 0.5}),
      style({opacity: 0.2, offset: 0.7}),
    ]))
  ])
])
```

## Animatable Properties and Units

Angular supports any CSS-animatable property. For numeric values, include units as strings:

- Pixels: `'50px'`
- Relative font size: `'3em'`
- Percentage: `'100%'`
- Numbers default to pixels: `50` equals `'50px'`

### Automatic Property Calculation with Wildcards

Use `*` when a dimension is not known until runtime:

```typescript
animations: [
  trigger('shrinkOut', [
    state('in', style({height: '*'})),
    transition('* => void', [
      style({height: '*'}),
      animate(250, style({height: 0}))
    ])
  ])
]
```

## Related Resources

- [Introduction to Angular Animations](/guide/animations)
- [Complex Animation Sequences](/guide/animations/complex-sequences)
- [Reusable Animations](/guide/animations/reusable-animations)
- [Route Transition Animations](/guide/animations/route-animations)
