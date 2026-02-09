# Deferred Loading with @defer
> Source: https://angular.dev/guide/templates/defer

## Overview

The `@defer` block is an Angular feature that reduces initial bundle size by deferring the loading of non-essential code. This improves Core Web Vitals metrics, particularly Largest Contentful Paint (LCP) and Time to First Byte (TTFB).

Basic syntax:
```typescript
@defer {
  <large-component />
}
```

## Which Dependencies Get Deferred?

Components, directives, pipes, and component CSS can be deferred when they meet two conditions:

1. **Must be standalone** -- Non-standalone dependencies remain eagerly loaded even inside `@defer` blocks
2. **Cannot be referenced outside the block** -- If used elsewhere in the same file or in ViewChild queries, they load eagerly

Transitive dependencies (dependencies of deferred components) don't need to be standalone and can use NgModule declarations.

## Managing Deferred Loading Stages

### @defer Block

The primary block containing lazily-loaded content. Defaults to triggering when the browser reaches an idle state.

### @placeholder Block

Optional content shown before deferring is triggered. Accepts a `minimum` parameter to prevent flickering:

```typescript
@defer {
  <large-component />
} @placeholder (minimum 500ms) {
  <p>Placeholder content</p>
}
```

Dependencies in placeholder blocks load eagerly.

### @loading Block

Optional content shown while deferred dependencies load, replacing the placeholder. Accepts two parameters:

- `after` -- delay before showing loading template
- `minimum` -- minimum display duration

Example:
```typescript
@loading (after 100ms; minimum 1s) {
  <img alt="loading..." src="loading.gif" />
}
```

### @error Block

Optional block displaying if deferred loading fails:

```typescript
@error {
  <p>Failed to load large component.</p>
}
```

## Controlling Loading with Triggers

### `on` Triggers

| Trigger | Behavior |
|---------|----------|
| `idle` | Triggers when browser is idle (default) |
| `viewport` | Triggers when content enters viewport |
| `interaction` | Triggers on user click or keydown |
| `hover` | Triggers on mouseover or focusin |
| `immediate` | Triggers after non-deferred content renders |
| `timer(duration)` | Triggers after specified time |

#### Viewport Example

```typescript
@defer (on viewport) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

With template reference:
```typescript
<div #greeting>Hello!</div>
@defer (on viewport(greeting)) {
  <greetings-cmp />
}
```

#### Interaction Example

```typescript
@defer (on interaction) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

#### Timer Example

```typescript
@defer (on timer(500ms)) {
  <large-cmp />
}
```

### `when` Trigger

Loads deferred content based on a custom condition:

```typescript
@defer (when condition) {
  <large-cmp />
}
```

Note: This is a one-time operation; reverting to placeholder if condition becomes falsy is not supported.

## Prefetching

Prefetch triggers load JavaScript before content displays, improving perceived performance:

```typescript
@defer (on interaction; prefetch on idle) {
  <large-cmp />
} @placeholder {
  <div>Large component placeholder</div>
}
```

Prefetch and main triggers are separated with semicolon.

## Testing @defer Blocks

Angular provides TestBed APIs for testing. Manual control mode allows stepping through states:

```typescript
TestBed.configureTestingModule({deferBlockBehavior: DeferBlockBehavior.Manual});

const deferBlockFixture = (await componentFixture.getDeferBlocks())[0];
await deferBlockFixture.render(DeferBlockState.Loading);
await deferBlockFixture.render(DeferBlockState.Complete);
```

## NgModule Compatibility

`@defer` works with both standalone and NgModule-based components, but only standalone dependencies can actually be deferred. NgModule-based dependencies load eagerly.

## Hot Module Replacement (HMR)

When HMR is active, all `@defer` chunks load eagerly, overriding configured triggers. To restore trigger behavior, disable HMR with the `--no-hmr` flag.

## Server-Side Rendering (SSR) and Static Generation (SSG)

By default, `@defer` blocks render only their `@placeholder` on the server. Client-side triggers activate after hydration. Enable Incremental Hydration and configure `hydrate` triggers to render main content on the server.

## Best Practices

**Avoid cascading loads** -- Nested `@defer` blocks should use different triggers to prevent simultaneous loading.

**Prevent layout shifts** -- Don't defer components visible on initial page load, as this increases Cumulative Layout Shift (CLS). Avoid `immediate`, `timer`, `viewport`, and conditional `when` triggers for above-the-fold content.

**Support accessibility** -- Wrap `@defer` in live regions for screen reader announcements:

```typescript
<div aria-live="polite" aria-atomic="true">
  @defer (on timer(2000)) {
    <user-profile [user]="currentUser" />
  } @placeholder {
    Loading user profile...
  }
</div>
```
