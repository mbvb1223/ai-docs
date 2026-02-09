# Redirecting Routes
> Source: https://angular.dev/guide/routing/redirecting-routes

## Overview

Route redirects enable automatic navigation from one path to another. This functionality supports handling legacy URLs, creating default routes, and managing access control through configuration.

## Basic Configuration

The `redirectTo` property in route configurations accepts a string value:

```typescript
import {Routes} from '@angular/router';

const routes: Routes = [
  // Simple redirect
  {path: 'marketing', redirectTo: 'newsletter'},

  // Redirect with path parameters
  {path: 'legacy-user/:id', redirectTo: 'users/:id'},

  // Wildcard redirect for unmatched paths
  {path: '**', redirectTo: '/login'},
];
```

This configuration demonstrates three redirect scenarios:
- Direct path mapping (`/marketing` to `/newsletter`)
- Parameter preservation (`/legacy-user/:id` to `/users/:id`)
- Catch-all wildcard routes using `**`

## Understanding `pathMatch`

The `pathMatch` property controls URL matching behavior and accepts two values:

| Value | Behavior |
|-------|----------|
| `'full'` | Entire URL path must match exactly |
| `'prefix'` | Only the beginning of the URL needs to match |

Default behavior uses the `prefix` strategy.

### Prefix Matching Strategy

With `pathMatch: 'prefix'` (default), Angular Router matches all subsequent routes:

```typescript
export const routes: Routes = [
  { path: 'news', redirectTo: 'blog' },
  // Equivalent to:
  { path: 'news', redirectTo: 'blog', pathMatch: 'prefix' },
];
```

Examples of prefix matching behavior:
- `/news` redirects to `/blog`
- `/news/article` redirects to `/blog/article`
- `/news/article/:id` redirects to `/blog/article/:id`

### Full Matching Strategy

`pathMatch: 'full'` redirects only exact path matches:

```typescript
export const routes: Routes = [
  {path: '', redirectTo: '/dashboard', pathMatch: 'full'}
];
```

This configuration redirects only the root path (`''`) to `/dashboard`. Other paths like `/login` or `/about` are unaffected.

**Important Note:** When configuring redirects on the root page (`"/"` or `""`), always set `pathMatch: 'full'` to prevent redirecting all URLs.

### Full vs. Prefix Comparison

Using `pathMatch: 'full'` with the news example:

```typescript
export const routes: Routes = [
  {path: 'news', redirectTo: '/blog', pathMatch: 'full'}
];
```

Results in:
- Only `/news` redirects to `/blog`
- `/news/articles` and `/news/articles/1` do NOT redirect

## Conditional Redirects

The `redirectTo` property accepts functions for dynamic redirect logic. The function receives a partial `ActivatedRouteSnapshot` (excluding resolved titles and lazy-loaded components) and typically returns a string or `URLTree`, though observables and promises are also supported.

### Time-Based Redirect Example

```typescript
import {Routes} from '@angular/router';
import {Menu} from './menu';

export const routes: Routes = [
  {
    path: 'restaurant/:location/menu',
    redirectTo: (activatedRouteSnapshot) => {
      const location = activatedRouteSnapshot.params['location'];
      const currentHour = new Date().getHours();

      // Check for specific meal query parameter
      if (activatedRouteSnapshot.queryParams['meal']) {
        return `/restaurant/${location}/menu/${queryParams['meal']}`;
      }

      // Auto-redirect based on time of day
      if (currentHour >= 5 && currentHour < 11) {
        return `/restaurant/${location}/menu/breakfast`;
      } else if (currentHour >= 11 && currentHour < 17) {
        return `/restaurant/${location}/menu/lunch`;
      } else {
        return `/restaurant/${location}/menu/dinner`;
      }
    },
  },
  // Destination routes
  {path: 'restaurant/:location/menu/breakfast', component: Menu},
  {path: 'restaurant/:location/menu/lunch', component: Menu},
  {path: 'restaurant/:location/menu/dinner', component: Menu},
  // Default redirect
  {path: '', redirectTo: '/restaurant/downtown/menu', pathMatch: 'full'},
];
```

This example demonstrates conditional logic that adapts redirect behavior based on request parameters and application state.

## Additional Resources

- [RedirectFunction API Documentation](https://angular.dev/api/router/RedirectFunction)
- [Route Configuration API Reference](https://angular.dev/api/router/Route#redirectTo)
