# Accessibility in Angular
> Source: https://angular.dev/best-practices/a11y

## Overview

The web serves diverse users, including those with visual or motor impairments. Angular provides best practices and tools to create applications that work well for everyone, including those relying on assistive technologies.

For comprehensive learning, Angular references Google's web.dev ["Learn Accessibility" course](https://web.dev/learn/accessibility/).

---

## Accessibility Attributes

Building accessible experiences involves setting ARIA (Accessible Rich Internet Applications) attributes to provide semantic meaning. Use Angular's attribute binding syntax to control accessibility-related attributes dynamically.

### ARIA Attributes and Properties

#### Dynamic ARIA Attributes

Bind ARIA attributes directly in templates like any HTML attribute:

```typescript
<button [aria-label]="myActionLabel">...</button>
```

#### Static ARIA Attributes

Static attributes work as regular HTML:

```typescript
<button aria-label="Save document">...</button>
```

**Note:** HTML attributes use lowercase (`tabindex`), while DOM properties use camelCase (`tabIndex`).

#### Structured ARIA Values

Some ARIA patterns expose APIs accepting structured values (like element collections). Use property binding to maintain synchronization:

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

The `[ariaLabelledByElements]` property binding keeps element references synchronized with template data changes.

See the binding guide for complete ARIA syntax summary.

---

## Angular UI Components

### Angular Material Library

[Angular Material](https://material.angular.dev) provides fully accessible, reusable UI components. The [Component Development Kit (CDK)](https://material.angular.dev/cdk/categories) includes the `a11y` package with accessibility tools:

- **LiveAnnouncer:** Announces screen-reader messages via `aria-live` regions
- **cdkTrapFocus:** Constrains Tab-key focus within elements (useful for modals)

Reference the [Angular CDK accessibility overview](https://material.angular.dev/cdk/a11y/overview) for complete tool documentation.

### Augmenting Native Elements

Reuse native HTML elements when possible rather than reimplementing standard behaviors. This applies primarily to `<button>` and `<a>`, but extends to other element types.

Angular Material examples:
- [`MatButton`](https://github.com/angular/components/blob/main/src/material/button/button.ts#L33C3-L36C5)
- [`MatTabNav`](https://github.com/angular/components/blob/main/src/material/tabs/tab-nav-bar/tab-nav-bar.ts#L62)
- [`MatTable`](https://github.com/angular/components/blob/main/src/material/table/table.ts#L40)

### Native Elements with Container Patterns

Some native elements (like `<input>`) cannot have children. When wrapping with extra elements, use content projection to expose the native control in the component API.

Example: [`MatFormField`](https://material.angular.dev/components/form-field/overview) demonstrates this pattern.

---

## Case Study: Custom Progress Bar

This example demonstrates building an accessible progress bar using host binding to control accessibility attributes.

### Component Implementation

```typescript
import { Component, input } from '@angular/core';

/**
 * Example progressbar component.
 */
@Component({
  selector: 'app-example-progressbar',
  template: '<div class="bar" [style.width.%]="value()"></div>',
  styleUrls: ['./progress-bar.component.css'],
  host: {
    // Sets the role for this component to "progressbar"
    role: 'progressbar',

    // Sets the minimum and maximum values for the progressbar role.
    'aria-valuemin': '0',
    'aria-valuemax': '100',

    // Binding that updates the current value of the progressbar.
    '[attr.aria-valuenow]': 'value',
  },
})
export class ExampleProgressbarComponent {
  /** Current value of the progressbar. */
  value = input(0);
}
```

### Usage Template

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

<!-- The user of the progressbar sets an aria-label to communicate what the progress means. -->
<app-example-progressbar [value]="progress" aria-label="Example of a progress bar" />
```

**Key points:**
- The component uses host binding to set ARIA attributes
- The `aria-label` allows users to describe the progress bar's purpose
- Dynamic binding keeps `aria-valuenow` synchronized with component state

---

## Routing Accessibility

### Focus Management After Navigation

Focus control is critical for accessibility. When routes change, manage focus intentionally to help users locate main content.

Use the `NavigationEnd` event from the `Router` service:

```typescript
router.events.pipe(filter((e) => e instanceof NavigationEnd)).subscribe(() => {
  const mainHeader = document.querySelector('#main-content-header');
  if (mainHeader) {
    mainHeader.focus();
  }
});
```

**Implementation considerations:**
- Identify the appropriate element for focus based on application structure
- Avoid returning focus to the `body` element after route changes
- Tailor focus targets to specific layouts

### Active Links Identification

CSS classes on active `RouterLink` elements (via `RouterLinkActive`) provide visual indicators. Add the `aria-current` attribute for assistive technologies.

`RouterLinkActive` provides the `ariaCurrentWhenActive` input:

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

This approach applies the `active-page` class while setting `aria-current="page"` on active links.

---

## Deferred Loading (@defer)

When using Angular's `@defer` blocks for lazy-loaded content, consider screen reader implications. Automatic announcements may not occur when deferred components load, leaving users unaware of new content.

**Solution:** Wrap `@defer` blocks in elements with ARIA live regions. Consult the defer guide accessibility section for detailed guidance and examples.

---

## Additional Resources

### Documentation and Guides
- [Accessibility - Google Web Fundamentals](https://developers.google.com/web/fundamentals/accessibility)
- [ARIA Specification and Authoring Practices](https://www.w3.org/TR/wai-aria)
- [Material Design - Accessibility](https://material.io/design/usability/accessibility.html)
- [Smashing Magazine](https://www.smashingmagazine.com/search/?q=accessibility)
- [Inclusive Components](https://inclusive-components.design)
- [Deque University Resources](https://dequeuniversity.com/resources)
- [W3C - Web Accessibility Initiative](https://www.w3.org/WAI/people-use-web)
- [Rob Dodson A11ycasts](https://www.youtube.com/watch?v=HtTyRajRuyY)

### Tools
- [Angular ESLint](https://github.com/angular-eslint/angular-eslint#functionality) provides linting rules for accessibility compliance

### Recommended Books
- "A Web for Everyone: Designing Accessible User Experiences" by Sarah Horton and Whitney Quesenbery
- "Inclusive Design Patterns" by Heydon Pickering
