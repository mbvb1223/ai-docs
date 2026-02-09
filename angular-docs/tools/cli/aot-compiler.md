# Ahead-of-Time (AOT) Compilation in Angular
> Source: https://angular.dev/tools/cli/aot-compiler

## Overview

Angular applications require compilation before running in browsers. The **Ahead-of-Time (AOT) compiler** converts Angular HTML and TypeScript into efficient JavaScript during the build phase, before browser download and execution.

## Key Benefits of AOT

| Benefit | Description |
|---------|-------------|
| **Faster Rendering** | Pre-compiled code loads immediately without client-side compilation |
| **Fewer Requests** | External templates and stylesheets inline into JavaScript, eliminating separate AJAX calls |
| **Smaller Payload** | No need to download Angular's compiler (~50% of framework size) |
| **Early Error Detection** | Template binding issues surface during build, not runtime |
| **Enhanced Security** | No client-side template evaluation reduces injection attack surface |

## Compilation Options

Angular supports two compilation approaches:

- **Just-in-Time (JIT)**: Runtime compilation in browser (Angular 8 default)
- **Ahead-of-Time (AOT)**: Build-time compilation (Angular 9+ default)

The `aot` property in `angular.json` controls which compiler the `ng build` and `ng serve` commands use.

## How AOT Works

### Metadata Extraction

The compiler extracts **metadata** from decorators like `@Component()` to understand how to construct and manage application instances. This metadata can be specified explicitly in decorators or implicitly in constructor declarations.

### Three Compilation Phases

**Phase 1: Code Analysis**
- TypeScript compiler and AOT collector create source representation
- Collector records metadata without interpretation
- Generates `.metadata.json` files describing decorator structures

**Phase 2: Code Generation**
- Compiler's `StaticReflector` interprets collected metadata
- Performs validation and throws errors for restriction violations
- Generates application code

**Phase 3: Template Type Checking** (Optional)
- TypeScript compiler validates template binding expressions
- Enable with `strictTemplates` compiler option
- Catches type errors before runtime

## Metadata Restrictions

Metadata must conform to a **restricted TypeScript subset**:

### Supported Syntax

| Syntax Type | Example |
|------------|---------|
| Literal object | `{cherry: true, apple: true}` |
| Literal array | `['cherries', 'flour', 'sugar']` |
| Spread in array | `['apples', 'flour', ...]` |
| Function calls | `bake(ingredients)` |
| Object instantiation | `new Oven()` |
| Property access | `pie.slice` |
| Array indexing | `ingredients[0]` |
| Template literals | `` `pie is ${multiplier} times better` `` |
| Boolean/number/string literals | `true`, `3.14`, `'pi'` |
| Null literal | `null` |
| Prefix operators | `!cake` |
| Binary operators | `a+b` |
| Ternary operator | `a ? b : c` |
| Parentheses | `(a+b)` |

### Unsupported Features

**Arrow Functions**: Metadata cannot use lambda expressions:

```typescript
// Invalid
@Component({
  providers: [{provide: server, useFactory: () => new Server()}]
})

// Valid
export function serverFactory() {
  return new Server();
}

@Component({
  providers: [{provide: server, useFactory: serverFactory}]
})
```

The compiler automatically rewrites arrow functions during `.js` emission in version 5+.

## Code Folding

The compiler evaluates expressions during collection and records results in `.metadata.json`, eliminating restrictions on non-exported symbols used in expressions.

### Foldable Expressions

| Syntax | Foldable |
|--------|----------|
| Literal object/array | Yes |
| Spread in array | No |
| Function/constructor calls | No |
| Property access | Yes (if target foldable) |
| Array indexing | Yes (if target and index foldable) |
| Local variable references | Yes |
| Template strings (no substitution) | Yes |
| Template strings (with foldable substitution) | Yes |
| Operators (if operands foldable) | Yes |

### Example

```typescript
const template = '<div>{{hero().name}}</div>';

@Component({
  selector: 'app-hero',
  template: template,
})
export class Hero {
  hero = input.required<Hero>();
}
```

The compiler folds `template` into the metadata, equivalent to:

```typescript
@Component({
  selector: 'app-hero',
  template: '<div>{{hero().name}}</div>',
})
```

## Visibility Requirements

- Decorated component members must be **public or protected** (never private)
- Data-bound properties must be public or protected
- `input()` properties cannot be private

## Supported Classes and Functions

**New Instances**: Only `InjectionToken` from `@angular/core`

**Decorators**: Only Angular decorators from `@angular/core`

**Functions**: Factory functions must be exported, named functions (no lambdas)

**Macros**: Support functions/static methods returning expressions

### Macro Example

```typescript
export function wrapInArray<T>(value: T): T[] {
  return [value];
}

@NgModule({
  declarations: wrapInArray(Typical),
})
export class TypicalModule {}
```

Treated as:

```typescript
@NgModule({
  declarations: [Typical],
})
```

## Metadata Rewriting

The compiler automatically rewrites object literals with `useClass`, `useValue`, `useFactory`, or `data` fields, converting expressions into exported variables:

```typescript
// Original
class TypicalServer {}
@NgModule({
  providers: [{provide: SERVER, useFactory: () => TypicalServer}],
})

// Rewritten as
class TypicalServer {}
export const θ0 = () => new TypicalServer();
@NgModule({
  providers: [{provide: SERVER, useFactory: θ0}],
})
```

This allows reference generation without knowing internal expression values.

## Template Type Checking

Enable via `strictTemplates` in TypeScript configuration:

```json
"angularCompilerOptions": {
  "strictTemplates": true
}
```

### Type Narrowing with ngIf

```typescript
@Component({
  selector: 'my-component',
  template: '<span *ngIf="person"> {{person.address.street}} </span>',
})
class MyComponent {
  person?: Person;
}
```

The `*ngIf` guard narrows the type, preventing "Object is possibly undefined" errors.

### Non-Null Assertion Operator

Use `!` to suppress errors when constraints guarantee non-null values:

```typescript
@Component({
  selector: 'my-component',
  template: '<span *ngIf="person"> {{person.name}} lives on {{address!.street}} </span>',
})
class MyComponent {
  person?: Person;
  address?: Address;
}
```

**Best Practice**: Include address checking in `*ngIf` instead when possible.

## Configuration

Set `strictMetadataEmit` to report syntax errors immediately rather than allowing errors in `.metadata.json`:

```json
"angularCompilerOptions": {
  "strictMetadataEmit": true
}
```

Angular libraries use this as standard practice.
