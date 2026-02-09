# Create Template Fragments with ng-template
> Source: https://angular.dev/guide/templates/ng-template

## Overview

The `<ng-template>` element, inspired by the native HTML `<template>` element, enables developers to declare template fragments -- sections of content that render dynamically or programmatically rather than appearing immediately in the DOM.

## Creating Template Fragments

Template fragments are created by wrapping content in `<ng-template>` tags:

```html
<p>This is a normal element</p>
<ng-template>
  <p>This is a template fragment</p>
</ng-template>
```

When rendered, the `<ng-template>` content does not appear on the page until explicitly rendered through code or directives.

### Binding Context

Template fragments support dynamic expressions evaluated against the component where they're declared:

```typescript
@Component({
  template: `<ng-template>You've selected {{count}} items.</ng-template>`,
})
export class ItemCounter {
  count: number = 0;
}
```

Bindings remain valid regardless of where the fragment ultimately renders.

## Referencing Template Fragments

### Template Reference Variables

Add template reference variables to `<ng-template>` elements for local template access:

```html
<ng-template #myFragment>
  <p>This is a template fragment</p>
</ng-template>
```

### Query-Based References

Use component or directive queries to obtain `TemplateRef` objects:

```typescript
@Component({
  template: `<ng-template><p>Fragment</p></ng-template>`,
})
export class ComponentWithFragment {
  templateRef = viewChild<TemplateRef<unknown>>(TemplateRef);
}
```

For multiple fragments, assign template reference variables and query by name:

```typescript
@Component({
  template: `
    <ng-template #fragmentOne><p>First</p></ng-template>
    <ng-template #fragmentTwo><p>Second</p></ng-template>
  `,
})
export class ComponentWithFragment {
  fragmentOne = viewChild<TemplateRef<unknown>>('fragmentOne');
  fragmentTwo = viewChild<TemplateRef<unknown>>('fragmentTwo');
}
```

### Directive Injection

Directives applied directly to `<ng-template>` elements can inject the `TemplateRef`:

```typescript
@Directive({
  selector: '[myDirective]',
})
export class MyDirective {
  private fragment = inject(TemplateRef);
}
```

```html
<ng-template myDirective>
  <p>This is one template fragment</p>
</ng-template>
```

## Rendering Template Fragments

### NgTemplateOutlet

The `NgTemplateOutlet` directive renders fragments as siblings to the outlet element. Use it on `<ng-container>`:

```typescript
import {NgTemplateOutlet} from '@angular/common';
```

```html
<ng-template #myFragment>
  <p>This is a fragment</p>
</ng-template>
<ng-container *ngTemplateOutlet="myFragment"></ng-container>
```

This produces:
```html
<p>This is a fragment</p>
```

### ViewContainerRef

Inject `ViewContainerRef` for programmatic rendering using the `createEmbeddedView` method:

```typescript
@Component({
  selector: 'my-outlet',
  template: `<button (click)="showFragment()">Show</button>`,
})
export class MyOutlet {
  private viewContainer = inject(ViewContainerRef);
  fragment = input<TemplateRef<unknown> | undefined>();

  showFragment() {
    if (this.fragment()) {
      this.viewContainer.createEmbeddedView(this.fragment());
    }
  }
}
```

The fragment appends as a sibling to the component injecting the `ViewContainerRef`.

## Passing Parameters

Template fragments accept parameters using `let-` prefixed attributes:

```html
<ng-template let-pizzaTopping="topping">
  <p>You selected: {{ pizzaTopping }}</p>
</ng-template>
```

### With NgTemplateOutlet

Bind context via `ngTemplateOutletContext`:

```html
<ng-container
  [ngTemplateOutlet]="myFragment"
  [ngTemplateOutletContext]="{topping: 'onion'}" />
```

### With ViewContainerRef

Pass context as the second argument:

```typescript
this.viewContainer.createEmbeddedView(this.myFragment, {topping: 'onion'});
```

## Structural Directives

Structural directives inject `TemplateRef` and `ViewContainerRef` to render fragments conditionally or repeatedly. They support asterisk (`*`) syntax sugar:

```html
<section *myDirective>
  <p>This is a fragment</p>
</section>
```

Equivalent to:

```html
<ng-template myDirective>
  <section>
    <p>This is a fragment</p>
  </section>
</ng-template>
```

## Real-World Applications

Angular Material components demonstrate practical implementations:
- **Tabs**: Content remains unrendered until tabs activate
- **Tables**: Custom rendering templates for data display
