# Incremental Hydration
> Source: https://angular.dev/guide/incremental-hydration

## Overview

**Incremental hydration** is an advanced hydration technique that allows sections of an application to remain dehydrated initially, then trigger hydration on-demand as needed.

## Why Use Incremental Hydration?

Incremental hydration provides performance improvements by:

- **Reducing bundle size**: Smaller initial bundles improve load times, lowering First Input Delay (FID) and Cumulative Layout Shift (CLS)
- **Enabling above-the-fold deferrable content**: Previously, placing `@defer` blocks above the fold caused layout shifts. Incremental hydration renders the main template without shifting layout during hydration
- **Maintaining comparable UX**: Delivers user experience comparable to full-application hydration despite smaller bundles

## Enabling Incremental Hydration

Prerequisites: Server-side rendering (SSR) with hydration must already be configured.

Enable by adding `withIncrementalHydration()` to the `provideClientHydration` provider:

```typescript
import {
  bootstrapApplication,
  provideClientHydration,
  withIncrementalHydration,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withIncrementalHydration())]
});
```

> **Note**: Incremental hydration automatically enables event replay, so `withEventReplay()` can be removed if already present.

## How Incremental Hydration Works

The feature builds on full-application hydration, deferrable views (`@defer`), and event replay. Key mechanics:

- **During SSR**: The `hydrate` trigger loads defer block dependencies and renders the main template instead of `@placeholder`
- **During CSR**: Dependencies remain deferred; content stays dehydrated until the `hydrate` trigger activates
- **Event handling**: Browser events matching registered listeners are queued and replayed after hydration completes

## Hydrate Triggers

Control when Angular loads and hydrates deferred content using these triggers:

### `hydrate on` Triggers

| Trigger | Behavior |
|---------|----------|
| `hydrate on idle` | Triggers when browser reaches idle state via `requestIdleCallback` |
| `hydrate on viewport` | Triggers when content enters viewport (Intersection Observer API) |
| `hydrate on interaction` | Triggers on user `click` or `keydown` events |
| `hydrate on hover` | Triggers on `mouseover` or `focusin` events |
| `hydrate on immediate` | Triggers immediately after non-deferred content renders |
| `hydrate on timer(duration)` | Triggers after specified duration (e.g., `timer(500ms)`) |

### Example Usage

```typescript
@defer (hydrate on idle) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

### `hydrate when` Trigger

Accepts custom conditional expressions:

```typescript
@defer (hydrate when condition) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

> **Constraint**: Only triggers when the `@defer` block is the topmost dehydrated block.

### `hydrate never` Trigger

Keeps content permanently dehydrated on initial render (static):

```typescript
@defer (on viewport; hydrate never) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

> **Constraint**: Prevents hydration of entire nested subtree; no other `hydrate` triggers apply to nested content.

## Multiple Triggers

Multiple hydrate triggers (separated by semicolons) can be combined. Angular triggers hydration when *any* condition fires:

```typescript
@defer (on idle; hydrate on interaction) {
  <example-cmp />
} @placeholder {
  <div>Example Placeholder</div>
}
```

During initial load, `hydrate on interaction` applies. On subsequent client-side renders, `on idle` applies.

## Nested `@defer` Blocks

Angular's component system requires parent components to be hydrated before children. When a child's `hydrate` trigger fires, hydration cascades from the topmost dehydrated block downward:

```typescript
@defer (hydrate on interaction) {
  <parent-block-cmp />
  @defer (hydrate on hover) {
    <child-block-cmp />
  } @placeholder {
    <div>Child placeholder</div>
  }
} @placeholder {
  <div>Parent Placeholder</div>
}
```

Hovering triggers the parent block to hydrate first, then the child.

## Constraints

Incremental hydration shares constraints with full-application hydration:
- Limits on direct DOM manipulation
- Requires valid HTML structure
- See the [hydration guide constraints](./hydration.md) for details

## `@placeholder` Blocks

Yes, `@placeholder` blocks remain necessary. They are not used during incremental hydration but are required for subsequent client-side rendering scenarios when content was not part of the initial server render.
