# Binding Dynamic Text, Properties and Attributes
> Source: https://angular.dev/guide/templates/binding

## Overview

In Angular, **bindings** establish dynamic connections between a component's template and its underlying data, ensuring template updates whenever data changes.

## Text Interpolation

### Basic Syntax

Use double curly braces to render dynamic text:

```typescript
@Component({
  template: `
    <p>Your color preference is {{ theme }}.</p>
  `,
  ...
})
export class App {
  theme = 'dark';
}
```

Renders as: `<p>Your color preference is dark.</p>`

### Using Signals for Reactive Updates

For values that change over time, use signals. Angular tracks signal reads in templates and automatically updates when values change:

```typescript
@Component({
  template: `
    <!-- Does not necessarily update when welcomeMessage changes -->
    <p>{{ welcomeMessage }}</p>
    <!-- Always updates when theme signal changes -->
    <p>Your color preference is {{ theme() }}.</p>
  `,
  ...
})
export class App {
  welcomeMessage = "Welcome, enjoy this app that we built for you";
  theme = signal('dark');
}
```

### Key Points

- Text interpolation works anywhere normal HTML text appears
- All expression values convert to strings using `toString()` method
- Objects and arrays are converted through their `toString` implementation

## Property and Attribute Bindings

### Native Element Properties

Use square brackets to bind values to DOM element properties:

```html
<!-- Bind the disabled property on the button element -->
<button [disabled]="isFormValid()">Save</button>
```

Angular automatically updates the property whenever `isFormValid` changes.

### Component and Directive Properties

Apply the same syntax to set component input properties:

```html
<!-- Bind to MyListbox component's value property -->
<my-listbox [value]="mySelection()" />
```

For directives:

```html
<!-- Bind to NgOptimizedImage directive's ngSrc property -->
<img [ngSrc]="profilePhotoUrl()" alt="The current user's profile photo" />
```

### HTML Attributes

For attributes without corresponding DOM properties (such as SVG attributes), use the `attr.` prefix:

```html
<!-- Bind the role attribute on the ul element -->
<ul [attr.role]="listRole()">
```

When attribute binding values equal `null`, Angular removes the attribute using `removeAttribute()`.

### Text Interpolation in Properties and Attributes

Text interpolation syntax can substitute for square brackets:

```html
<!-- Binds to the alt property using interpolation -->
<img src="profile-photo.jpg" alt="Profile photo of {{ firstName() }}" />
```

## CSS Class and Style Bindings

### Conditional CSS Classes

Add or remove classes based on truthy/falsy values:

```html
<!-- When isExpanded is truthy, add the expanded class -->
<ul [class.expanded]="isExpanded()">
```

### Direct Class Binding

Bind to the `class` property with three supported formats:

**String format:**
```typescript
listClasses = 'full-width outlined';
```

**Array format:**
```typescript
sectionClasses = signal(['expandable', 'elevated']);
```

**Object format:**
```typescript
buttonClasses = signal({
  highlighted: true,
  embiggened: false,
});
```

Template usage:
```html
<ul [class]="listClasses"> ... </ul>
<section [class]="sectionClasses()"> ... </section>
<button [class]="buttonClasses()"> ... </button>
```

Renders as:
```html
<ul class="full-width outlined"> ... </ul>
<section class="expandable elevated"> ... </section>
<button class="highlighted"> ... </button>
```

### Combining Multiple Class Bindings

Angular intelligently combines static classes, direct bindings, and specific class bindings:

```html
<ul class="list" [class]="listType()" [class.expanded]="isExpanded()">
```

Results in: `<ul class="list box expanded">`

**Note:** Angular does not guarantee any specific order of CSS classes on rendered elements.

### CSS Style Properties

Bind directly to CSS style properties:

```html
<!-- Set display property based on isExpanded -->
<section [style.display]="isExpanded() ? 'block' : 'none'">
```

Specify units for properties that accept them:

```html
<!-- Set height with pixel units -->
<section [style.height.px]="sectionHeightInPixels()">
```

### Multiple Style Values

**String format:**
```typescript
listStyles = signal('display: flex; padding: 8px');
```

**Object format:**
```typescript
sectionStyles = signal({
  border: '1px solid black',
  'font-weight': 'bold',
});
```

Template usage:
```html
<ul [style]="listStyles()"> ... </ul>
<section [style]="sectionStyles()"> ... </section>
```

Renders as:
```html
<ul style="display: flex; padding: 8px"> ... </ul>
<section style="border: 1px solid black; font-weight: bold"> ... </section>
```

**Important:** When binding `style` to an object, Angular compares the previous value to the current value with the triple-equals operator (`===`). You must create a new object instance when you modify these values in order for Angular to apply any updates.

## ARIA Attributes

Bind string values to ARIA attributes:

```html
<button type="button" [aria-label]="actionLabel()">
  {{ actionLabel() }}
</button>
```

Angular writes values to the `aria-label` attribute and removes it when the value is `null`.

For ARIA features with structured values or element references, use standard property bindings as described in the accessibility guide.
