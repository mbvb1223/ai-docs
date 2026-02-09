# Variables in Templates
> Source: https://angular.dev/guide/templates/variables

## Overview

Angular supports two types of variable declarations in templates: local template variables and template reference variables. These allow developers to store and reference values within template expressions.

## Local Template Variables with @let

### Purpose

The `@let` syntax enables declaration of local variables that can be reused across a template, similar to JavaScript's `let` keyword. Angular automatically maintains the variable's value in sync with its expression.

### Declaration Examples

Variables can be declared for various expression types:

- Simple property access: `@let name = user.name;`
- String concatenation: `@let greeting = 'Hello, ' + name;`
- Async pipe operations: `@let data = data$ | async;`
- Numeric literals: `@let pi = 3.14159;`
- Object literals: `@let coordinates = {x: 50, y: 100};`
- Multi-line expressions

**Key constraint:** Each `@let` block declares exactly one variable; multiple variables per block are not permitted.

### Usage Example

Once declared, variables are accessible throughout the template:

```html
@let user = user$ | async;

@if (user) {
  <h1>Hello, {{ user.name }}</h1>
  <user-avatar [photo]="user.photo" />
  <ul>
    @for (snack of user.favoriteSnacks; track snack.id) {
      <li>{{ snack.name }}</li>
    }
  </ul>
  <button (click)="update(user)">Update profile</button>
}
```

### Assignability Constraints

Unlike JavaScript's `let`, template variables cannot be reassigned after declaration. Angular automatically updates the variable value when the underlying expression changes -- reassignment through event handlers is invalid syntax.

### Variable Scope

`@let` declarations are scoped to the current view and its descendants. New views are created at component boundaries and wherever templates contain dynamic content (control flow blocks, `@defer` blocks, structural directives).

**Important limitation:** Declarations are not hoisted, so they cannot be accessed by parent views or sibling scopes. They're only accessible within the view they're declared and nested views.

### Formal Syntax Definition

The `@let` syntax requires:

1. The `@let` keyword
2. One or more whitespaces (excluding newlines)
3. A valid JavaScript identifier with optional trailing whitespace
4. An equals symbol with optional surrounding whitespace
5. An Angular expression (can span multiple lines)
6. Termination with a semicolon

## Template Reference Variables

### Purpose

Template reference variables provide a mechanism to declare variables that reference values from template elements, enabling cross-element communication within the same template.

### Eligible References

Variables can reference:

- DOM elements (including custom elements)
- Angular component or directive instances
- `TemplateRef` from `ng-template` elements

### Declaration Syntax

Variables are declared using a hash symbol (`#`) followed by the variable name:

```html
<!-- References HTMLInputElement -->
<input #taskInput placeholder="Enter task name" />
```

### Value Assignment Rules

Angular assigns values based on the element type:

**Angular Components:** The variable refers to the component instance.

```html
<!-- Variable holds MyDatepicker instance -->
<my-datepicker #startDate />
```

**ng-template Elements:** The variable references a `TemplateRef` instance representing the template.

```html
<!-- Variable holds TemplateRef for this fragment -->
<ng-template #myFragment>
  <p>This is a template fragment</p>
</ng-template>
```

**Other Elements:** The variable references the `HTMLElement` instance.

```html
<!-- Variable holds HTMLInputElement -->
<input #taskInput placeholder="Enter task name" />
```

### Directive Reference with exportAs

Angular directives can define an `exportAs` property for template reference access:

```typescript
@Directive({
  selector: '[dropZone]',
  exportAs: 'dropZone',
})
export class DropZone {
  /* ... */
}
```

To reference a directive instance, specify the `exportAs` name:

```html
<!-- Variable refers to DropZone directive instance -->
<section dropZone #firstZone="dropZone">...</section>
```

**Note:** Only directives with an `exportAs` property can be referenced this way.

### Integration with Component Queries

Template reference variables work with `@ViewChild` and related query decorators to access template elements in component code:

```html
<input #description value="Original description" />
```

```typescript
@Component({
  template: `<input #description value="Original description">`,
})
export class AppComponent {
  // Query element using template variable name
  @ViewChild('description') input: ElementRef | undefined;
}
```

This approach is detailed in the "Referencing children with queries" guide.
