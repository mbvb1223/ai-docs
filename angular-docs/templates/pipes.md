# Pipes
> Source: https://angular.dev/guide/templates/pipes

## Overview

Pipes enable declarative data transformation within Angular templates. They use the vertical bar character (`|`) syntax, inspired by Unix pipes. This feature allows you to define a transformation function once and apply it across multiple templates.

**Important Note:** Angular's pipe syntax differs from standard JavaScript, which uses `|` for the bitwise OR operator. Angular template expressions do not support bitwise operations.

## Basic Example

```typescript
import {Component} from '@angular/core';
import {CurrencyPipe, DatePipe, TitleCasePipe} from '@angular/common';

@Component({
  selector: 'app-root',
  imports: [CurrencyPipe, DatePipe, TitleCasePipe],
  template: `
    <main>
      <h1>Purchases from {{ company | titlecase }} on {{ purchasedOn | date }}</h1>
      <p>Total: {{ amount | currency }}</p>
    </main>
  `,
})
export class ShoppingCart {
  amount = 123.45;
  company = 'acme corporation';
  purchasedOn = '2024-07-08';
}
```

When rendered for a US user, this produces:
```html
<main>
  <h1>Purchases from Acme Corporation on Jul 8, 2024</h1>
  <p>Total: $123.45</p>
</main>
```

Angular automatically applies locale-specific formatting. See the internationalization guide for localization details.

## Built-in Pipes

Angular provides these pipes in the `@angular/common` package:

| Pipe | Purpose |
|------|---------|
| `AsyncPipe` | Extracts values from Promises or RxJS Observables |
| `CurrencyPipe` | Formats numbers as currency strings using locale rules |
| `DatePipe` | Formats Date values according to locale conventions |
| `DecimalPipe` | Transforms numbers to strings with decimal points |
| `I18nPluralPipe` | Maps values to pluralized strings by locale rules |
| `I18nSelectPipe` | Maps keys to custom selector return values |
| `JsonPipe` | Converts objects to JSON string representation (debugging) |
| `KeyValuePipe` | Transforms Objects or Maps into key-value pair arrays |
| `LowerCasePipe` | Converts text to lowercase |
| `PercentPipe` | Formats numbers as percentage strings by locale |
| `SlicePipe` | Creates subset arrays or strings |
| `TitleCasePipe` | Converts text to title case |
| `UpperCasePipe` | Converts text to uppercase |

## Using Pipes

### Basic Syntax

```html
<p>Total: {{ amount | currency }}</p>
```

The left operand (value) passes to the transformation function; the right operand specifies the pipe name and optional parameters.

### Chaining Multiple Pipes

Apply multiple transformations by using successive pipe operators. Angular executes pipes left to right:

```html
<p>The event will occur on {{ scheduledOn | date | uppercase }}.</p>
```

### Passing Parameters

Some pipes accept configuration parameters. Use a colon (`:`) to separate the pipe name from its parameter value:

```html
<p>The event will occur at {{ scheduledOn | date: 'hh:mm' }}.</p>
```

For multiple parameters, separate each with a colon:

```html
<p>The event will occur at {{ scheduledOn | date: 'hh:mm' : 'UTC' }}.</p>
```

## Pipe Mechanics

### Operator Precedence

Pipes have lower precedence than binary operators (`+`, `-`, `*`, `/`, `%`, `&&`, `||`, `??`):

```html
<!-- Concatenation occurs before the uppercase pipe -->
{{ firstName + lastName | uppercase }}
```

Pipes have higher precedence than the ternary conditional operator:

```html
{{ (isAdmin ? 'Access granted' : 'Access denied') | uppercase }}
```

Without parentheses, this parses as:

```html
{{ isAdmin ? 'Access granted' : ('Access denied' | uppercase) }}
```

**Best Practice:** Use parentheses to eliminate ambiguity.

### Change Detection

By default, all pipes are "pure" -- they execute only when primitive input values (String, Number, Boolean, Symbol) or object references (Array, Object, Function, Date) change. Pure pipes optimize performance by avoiding unnecessary transformations when input hasn't changed.

However, mutations to object properties or array items go undetected unless the entire reference changes. For detecting internal changes, see the impure pipes section.

## Creating Custom Pipes

### Basic Structure

Create a custom pipe by decorating a TypeScript class with `@Pipe` and implementing `PipeTransform`:

```typescript
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'kebabCase',
})
export class KebabCasePipe implements PipeTransform {
  transform(value: string): string {
    return value.toLowerCase().replace(/ /g, '-');
  }
}
```

### Decorator Requirements

The `@Pipe` decorator requires a `name` property that determines template usage:

```typescript
@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe {}
```

### Naming Conventions

- **Pipe name:** Use camelCase; avoid hyphens
- **Class name:** Use PascalCase version of the name, appending "Pipe"

Example: pipe name `kebabCase` becomes class name `KebabCasePipe`

### Transform Method

The required `transform` method receives the input value and returns the transformed result:

```typescript
@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe implements PipeTransform {
  transform(value: string): string {
    return `My custom transformation of ${value}.`;
  }
}
```

### Adding Parameters

Extend the `transform` method with additional parameters to accept pipe arguments:

```typescript
@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe implements PipeTransform {
  transform(value: string, format: string): string {
    let msg = `My custom transformation of ${value}.`;
    if (format === 'uppercase') {
      return msg.toUpperCase();
    } else {
      return msg;
    }
  }
}
```

Usage:
```html
{{ myValue | myCustomTransformation: 'uppercase' }}
```

### Detecting Changes in Objects and Arrays

To detect mutations within objects or arrays, mark the pipe as impure by setting `pure: false`:

```typescript
@Pipe({
  name: 'joinNamesImpure',
  pure: false,
})
export class JoinNamesImpurePipe implements PipeTransform {
  transform(names: string[]): string {
    return names.join();
  }
}
```

**Warning:** Impure pipes can significantly impact performance. Include "Impure" in the name and class name to alert developers to the performance implications.
