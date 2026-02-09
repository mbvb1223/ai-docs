# Router Lifecycle and Events
> Source: https://angular.dev/guide/routing/lifecycle-and-events

## Overview

Angular Router provides lifecycle hooks and events through the `Router.events` observable, enabling developers to respond to navigation changes and execute custom logic throughout the routing process.

## Common Router Events

The router emits events in chronological order during navigation:

| Event | Purpose |
|-------|---------|
| **NavigationStart** | Fired when navigation begins; includes requested URL |
| **RoutesRecognized** | Fired after router matches URL to a route; contains route state |
| **GuardsCheckStart** | Begins evaluation of route guards like `canActivate` and `canDeactivate` |
| **GuardsCheckEnd** | Completes guard evaluation; contains allow/deny result |
| **ResolveStart** | Begins data resolution phase; resolvers start fetching |
| **ResolveEnd** | Data resolution completes; all required data available |
| **NavigationEnd** | Navigation succeeds; router updates URL |
| **NavigationSkipped** | Router skips navigation (e.g., same URL requested) |

### Error Events

| Event | Cause |
|-------|-------|
| **NavigationCancel** | Router cancels navigation, often due to guard returning false |
| **NavigationError** | Navigation fails; may result from invalid routes or resolver errors |

## Subscribing to Router Events

Subscribe to `router.events` and check event instances:

```typescript
import {Component, inject, signal, effect} from '@angular/core';
import {Event, Router, NavigationStart, NavigationEnd} from '@angular/router';

@Component({/*...*/})
export class RouterEvents {
  private readonly router = inject(Router);

  constructor() {
    this.router.events.pipe(takeUntilDestroyed()).subscribe((event: Event) => {
      if (event instanceof NavigationStart) {
        console.log('Navigation starting:', event.url);
      }
      if (event instanceof NavigationEnd) {
        console.log('Navigation completed:', event.url);
      }
    });
  }
}
```

**Note:** The `Event` type from `@angular/router` differs from the global browser `Event` type and the `RouterEvent` type.

## Debugging Routing Events

Enable detailed console logging using the `withDebugTracing()` configuration function:

```typescript
import {provideRouter, withDebugTracing} from '@angular/router';

const appRoutes: Routes = [];
bootstrapApplication(App, {
  providers: [provideRouter(appRoutes, withDebugTracing())],
});
```

This logs all router events to help understand navigation flow and identify issues.

## Common Use Cases

### Loading Indicators

Display loading state during navigation:

```typescript
import {Component, inject} from '@angular/core';
import {Router} from '@angular/router';

@Component({
  selector: 'app-root',
  template: `
    @if (isNavigating()) {
      <div class="loading-bar">Loading...</div>
    }
    <router-outlet />
  `,
})
export class App {
  private router = inject(Router);
  isNavigating = computed(() => !!this.router.currentNavigation());
}
```

### Analytics Tracking

Track page views when navigation completes:

```typescript
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';
import {inject, Injectable, DestroyRef} from '@angular/core';
import {Router, NavigationEnd} from '@angular/router';

@Injectable({providedIn: 'root'})
export class AnalyticsService {
  private router = inject(Router);
  private destroyRef = inject(DestroyRef);

  startTracking() {
    this.router.events.pipe(takeUntilDestroyed(this.destroyRef)).subscribe((event) => {
      if (event instanceof NavigationEnd) {
        this.analytics.trackPageView(event.url);
      }
    });
  }

  private analytics = {
    trackPageView: (url: string) => {
      console.log('Page view tracked:', url);
    },
  };
}
```

### Error Handling

Handle navigation errors and cancellations gracefully:

```typescript
import {Component, inject, signal} from '@angular/core';
import {
  Router,
  NavigationStart,
  NavigationError,
  NavigationCancel,
  NavigationCancellationCode,
} from '@angular/router';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-error-handler',
  template: `
    @if (errorMessage()) {
      <div class="error-banner">
        {{ errorMessage() }}
        <button (click)="dismissError()">Dismiss</button>
      </div>
    }
  `,
})
export class ErrorHandler {
  private router = inject(Router);
  readonly errorMessage = signal('');

  constructor() {
    this.router.events.pipe(takeUntilDestroyed()).subscribe((event) => {
      if (event instanceof NavigationStart) {
        this.errorMessage.set('');
      } else if (event instanceof NavigationError) {
        console.error('Navigation error:', event.error);
        this.errorMessage.set('Failed to load page. Please try again.');
      } else if (event instanceof NavigationCancel) {
        console.warn('Navigation cancelled:', event.reason);
        if (event.code === NavigationCancellationCode.GuardRejected) {
          this.errorMessage.set('Access denied. Check permissions.');
        }
      }
    });
  }

  dismissError() {
    this.errorMessage.set('');
  }
}
```

## Complete Event Categories

### Navigation Events
Tracks core navigation phases: start, route recognition, guard checks, data resolution.

### Activation Events
Fires during component instantiation for each route in the tree:
- **ActivationStart** / **ActivationEnd**: Route activation
- **ChildActivationStart** / **ChildActivationEnd**: Child route activation

### Navigation Completion Events
Final outcome of navigation (exactly one fires per navigation):
- **NavigationEnd**: Success
- **NavigationCancel**: Cancelled by guard
- **NavigationError**: Unexpected error
- **NavigationSkipped**: Same URL navigation

### Other Events
- **Scroll**: Occurs during scrolling

## Related Topics

- [Route Guards](/guide/routing/route-guards)
- [Common Router Tasks](/guide/routing/common-router-tasks)
