# Control Route Access with Guards
> Source: https://angular.dev/guide/routing/route-guards

## Overview

Route guards are functions that control navigation to or from particular routes. They act as checkpoints managing user access to specific routes, commonly used for authentication and access control.

**Critical Security Note:** Never rely on client-side guards as the sole source of access control. All JavaScript that runs in a web browser can be modified by the user running the browser. Always enforce user authorization server-side, in addition to any client-side guards.

## Creating a Route Guard

Generate a guard using Angular CLI:

```bash
ng generate guard CUSTOM_NAME
```

This prompts selection of guard type and creates a corresponding `CUSTOM_NAME-guard.ts` file.

**Tip:** Guards can be created manually as separate TypeScript files, typically using the `-guard.ts` filename suffix.

## Route Guard Return Types

All route guards support the same return types:

| Return Type | Behavior |
|---|---|
| `boolean` | `true` allows navigation; `false` blocks it (except `CanMatch` tries other routes) |
| `UrlTree` or `RedirectCommand` | Redirects to another route instead of blocking |
| `Promise<T>` or `Observable<T>` | Router uses first emitted value then unsubscribes |

**Note:** `CanMatch` behaves differently -- returning `false` triggers attempts to match other routes rather than complete navigation blocking.

## Types of Route Guards

### CanActivate

Determines whether a user can access a route; primarily for authentication and authorization.

**Default Arguments:**
- `route: ActivatedRouteSnapshot` - Information about the route being activated
- `state: RouterStateSnapshot` - Current router state

**Example:**

```typescript
export const authGuard: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const authService = inject(AuthService);
  return authService.isAuthenticated();
};
```

**Guidance:** When redirecting, return a `UrlTree` or `RedirectCommand` -- avoid returning `false` then programmatically navigating.

### CanActivateChild

Determines whether users can access child routes of a parent route; protects entire nested route sections. The guard executes once for all children regardless of depth.

**Default Arguments:**
- `childRoute: ActivatedRouteSnapshot` - Future snapshot of the child route being activated
- `state: RouterStateSnapshot` - Current router state

**Example:**

```typescript
export const adminChildGuard: CanActivateChildFn = (
  childRoute: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const authService = inject(AuthService);
  return authService.hasRole('admin');
};
```

### CanDeactivate

Determines whether users can leave a route; commonly prevents navigation away from unsaved forms.

**Default Arguments:**
- `component: T` - Component instance being deactivated
- `currentRoute: ActivatedRouteSnapshot` - Current route information
- `currentState: RouterStateSnapshot` - Current router state
- `nextState: RouterStateSnapshot` - Next router state being navigated to

**Example:**

```typescript
export const unsavedChangesGuard: CanDeactivateFn<Form> = (
  component: Form,
  currentRoute: ActivatedRouteSnapshot,
  currentState: RouterStateSnapshot,
  nextState: RouterStateSnapshot,
) => {
  return component.hasUnsavedChanges()
    ? confirm('You have unsaved changes. Are you sure you want to leave?')
    : true;
};
```

### CanMatch

Determines whether a route matches during path matching. Unlike other guards, rejection attempts matching other routes instead of complete blocking; useful for feature flags, A/B testing, or conditional route loading.

**Default Arguments:**
- `route: Route` - Route configuration being evaluated
- `segments: UrlSegment[]` - Unconsumed URL segments

**Basic Example:**

```typescript
export const featureToggleGuard: CanMatchFn = (route: Route, segments: UrlSegment[]) => {
  const featureService = inject(FeatureService);
  return featureService.isFeatureEnabled('newDashboard');
};
```

**Advanced Example -- Multiple Components for Same Path:**

```typescript
const routes: Routes = [
  {
    path: 'dashboard',
    component: AdminDashboard,
    canMatch: [adminGuard],
  },
  {
    path: 'dashboard',
    component: UserDashboard,
    canMatch: [userGuard],
  },
];
```

When visiting `/dashboard`, the first matching guard determines which component renders.

## Applying Guards to Routes

Guards are specified as arrays in route configuration, executing in order specified. Multiple guards can protect single routes:

```typescript
import {Routes} from '@angular/router';
import {authGuard} from './guards/auth.guard';
import {adminGuard} from './guards/admin.guard';
import {canDeactivateGuard} from './guards/can-deactivate.guard';
import {featureToggleGuard} from './guards/feature-toggle.guard';

const routes: Routes = [
  // Basic CanActivate - requires authentication
  {
    path: 'dashboard',
    component: Dashboard,
    canActivate: [authGuard],
  },
  // Multiple CanActivate guards - requires authentication AND admin role
  {
    path: 'admin',
    component: Admin,
    canActivate: [authGuard, adminGuard],
  },
  // CanActivate + CanDeactivate - protected route with unsaved changes check
  {
    path: 'profile',
    component: Profile,
    canActivate: [authGuard],
    canDeactivate: [canDeactivateGuard],
  },
  // CanActivateChild - protects all child routes
  {
    path: 'users',
    canActivateChild: [authGuard],
    children: [
      {path: 'list', component: UserList},
      {path: 'detail/:id', component: UserDetail},
    ],
  },
  // CanMatch - conditionally matches route based on feature flag
  {
    path: 'beta-feature',
    component: BetaFeature,
    canMatch: [featureToggleGuard],
  },
  // Fallback route if beta feature is disabled
  {
    path: 'beta-feature',
    component: ComingSoon,
  },
];
```
