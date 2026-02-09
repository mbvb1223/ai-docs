# Content Projection with ng-content
> Source: https://angular.dev/guide/components/content-projection

## Overview

Angular provides the `<ng-content>` element as a placeholder mechanism for marking where child component content should be rendered within a parent component's template.

## Basic Concept

### What is Content Projection?

You can use the `<ng-content>` element as a placeholder to mark where content should go within component templates. This allows components to act as flexible containers for varied content types.

**Example - Basic Card Component:**

```typescript
@Component({
  selector: 'custom-card',
  template: `
    <div class="card-shadow">
      <ng-content />
    </div>
  `,
})
export class CustomCard {
  /* ... */
}
```

### How It Works in Practice

When a component uses `<ng-content>`, child elements passed to that component are rendered at the placeholder location:

```typescript
// Component definition
@Component({
  selector: 'custom-card',
  template: `
    <div class="card-shadow">
      <ng-content />
    </div>
  `,
})
export class CustomCard { }

// Usage
<custom-card>
  <p>This is the projected content</p>
</custom-card>

// Rendered output
<custom-card>
  <div class="card-shadow">
    <p>This is the projected content</p>
  </div>
</custom-card>
```

## Key Distinctions

- **Content**: Child elements passed to a component
- **View**: Elements defined within the component's template
- `<ng-content>` is neither a component nor DOM element but rather a special placeholder that tells Angular where to render content

## Important Constraints

`<ng-content>` elements are processed at build-time by Angular's compiler. Runtime manipulation is not possible:

- Cannot insert, remove, or modify `<ng-content>` dynamically
- Cannot add directives, styles, or attributes to `<ng-content>`
- Should not conditionally include with `@if`, `@for`, or `@switch`

## Multiple Content Placeholders

### Using the `select` Attribute

Multiple placeholders can target different content using CSS selectors:

```typescript
@Component({
  selector: 'card-title',
  template: `<ng-content>card-title</ng-content>`,
})
export class CardTitle {}

@Component({
  selector: 'card-body',
  template: `<ng-content>card-body</ng-content>`,
})
export class CardBody {}

@Component({
  selector: 'custom-card',
  template: `
    <div class="card-shadow">
      <ng-content select="card-title"></ng-content>
      <div class="card-divider"></div>
      <ng-content select="card-body"></ng-content>
    </div>
  `,
})
export class CustomCard {}
```

### Usage Pattern

```typescript
@Component({
  selector: 'app-root',
  imports: [CustomCard, CardTitle, CardBody],
  template: `
    <custom-card>
      <card-title>Hello</card-title>
      <card-body>Welcome to the example</card-body>
    </custom-card>`,
})
export class App {}
```

### Default Fallback Placeholder

When multiple `select` placeholders exist alongside one without `select`, the latter captures unmatched elements:

```typescript
<div class="card-shadow">
  <ng-content select="card-title"></ng-content>
  <div class="card-divider"></div>
  <!-- capture anything except "card-title" -->
  <ng-content></ng-content>
</div>
```

## Fallback Content

Default content can be specified within `<ng-content>` tags when no matching child exists:

```typescript
<div class="card-shadow">
  <ng-content select="card-title">Default Title</ng-content>
  <div class="card-divider"></div>
  <ng-content select="card-body">Default Body</ng-content>
</div>
```

When using the component with only a title, the body displays its default:

```typescript
<custom-card>
  <card-title>Hello</card-title>
  <!-- No card-body provided -->
</custom-card>
```

## Aliasing with `ngProjectAs`

The `ngProjectAs` attribute allows elements to match selectors different from their actual tag:

```typescript
<!-- Component template -->
<div class="card-shadow">
  <ng-content select="card-title"></ng-content>
  <div class="card-divider"></div>
  <ng-content />
</div>

<!-- Usage with aliasing -->
<custom-card>
  <h3 ngProjectAs="card-title">Hello</h3>
  <p>Welcome to the example</p>
</custom-card>
```

**Note**: `ngProjectAs` supports only static values and cannot be bound to dynamic expressions.

## Related Concepts

For conditional rendering of component content instead of unconditional projection, Angular recommends exploring Template fragments (`ng-template`).
