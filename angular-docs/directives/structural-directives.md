# Structural Directives
> Source: https://angular.dev/guide/directives/structural-directives

## Overview

Structural directives are directives applied to an `<ng-template>` element that conditionally or repeatedly render that template's content.

## Example Use Case: SelectDirective

This guide demonstrates building a `SelectDirective` that fetches data from a source and renders its template once data becomes available.

### Basic Usage

**Long-form syntax:**
```html
<ng-template select let-data [selectFrom]="source">
  <p>The data is: {{ data }}</p>
</ng-template>
```

**Important note:** Angular's `<ng-template>` element doesn't render by default without an applied structural directive.

## Structural Directive Shorthand

Angular provides shorthand syntax using the asterisk (`*`) prefix, eliminating the need to explicitly write `<ng-template>`:

```html
<p *select="let data; from: source">The data is: {{ data }}</p>
```

### Shorthand Transformation

The asterisk is transformed into an `<ng-template>` wrapper. The syntax consists of:

- **Template variable declaration:** `let data` binds to the context's `$implicit` property
- **Key-expression pairs:** `from: source` maps to a PascalCase property prefixed with the directive selector (`selectFrom`)

### Equivalent Forms

Both syntaxes produce identical results:

```html
<!-- Shorthand -->
<p class="data-view" *select="let data; from: source">
  The data is: {{ data }}
</p>

<!-- Long-form -->
<ng-template select let-data [selectFrom]="source">
  <p class="data-view">The data is: {{ data }}</p>
</ng-template>
```

## One Structural Directive Per Element

Only one structural directive can be applied per element using shorthand syntax, since there's only one `<ng-template>` element. Use `<ng-container>` to create wrapper layers when multiple structural directives are needed.

## Creating a Structural Directive

### Step 1: Generate the Directive

```bash
ng generate directive select
```

This creates the directive class with CSS selector `[select]`.

### Step 2: Make It Structural

Import and inject `TemplateRef` and `ViewContainerRef`:

```typescript
import {Directive, TemplateRef, ViewContainerRef} from '@angular/core';

@Directive({
  selector: '[select]',
})
export class SelectDirective {
  private templateRef = inject(TemplateRef);
  private viewContainerRef = inject(ViewContainerRef);
}
```

### Step 3: Add the selectFrom Input

```typescript
export class SelectDirective {
  // ...
  selectFrom = input.required<DataSource>();
}
```

### Step 4: Implement Business Logic

```typescript
export class SelectDirective {
  // ...
  async ngOnInit() {
    const data = await this.selectFrom.load();
    this.viewContainerRef.createEmbeddedView(this.templateRef, {
      $implicit: data,
    });
  }
}
```

## Structural Directive Syntax Reference

### Grammar Definition

```
prefix = "( :let | :expression ) (';' | ',')? ( :let | :as | :keyExp )";
as = :export "as" :local ";"?
keyExp = :key ":"? :expression ("as" :local)? ";"?
let = "let" :local "=" :export ";"?
```

### Translation Rules

| Shorthand | Translation |
|-----------|-------------|
| `prefix` with naked expression | `[prefix]="expression"` |
| `keyExp` | `[prefixKey]="expression"` |
| `let local` | `let-local="export"` |

### Examples

| Shorthand | Interpretation |
|-----------|---|
| `*myDir="let item of [1,2,3]"` | `<ng-template myDir let-item [myDirOf]="[1, 2, 3]">` |
| `*myDir="let item of [1,2,3] as items; trackBy: myTrack; index as i"` | `<ng-template myDir let-item [myDirOf]="[1,2,3]" let-items="myDirOf" [myDirTrackBy]="myTrack" let-i="index">` |
| `*ngComponentOutlet="componentClass"` | `<ng-template [ngComponentOutlet]="componentClass">` |

## Template Type Checking for Custom Directives

### Type Narrowing with ngTemplateGuard

Control how input expressions are narrowed using type assertion functions:

```typescript
@Directive(...)
class ActorIsUser {
  actor = input<User | Robot>();

  static ngTemplateGuard_actor(
    dir: ActorIsUser,
    expr: User | Robot
  ): expr is User {
    return true;
  }
}
```

For truthiness-based narrowing, use the literal type `'binding'`:

```typescript
@Directive(...)
class CustomIf {
  condition = input.required<boolean>();

  static ngTemplateGuard_condition: 'binding';
}
```

### Typing the Context with ngTemplateContextGuard

Define an interface for template context and implement a guard function to properly type it:

```typescript
export interface SelectTemplateContext<T> {
  $implicit: T;
}

@Directive(...)
export class SelectDirective<T> {
  selectFrom = input.required<DataSource<T>>();

  static ngTemplateContextGuard<T>(
    dir: SelectDirective<T>,
    ctx: any
  ): ctx is SelectTemplateContext<T> {
    return true;
  }
}
```

This enables the template type-checker to catch compile-time errors and provide proper type inference for generic directives.
