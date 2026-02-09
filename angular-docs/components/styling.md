# Styling Components
> Source: https://angular.dev/guide/components/styling

## Overview

Angular components can optionally include CSS styles that apply to that component's DOM. Styles can be defined inline or in separate files, and Angular automatically includes associated styles when rendering components, even during lazy-loading.

## Inline Styles

Define styles directly in the component decorator using the `styles` property:

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
  styles: `
    img {
      border-radius: 50%;
    }
  `,
})
export class ProfilePhoto {}
```

## External Style Files

Separate styles into dedicated files using the `styleUrl` property:

```typescript
@Component({
  selector: 'profile-photo',
  templateUrl: 'profile-photo.html',
  styleUrl: 'profile-photo.css',
})
export class ProfilePhoto {}
```

## CSS Processing

Angular integrates with CSS preprocessors including Sass, Less, and Stylus. Component styles participate in the JavaScript module system and are emitted with component output.

---

## Style Scoping

Every component has a **view encapsulation** setting determining how the framework scopes component styles. Four encapsulation modes exist: `Emulated`, `ShadowDom`, `ExperimentalIsolatedShadowDom`, and `None`.

```typescript
@Component({
  ...,
  encapsulation: ViewEncapsulation.None,
})
export class ProfilePhoto { }
```

### ViewEncapsulation.Emulated (Default)

Angular generates unique HTML attributes for each component instance and adds those attributes to template elements. This ensures component styles don't leak to other components, though global styles can still affect encapsulated components.

**Key features:**
- Supports `:host` pseudo-class
- Supports `:host-context()` pseudo-class (deprecated in browsers but fully supported by Angular compiler)
- Does not support Shadow DOM-related pseudo-classes like `::shadow` or `::part`

#### ::ng-deep

The `::ng-deep` pseudo-class is a custom Angular feature that disables view-encapsulation boundaries after that point in selectors. The Angular team strongly discourages new use of `::ng-deep` and maintains it only for backwards compatibility.

**Behavior examples:**
- `p a` - matches `<a>` descendants of `<p>` within the component template
- `::ng-deep p a` - matches `<a>` elements anywhere in the application that are descendants of any `<p>`
- `p ::ng-deep a` - requires `<p>` from component template; `<a>` can be anywhere in the app
- `:host ::ng-deep p a` - both elements must be descendants of the component's host element

### ViewEncapsulation.ShadowDom

Uses the web standard Shadow DOM API to scope styles. Angular attaches a shadow root to the host element and renders the template and styles into the shadow tree.

**Important considerations:**
- Styles inside shadow trees cannot affect external elements
- Impacts event propagation and `<slot>` API interaction
- Affects how browser developer tools display elements

### ViewEncapsulation.ExperimentalIsolatedShadowDom

Similar to `ShadowDom` mode but with stricter guarantees: only the component's styles apply to its template elements. Global styles cannot penetrate the shadow tree, and internal styles cannot escape it.

### ViewEncapsulation.None

Disables all style encapsulation, making component styles behave as global styles.

**Note:** In `Emulated` and `ShadowDom` modes, Angular doesn't guarantee 100% style override priority. Styles with equal specificity may conflict.

---

## Styles in Templates

Use `<style>` elements directly in component templates to define additional styles. The component's view encapsulation mode applies to these styles.

**Limitation:** Angular does not support bindings inside style elements.

---

## External Style Files

Component templates can reference CSS files using the `<link>` element. CSS can also use the `@import` at-rule to reference external files.

**Key distinction:** External styles are not affected by emulated view encapsulation.
