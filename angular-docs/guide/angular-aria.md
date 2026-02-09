# Angular Accessibility and ARIA
> Source: https://angular.dev/guide/angular-aria

## Overview

The web serves users with diverse abilities, including those with visual or motor impairments. Angular provides tools and best practices for building accessible applications. Google's web.dev offers foundational learning through its "Learn Accessibility" course.

## Accessibility Attributes

### ARIA Attributes and Properties

Angular supports binding to ARIA attributes using standard template syntax. Dynamic values bind like other HTML attributes:

```html
<button [aria-label]="myActionLabel">...</button>
```

Static ARIA attributes function as regular HTML:

```html
<button aria-label="Save document">...</button>
```

**Convention note:** HTML attributes use lowercase (`tabindex`), while DOM properties use camelCase (`tabIndex`).

For ARIA patterns requiring structured values like element collections, use property bindings to maintain synchronization:

```typescript
@Component({
  template: `
    <h2 #dialogTitle>Attention</h2>
    <p #dialogDescription>Please review your answers before continuing.</p>
    <section role="dialog" [ariaLabelledByElements]="[dialogTitle, dialogDescription]">
      <ng-content />
    </section>
  `,
})
export class ReviewDialog {}
```

The `[ariaLabelledByElements]` property accepts element arrays, keeping references current with template changes.

## Angular UI Components

**Angular Material** provides fully accessible, reusable UI components. The **Component Development Kit (CDK)** includes an `a11y` package offering accessibility tools:

- **LiveAnnouncer:** Announces messages to screen-reader users via `aria-live` regions (W3C documentation available)
- **cdkTrapFocus:** Constrains Tab-key focus within elements, useful for modal dialogs

### Augmenting Native Elements

Reuse native HTML elements instead of reimplementing behaviors. Prefer native `<button>` and `<a>` elements with attribute selectors over custom alternatives. Angular Material demonstrates this pattern: `MatButton`, `MatTabNav`, and `MatTable` all leverage native semantics.

### Using Containers for Native Elements

When native elements need wrapping (e.g., `<input>` elements cannot have children), create container components using content projection. This allows users to set arbitrary properties and attributes on the native control. `MatFormField` exemplifies this approach.

## Case Study: Custom Progress Bar

This example demonstrates accessibility through host binding:

```typescript
import {Component, input} from '@angular/core';

@Component({
  selector: 'app-example-progressbar',
  template: '<div class="bar" [style.width.%]="value()"></div>',
  styleUrls: ['./progress-bar.component.css'],
  host: {
    role: 'progressbar',
    'aria-valuemin': '0',
    'aria-valuemax': '100',
    '[attr.aria-valuenow]': 'value',
  },
})
export class ExampleProgressbarComponent {
  /** Current value of the progressbar. */
  value = input(0);
}
```

Usage with `aria-label`:

```html
<h1>Accessibility Example</h1>
<label for="progress-value">
  Enter an example progress value
  <input
    id="progress-value"
    type="number"
    min="0"
    max="100"
    [value]="progress"
    (input)="setProgress($event)"
  />
</label>

<app-example-progressbar [value]="progress" aria-label="Example of a progress bar" />
```

## Routing and Accessibility

### Focus Management After Navigation

When routing changes page content, focus management becomes critical. Use `NavigationEnd` events to update focus appropriately:

```typescript
router.events.pipe(filter((e) => e instanceof NavigationEnd)).subscribe(() => {
  const mainHeader = document.querySelector('#main-content-header');
  if (mainHeader) {
    mainHeader.focus();
  }
});
```

Focus should move to main content, not return to the body element after navigation.

### Active Links Identification

The `RouterLinkActive` directive provides `ariaCurrentWhenActive` input to set `aria-current` on active links. This supplements visual styling for users relying on assistive technologies:

```html
<nav>
  <a routerLink="home" routerLinkActive="active-page" ariaCurrentWhenActive="page">
    Home
  </a>
  <a routerLink="about" routerLinkActive="active-page" ariaCurrentWhenActive="page">
    About
  </a>
  <a routerLink="shop" routerLinkActive="active-page" ariaCurrentWhenActive="page">
    Shop
  </a>
</nav>
```

## Deferred Loading Accessibility

When using `@defer` blocks for lazy-loaded content, screen readers may not automatically announce changes. Wrap deferred content in ARIA live regions to ensure users are notified of new content. The defer guide provides detailed guidance and examples.

## Additional Resources

### Learning
- Google's web.dev "Learn Accessibility" course
- ARIA specification
- Material Design accessibility documentation
- W3C Web Accessibility Initiative

### Tools
- Angular ESLint provides linting rules supporting accessibility standards

### References
- Inclusive Components
- Deque University resources
- Rob Dodson A11ycasts

### Books
- "A Web for Everyone" (Horton & Quesenbery)
- "Inclusive Design Patterns" (Pickering)
