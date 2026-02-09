# Hydration
> Source: https://angular.dev/guide/hydration

## Overview

Hydration is the mechanism that restores server-side rendered Angular applications on the client. It encompasses DOM structure reuse, application state persistence, and data transfer from server-executed operations.

## Why Hydration Matters

Hydration significantly improves performance by avoiding redundant DOM recreation. Rather than rebuilding components, Angular matches and reuses existing server-rendered DOM elements. This approach yields measurable improvements in Core Web Vitals metrics including:

- First Input Delay (FID)
- Largest Contentful Paint (LCP)
- Cumulative Layout Shift (CLS)

Without hydration, SSR applications destroy and re-render the DOM, causing visible flicker and negatively impacting user experience and SEO performance.

## Enabling Hydration

### Angular CLI Approach

When using Angular CLI for SSR setup (via `ng new` or `ng add @angular/ssr`), hydration configuration is automatically included.

### Manual Configuration

For custom setups, import and add the provider manually:

```typescript
import { bootstrapApplication, provideClientHydration } from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration()]
});
```

For NgModule-based applications:

```typescript
import { provideClientHydration } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

@NgModule({
  declarations: [App],
  exports: [App],
  bootstrap: [App],
  providers: [provideClientHydration()],
})
export class AppModule {}
```

> **Critical**: The `provideClientHydration()` call must be included in both client and server bootstrap configurations.

## Verification

Load your application in a browser and check the developer console. You should see hydration statistics including component and node counts. The Angular DevTools browser extension provides visual hydration status indicators.

## Event Capture and Replay

Starting in v18, enable event replay to capture user interactions occurring before hydration completes:

```typescript
import { provideClientHydration, withEventReplay } from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withEventReplay())],
});
```

### Event Replay Process

1. **Phase 1 - Capturing**: User interactions (clicks, mouseover, focusin) are captured before hydration.
2. **Phase 2 - Storage**: The Event Contract maintains captured events in memory.
3. **Phase 3 - Replay**: Once hydration completes, captured events execute to ensure no user actions are lost.

## Critical Constraints

### DOM Structure Consistency

Server and client DOM structures must match identically, including whitespaces and comment nodes. Any discrepancy prevents successful hydration.

### Direct DOM Manipulation

Components using native DOM APIs (`document` queries, `appendChild`, `innerHTML`, `outerHTML`) cause hydration failures. These operations alter DOM unexpectedly, creating mismatches between server and client structures.

**Solution**: Refactor using Angular APIs or apply `ngSkipHydration` as a temporary workaround.

### Valid HTML Structure

Invalid HTML patterns cause DOM mismatch errors:

- `<table>` without explicit `<tbody>`
- `<div>` nested within `<p>`
- `<a>` inside another `<a>`

Always use valid HTML structure. Modern browsers auto-insert `<tbody>`, creating inconsistencies -- explicitly declare it.

### Whitespace Configuration

Use the default `preserveWhitespaces: false` setting. If enabled, ensure consistency between `tsconfig.server.json` and `tsconfig.app.json`. Mismatches break hydration.

### Zone.js Requirements

Custom or "noop" Zone.js implementations may cause timing issues with the stability signal. Adjust `onStable` event timing if needed.

## Error Handling

Hydration errors typically result from:

1. DOM structure mismatches (most common)
2. Invalid HTML structure
3. Direct DOM manipulation
4. Inconsistent whitespace configuration

Reference the Errors Guide for complete hydration error documentation.

## Skipping Hydration for Components

When components cannot be refactored for hydration compatibility, use `ngSkipHydration`:

```html
<app-example ngSkipHydration />
```

Or via host binding:

```typescript
@Component({
  host: { ngSkipHydration: 'true' },
})
class ExampleComponent {}
```

This attribute forces Angular to skip hydration for the component and its children, treating them as non-hydrated. The component loses hydration benefits but renders normally.

> **Important**: `ngSkipHydration` applies only to component host nodes. Using it on the root component effectively disables hydration application-wide. Use sparingly as a last resort.

## Application Stability

Hydration depends on application stability. Delayed stability caused by timeouts, unresolved promises, or pending microtasks prevents hydration completion. The "Application remains unstable" error indicates the app hasn't stabilized within 10 seconds.

### Debugging Stability Issues

Import `provideStabilityDebugging` to identify stability blockers:

```typescript
import { provideStabilityDebugging } from '@angular/core';
import { bootstrapApplication } from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideStabilityDebugging()],
});
```

This logs pending tasks to the console. With `zone.js/plugins/task-tracking`, view macrotask stack traces:

```typescript
import 'zone.js/plugins/task-tracking';
```

> **Note**: These debugging utilities remain in production builds -- use only during development.

## Internationalization (i18n)

By default, Angular skips hydration for i18n block components. Enable i18n support:

```typescript
import {
  bootstrapApplication,
  provideClientHydration,
  withI18nSupport,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withI18nSupport())],
});
```

## Rendering Consistency

Avoid conditional rendering differences between server and client (using `@if` with `isPlatformBrowser`). Such inconsistencies cause layout shifts and degrade Core Web Vitals.

## Third-Party Library Considerations

### DOM-Manipulating Libraries

Libraries like D3 charts that perform DOM manipulation may cause hydration mismatches. Apply `ngSkipHydration` to affected components temporarily while seeking hydration-compatible alternatives.

### External Scripts

Ad trackers and analytics scripts modifying the DOM before hydration can trigger errors. Defer such scripts until post-hydration using `AfterNextRender`:

```typescript
import { AfterNextRender } from '@angular/core';
```

## Incremental Hydration

Advanced hydration allowing granular control over hydration timing. See the [incremental hydration guide](./incremental-hydration.md) for details.
