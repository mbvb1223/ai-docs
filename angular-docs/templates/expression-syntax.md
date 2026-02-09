# Expression Syntax
> Source: https://angular.dev/guide/templates/expression-syntax

## Overview

Angular expressions follow JavaScript syntax but with important differences. This guide explains what's supported and what's not in Angular template expressions.

## Value Literals

Angular supports a subset of JavaScript literal values:

### Supported Literals

| Type | Examples |
|------|----------|
| String | `'Hello'`, `"World"` |
| Boolean | `true`, `false` |
| Number | `123`, `3.14` |
| Object | `{name: 'Alice'}` |
| Array | `['Onion', 'Cheese', 'Garlic']` |
| null | `null` |
| RegExp | `/\d+/` |
| Template string | `` `Hello ${name}` `` |
| Tagged template string | `` tag`Hello ${name}` `` |

### Unsupported Literals

| Type | Examples |
|------|----------|
| BigInt | `1n` |

## Globals

Angular expressions support only:
- `undefined`
- `$any`

Common JavaScript globals like `Number`, `Boolean`, `NaN`, `Infinity`, and `parseInt` are not available.

## Local Variables

Angular automatically provides special local variables in specific contexts, prefixed with `$`. For example, `@for` blocks provide variables like `$index`.

## Supported Operators

### Standard JavaScript Operators

| Operator | Examples |
|----------|----------|
| Arithmetic | `1 + 2`, `52 - 3`, `41 * 6`, `20 / 4`, `17 % 5`, `10 ** 3` |
| Grouping | `9 * (8 + 4)` |
| Conditional | `a > b ? true : false` |
| Logical | `&&`, `\|\|`, `!` |
| Nullish Coalescing | `possiblyNullValue ?? 'default'` |
| Comparison | `<`, `<=`, `>`, `>=`, `==`, `===`, `!==`, `!=` |
| Unary | `-x`, `+y` |
| Property Access | `person['name']` |
| typeof | `typeof 42` |
| void | `void 1` |
| in | `'model' in car` |
| Assignment | `a = b` |
| Compound Assignment | `+=`, `-=`, `*=`, `/=`, `%=`, `**=`, `&&=`, `\|\|=`, `??=` |
| Spread | `{...obj, foo: 'bar'}`, `[...arr, 1, 2, 3]`, `fn(...args)` |

### Non-Standard Angular Operators

| Operator | Examples |
|----------|----------|
| Pipe | `{{ total \| currency }}` |
| Optional Chaining | `someObj.someProp?.nestedProp` |
| Non-null Assertion | `someObj!.someProp` |

**Note:** Angular's optional chaining returns `null` (not `undefined`) when the left side is `null` or `undefined`.

## Unsupported Operators

- All bitwise operators (`&`, `&=`, `~`, `|=`, `^=`, etc.)
- Object destructuring (`const { name } = person`)
- Array destructuring (`const [firstItem] = items`)
- Comma operator (`x = (x++, x)`)
- instanceof (`car instanceof Automobile`)
- new (`new Car()`)

## Lexical Context

Expressions evaluate within the component class context plus any template variables, locals, and globals. The `this` keyword is implicit when referencing component members. However, template variables with the same name shadow component members. Explicit `this.` reference disambiguates, useful for signal narrowing with `@let` declarations.

## Declarations (Not Supported)

Angular expressions don't support declarations:
- Variables (`let label = 'abc'`, `const item = 'apple'`)
- Functions (`function myCustomFunction() { }`)
- Arrow functions (`() => { }`)
- Classes (`class Rectangle { }`)

## Event Listener Statements

Event handlers are statements, not expressions. They differ in two key ways:

1. **Statements support** assignment operators (excluding destructuring assignments)
2. **Statements do not support** pipes
