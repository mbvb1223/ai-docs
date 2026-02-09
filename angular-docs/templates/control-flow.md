# Control Flow
> Source: https://angular.dev/guide/templates/control-flow

## Overview

Angular templates support control flow blocks enabling conditional display and element repetition.

## Conditional Display with @if, @else if, and @else

### Basic @if Block

The `@if` block conditionally renders content when its condition expression evaluates to true:

```
@if (a > b) {
  <p>{{ a }} is greater than {{ b }}</p>
}
```

### Multiple Conditions

Chain multiple conditions using `@else if` and `@else`:

```
@if (a > b) {
  {{ a }} is greater than {{ b }}
} @else if (b > a) {
  {{ a }} is less than {{ b }}
} @else {
  {{ a }} is equal to {{ b }}
}
```

### Saving Conditional Results

Store conditional expression results in a template variable using the `as` keyword:

```
@if (user.profile.settings.startDate; as startDate) {
  {{ startDate }}
}
```

This approach improves readability when working with complex expressions.

## Repeating Content with @for

### Basic Syntax

The `@for` block iterates through collections and renders content repeatedly:

```
@for (item of items; track item.id) {
  {{ item.name }}
}
```

The `track` expression maintains relationships between data and DOM nodes, optimizing performance during updates.

**Note:** Angular's `@for` lacks flow-modifying statements like `continue` or `break` found in JavaScript.

### Track Expression Importance

The `track` expression is critical for performance optimization. Select a property uniquely identifying each item:

- **Preferred:** Use unique identifiers like `id` or `uuid`
- **Static collections:** Use `$index` for unchanging data
- **Last resort:** Use the item itself (`track item`), but this significantly impacts rendering performance

**Key difference:** Unlike `*ngFor`, the `@for` block prioritizes view reuse -- if tracked properties change but object references remain identical, Angular updates bindings rather than destroying and recreating elements.

### Contextual Variables

Several implicit variables are available within `@for` blocks:

| Variable | Meaning |
|----------|---------|
| `$count` | Total items in the collection |
| `$index` | Current row index |
| `$first` | Whether current row is first |
| `$last` | Whether current row is last |
| `$even` | Whether current row index is even |
| `$odd` | Whether current row index is odd |

### Aliasing Variables

Create aliases for contextual variables using `let`:

```
@for (item of items; track item.id; let idx = $index, e = $even) {
  <p>Item #{{ idx }}: {{ item.name }}</p>
}
```

Aliasing proves useful for nested `@for` blocks, allowing access to outer block variables.

### Empty State Handling

Optionally include an `@empty` block displaying content when collections contain no items:

```
@for (item of items; track item.name) {
  <li>{{ item.name }}</li>
} @empty {
  <li>There are no items.</li>
}
```

## Conditional Display with @switch

### Basic Syntax

The `@switch` block provides an alternative to `@if` for conditional rendering, resembling JavaScript's switch statement:

```
@switch (userPermissions) {
  @case ('admin') {
    <app-admin-dashboard />
  }
  @case ('reviewer')
  @case ('editor') {
    <app-editor-dashboard />
  }
  @default {
    <app-viewer-dashboard />
  }
}
```

### Key Characteristics

- Values are compared using the `===` operator
- **No fallthrough behavior** -- no `break` or `return` statements needed
- Multiple `@case` statements can target a single block
- Optional `@default` block renders when no cases match
- If no case matches and no default exists, nothing displays
