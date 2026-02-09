# Angular Animations
> Source: https://angular.dev/guide/animations

## Overview

Angular provides `animate.enter` and `animate.leave` APIs to create smooth transitions for DOM elements. These compiler-supported features apply CSS classes or trigger callback functions at appropriate timing, enhancing user experience by making page transitions less abrupt and directing attention through workflows.

## `animate.enter`

### Purpose

Animates elements as they enter the DOM using CSS classes with transitions or keyframe animations.

### Basic Implementation

**Component (TypeScript):**

```typescript
import {Component, signal} from '@angular/core';

@Component({
  selector: 'app-enter',
  templateUrl: 'enter.html',
  styleUrls: ['enter.css'],
})
export class Enter {
  isShown = signal(false);
  toggle() {
    this.isShown.update((isShown) => !isShown);
  }
}
```

**Template (HTML):**

```html
<h2><code>animate.enter</code> Example</h2>
<button type="button" class="toggle-btn" (click)="toggle()">
  Toggle Element
</button>
@if (isShown()) {
  <div class="enter-container" animate.enter="enter-animation">
    <p>The box is entering.</p>
  </div>
}
```

**Styles (CSS):**

```css
:host {
  display: block;
  height: 200px;
}

.enter-container {
  border: 1px solid #dddddd;
  margin-top: 1em;
  padding: 20px;
  font-weight: bold;
  font-size: 20px;
}

.enter-animation {
  animation: slide-fade 1s;
}

@keyframes slide-fade {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

### Key Features

- Angular removes animation classes after completion
- Supports binding with dynamic expressions: `[animate.enter]="enterClass()"`
- Accepts single class strings or arrays: `animate.enter="class1 class2"` or `[animate.enter]="['class1', 'class2']"`
- When multiple animations run, Angular waits for the longest to complete

### CSS Transitions with `@starting-style`

When using transitions, the class represents the target state. Pair with `@starting-style` for proper animations:

```css
.element {
  transition: opacity 0.3s ease-in;
  @starting-style {
    opacity: 0;
  }
}
```

## `animate.leave`

### Purpose

Animates elements as they leave the DOM, then automatically removes them after animation completes.

### Basic Implementation

**Component (TypeScript):**

```typescript
import {Component, signal} from '@angular/core';

@Component({
  selector: 'app-leave',
  templateUrl: 'leave.html',
  styleUrls: ['leave.css'],
})
export class Leave {
  isShown = signal(false);
  toggle() {
    this.isShown.update((isShown) => !isShown);
  }
}
```

**Template (HTML):**

```html
<h2><code>animate.leave</code> Example</h2>
<button type="button" class="toggle-btn" (click)="toggle()">
  Toggle Element
</button>
@if (isShown()) {
  <div class="leave-container" animate.leave="leaving">
    <p>Goodbye</p>
  </div>
}
```

**Styles (CSS):**

```css
:host {
  display: block;
  height: 200px;
}

.leave-container {
  border: 1px solid #dddddd;
  margin-top: 1em;
  padding: 20px;
  font-weight: bold;
  font-size: 20px;
  opacity: 1;
  transition: opacity 200ms ease-in;
  @starting-style {
    opacity: 0;
  }
}

.leaving {
  opacity: 0;
  transform: translateY(20px);
  transition:
    opacity 500ms ease-out,
    transform 500ms ease-out;
}
```

### Key Features

- Automatically removes element from DOM after animation completes
- Waits for the longest animation when multiple properties animate
- Supports binding and dynamic expressions like `[animate.leave]="farewell()"`
- Accepts single or multiple class strings

## Critical: Element Removal Order

> **Important constraint:** `animate.leave` only works on the element being removed directly. If placed on a descendant of the removed element, it will not animate because the parent removal takes the entire subtree immediately.

**Example of what will not work:**

```html
@if (isShown()) {
  <!-- Parent removal happens first, child animate.leave never executes -->
  <div class="leave-parent">
    <div class="leave-container" animate.leave="leaving">
      <p>Goodbye</p>
    </div>
  </div>
}
```

## Event Bindings and Third-Party Libraries

Both `animate.enter` and `animate.leave` support event binding syntax for function calls, enabling integration with libraries like GSAP or anime.js.

**Component with Function Call:**

```typescript
import {AnimationCallbackEvent, Component, signal} from '@angular/core';

@Component({
  selector: 'app-leave-binding',
  templateUrl: 'leave-event.html',
  styleUrls: ['leave-event.css'],
})
export class LeaveEvent {
  isShown = signal(false);
  toggle() {
    this.isShown.update((isShown) => !isShown);
  }

  leavingFn(event: AnimationCallbackEvent) {
    // Example with GSAP:
    // gsap.to(event.target, {
    //   duration: 1,
    //   x: 100,
    //   onComplete: () => event.animationComplete()
    // });
    event.animationComplete();
  }
}
```

**Template:**

```html
<h2><code>animate.leave</code> Function Example</h2>
<button type="button" class="toggle-btn" (click)="toggle()">
  Toggle Element
</button>
@if (isShown()) {
  <div class="leave-container" (animate.leave)="leavingFn($event)">
    <p>Goodbye</p>
  </div>
}
```

### Event Object Details

The `$event` has type `AnimationCallbackEvent` containing:

- `target`: The animated DOM element
- `animationComplete()`: Function to notify Angular the animation finished

> **Critical requirement:** When using event binding with `animate.leave`, you must call `animationComplete()`. If not called, Angular automatically removes the element after a 4-second default delay (configurable via `MAX_ANIMATION_TIMEOUT` token).

```typescript
// Configuration example:
{ provide: MAX_ANIMATION_TIMEOUT, useValue: 6000 }
```

## Compatibility with Legacy Animations

Cannot mix legacy Angular animations with `animate.enter` and `animate.leave` in the same component -- this causes enter classes to persist or leaving nodes to remain in DOM.

However, both can coexist in the same application with one exception: content projection across components using different animation systems is unsupported. If projecting content from a legacy animation component into an `animate.enter`/`animate.leave` component (or vice versa), behavior matches same-component usage -- unsupported.

## Testing

TestBed disables animations by default in test environments.

**Enable animations for testing:**

```typescript
TestBed.configureTestingModule({animationsEnabled: true});
```

> **Note:** Some test environments do not emit animation events like `animationstart`, `animationend`, or transition equivalents.

## Summary Benefits

- Smooth, non-jarring transitions enhance user experience
- Motion helps users detect application responses to actions
- Good animations direct user attention through workflows
- Simple API integrated directly into Angular's compiler
- Flexible: works with CSS or custom callback functions
