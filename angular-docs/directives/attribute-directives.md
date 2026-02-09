# Attribute Directives
> Source: https://angular.dev/guide/directives/attribute-directives

## Overview

Attribute directives modify the appearance or behavior of DOM elements and Angular components. They're applied as attributes to elements and can respond to user interactions, bind to properties, and manage styles dynamically.

## Building an Attribute Directive

### Step 1: Generate the Directive

Use Angular CLI to create a new directive:

```bash
ng generate directive highlight
```

This creates two files:
- `src/app/highlight.directive.ts` - the directive class
- `src/app/highlight.directive.spec.ts` - test file

### Step 2: Basic Directive Structure

```typescript
import {Directive} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {}
```

The `@Directive()` decorator's `selector` property specifies the CSS attribute selector `[appHighlight]`.

### Step 3: Access the Host Element

Import `ElementRef` and `inject` to access the DOM element:

```typescript
import {Directive, ElementRef, inject} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {
  private el = inject(ElementRef);

  constructor() {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

The `ElementRef` provides direct access to the host element through its `nativeElement` property.

## Applying the Directive

To use the directive, add it as an attribute to an HTML element:

```html
<p appHighlight>Highlight me!</p>
```

Angular instantiates the directive and injects a reference to the element, applying the yellow background.

**Note:** Directives do not support namespaces. Avoid syntax like `app:Highlight`.

## Handling User Events

### Configure Host Event Bindings

Use the `host` property in the decorator to bind to element events:

```typescript
@Directive({
  selector: '[appHighlight]',
  host: {
    '(mouseenter)': 'onMouseEnter()',
    '(mouseleave)': 'onMouseLeave()',
  },
})
export class HighlightDirective {
  private el = inject(ElementRef);

  onMouseEnter() {
    this.highlight('yellow');
  }

  onMouseLeave() {
    this.highlight('');
  }

  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

The background color appears on hover and disappears when the pointer leaves.

## Passing Values to Directives

### Using @Input Properties

Import the `input` function to make directive properties bindable:

```typescript
import {Directive, ElementRef, inject, input} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {
  private el = inject(ElementRef);
  appHighlight = input('');

  onMouseEnter() {
    this.highlight(this.appHighlight() || 'red');
  }
}
```

### Binding a Value

```typescript
// Component class
export class AppComponent {
  color = 'yellow';
}
```

```html
<p [appHighlight]="color">Highlight me!</p>
```

The `[appHighlight]` binding simultaneously applies the directive and sets the highlight color.

### Adding User Input

```html
<h1>My First Attribute Directive</h1>
<h2>Pick a highlight color</h2>
<div>
  <input type="radio" name="colors" (click)="color = 'lightgreen'" />Green
  <input type="radio" name="colors" (click)="color = 'yellow'" />Yellow
  <input type="radio" name="colors" (click)="color = 'cyan'" />Cyan
</div>
<p [appHighlight]="color">Highlight me!</p>
```

## Binding Multiple Properties

Add a second input property for a default color:

```typescript
@Directive({
  selector: '[appHighlight]',
  host: {
    '(mouseenter)': 'onMouseEnter()',
    '(mouseleave)': 'onMouseLeave()',
  },
})
export class HighlightDirective {
  private el = inject(ElementRef);
  appHighlight = input('');
  defaultColor = input('');

  onMouseEnter() {
    this.highlight(this.appHighlight() || this.defaultColor() || 'red');
  }

  onMouseLeave() {
    this.highlight('');
  }

  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

```html
<p [appHighlight]="color" defaultColor="violet">Highlight me too!</p>
```

When no color is selected, "violet" serves as the fallback; selected colors take precedence.

## Deactivating Angular Processing with NgNonBindable

The `ngNonBindable` directive prevents Angular from evaluating expressions and processing bindings in templates:

```html
<p ngNonBindable>This should not evaluate: {{ 1 + 1 }}</p>
```

### Using NgNonBindable with Directives

Directives still function when `ngNonBindable` is applied to the same element, but expression evaluation is disabled for child elements:

```html
<h3>ngNonBindable with a directive</h3>
<div ngNonBindable [appHighlight]="'yellow'">
  This should not evaluate: {{ 1 + 1 }}, but will highlight yellow.
</div>
```

This example highlights the element in yellow while preventing the expression `{{ 1 + 1 }}` from being evaluated.

## Key Takeaways

- Attribute directives modify element behavior and appearance
- Use `@Directive()` decorator with a selector for attribute binding
- Access the host element via injected `ElementRef`
- Configure event listeners through the `host` property
- Use `input()` for bindable properties
- `NgNonBindable` disables expression evaluation while allowing directives to function
