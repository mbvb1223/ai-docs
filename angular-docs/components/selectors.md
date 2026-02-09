# Component Selectors
> Source: https://angular.dev/guide/components/selectors

## Overview

Every Angular component requires a CSS selector that determines how it gets used. Selectors are matched statically at compile-time, meaning runtime DOM changes don't affect component rendering. An element can match exactly one component selector, and selectors are case-sensitive.

## Supported Selector Types

Angular supports a limited subset of basic CSS selectors:

| Selector Type | Description | Examples |
|---|---|---|
| Type selector | Matches elements by HTML tag name | `profile-photo` |
| Attribute selector | Matches elements by presence or exact value of an attribute | `[dropzone]` `[type="reset"]` |
| Class selector | Matches elements by CSS class presence | `.menu-item` |

**Important limitations:**
- Only the equals (`=`) operator is supported for attribute values
- No combinators (descendant or child combinators) are supported
- Namespaces are not supported

### The `:not` Pseudo-Class

Angular supports the `:not` pseudo-class to narrow selector matches:

```typescript
@Component({
  selector: '[dropzone]:not(textarea)',
  ...
})
export class DropZone { }
```

No other pseudo-classes or pseudo-elements are supported.

## Combining Selectors

### Concatenating Selectors

Multiple selectors can be combined by concatenation:

```typescript
@Component({
  selector: 'button[type="reset"]',
  ...
})
export class ResetButton { }
```

### Comma-Separated Lists

Define multiple selectors using comma separation:

```typescript
@Component({
  selector: 'drop-zone, [dropzone]',
  ...
})
export class DropZone { }
```

Angular creates a component for each element matching *any* selector in the list.

## Choosing a Selector

### Recommended Approach: Custom Element Names

Most components should use custom element names with hyphens per HTML specifications. Angular reports errors for unknown custom tags, preventing typo-related bugs.

### Selector Prefixes

The Angular team recommends using a short, consistent prefix for all custom components within your project. Example: YouTube-built-with-Angular might use `yt-menu`, `yt-player`, etc. The Angular CLI uses `app-` by default.

**Critical:** Never use the `ng` prefix -- that's reserved for Angular's own APIs.

### When to Use Attribute Selectors

Consider attribute selectors for creating components on standard HTML elements. For custom buttons:

```typescript
@Component({
  selector: 'button[yt-upload]',
  ...
})
export class YouTubeUploadButton { }
```

This approach leverages native element APIs and standard attributes like `aria-label` without additional work.

**Note:** Angular doesn't error on unknown custom attributes. Components with attribute selectors may fail silently if consumers forget imports, unlike type selectors that trigger compile-time errors.

Attribute selectors should use lowercase, dash-case attributes following the same prefixing conventions as type selectors.
