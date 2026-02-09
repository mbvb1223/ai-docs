# Using DOM APIs
> Source: https://angular.dev/guide/components/dom-apis

## Overview

Angular handles most DOM creation, updates, and removals automatically. However, rare cases may require direct DOM interaction. Components can inject `ElementRef` to access the host element.

## Getting Element References

Components can inject `ElementRef` to obtain a reference to their host element:

```typescript
@Component({/*...*/})
export class ProfilePhoto {
  constructor() {
    const elementRef = inject(ElementRef);
    console.log(elementRef.nativeElement);
  }
}
```

The `nativeElement` property references the host [Element](https://developer.mozilla.org/docs/Web/API/Element) instance.

## Render Callbacks

Angular provides `afterEveryRender` and `afterNextRender` functions to register callbacks that execute after rendering completes:

```typescript
@Component({/*...*/})
export class ProfilePhoto {
  constructor() {
    const elementRef = inject(ElementRef);
    afterEveryRender(() => {
      // Focus the first input element in this component.
      elementRef.nativeElement.querySelector('input')?.focus();
    });
  }
}
```

**Key Constraints:**
- Must be called within an injection context (typically in constructor)
- Never execute during server-side rendering or build-time pre-rendering
- Never manipulate the DOM inside other Angular lifecycle hooks
- Performance impact possible from layout thrashing

## Using Renderer2

Components can inject `Renderer2` for DOM manipulations tied to Angular features:

```typescript
constructor(private renderer: Renderer2) {}
```

**Capabilities:**
- DOM elements participate in component style encapsulation
- `setProperty` method updates synthetic animation properties
- `listen` method adds event listeners for synthetic animation events
- No difference from native DOM APIs except for animation integration
- Does not support server-side rendering or pre-rendering contexts

## When to Use DOM APIs

Appropriate use cases include:

- Managing element focus
- Measuring element geometry (`getBoundingClientRect`)
- Reading element text content
- Setting up native observers (`MutationObserver`, `ResizeObserver`, `IntersectionObserver`)

## What to Avoid

**Never directly insert, remove, or modify DOM elements.**

**Never set `innerHTML` directly** - exposes application to cross-site scripting (XSS) attacks. Angular's template bindings include safeguards against XSS. See the Security guide for details.

---

**Best Practice:** Always prefer expressing your DOM's structure in component templates and updating that DOM with bindings rather than direct manipulation.
