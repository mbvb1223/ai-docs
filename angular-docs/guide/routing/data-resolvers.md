# Data Resolvers
> Source: https://angular.dev/guide/routing/data-resolvers

## Overview

Data resolvers are functions implementing the `ResolveFn` interface that execute before route activation. They fetch essential data from APIs or databases, making it available to components immediately upon loading without requiring intermediate loading states.

## Key Benefits

- **Prevent empty states**: Components receive data immediately upon loading
- **Improved UX**: Eliminate loading spinners for critical data
- **Error handling**: Process data fetching failures before navigation occurs
- **Data consistency**: Guarantee required information exists before rendering, which proves important for server-side rendering scenarios

## Creating a Resolver

Resolvers are functions typed with `ResolveFn` that receive `ActivatedRouteSnapshot` and `RouterStateSnapshot` parameters:

```typescript
import {inject} from '@angular/core';
import {UserStore, SettingsStore} from './user-store';
import type {ActivatedRouteSnapshot, ResolveFn, RouterStateSnapshot} from '@angular/router';
import type {User, Settings} from './types';

export const userResolver: ResolveFn<User> = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const userStore = inject(UserStore);
  const userId = route.paramMap.get('id')!;
  return userStore.getUser(userId);
};

export const settingsResolver: ResolveFn<Settings> = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  const settingsStore = inject(SettingsStore);
  const userId = route.paramMap.get('id')!;
  return settingsStore.getUserSettings(userId);
};
```

## Route Configuration

Add resolvers to routes using the `resolve` property:

```typescript
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'user/:id',
    component: UserDetail,
    resolve: {
      user: userResolver,
      settings: settingsResolver,
    },
  },
];
```

## Accessing Resolved Data

### Method 1: ActivatedRoute with Signals

```typescript
import {Component, inject, computed} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';
import type {User, Settings} from './types';

@Component({
  template: `
    <h1>{{ user().name }}</h1>
    <p>{{ user().email }}</p>
    <div>Theme: {{ settings().theme }}</div>
  `,
})
export class UserDetail {
  private route = inject(ActivatedRoute);
  private data = toSignal(this.route.data);
  user = computed(() => this.data().user as User);
  settings = computed(() => this.data().settings as Settings);
}
```

### Method 2: Component Input Binding

Configure the router with `withComponentInputBinding()`:

```typescript
import {bootstrapApplication} from '@angular/platform-browser';
import {provideRouter, withComponentInputBinding} from '@angular/router';
import {routes} from './app.routes';

bootstrapApplication(App, {
  providers: [provideRouter(routes, withComponentInputBinding())],
});
```

Then use inputs in components:

```typescript
import {Component, input} from '@angular/core';
import type {User, Settings} from './types';

@Component({
  template: `
    <h1>{{ user().name }}</h1>
    <p>{{ user().email }}</p>
    <div>Theme: {{ settings().theme }}</div>
  `,
})
export class UserDetail {
  user = input.required<User>();
  settings = input.required<Settings>();
}
```

This approach provides better type safety and eliminates the need to inject `ActivatedRoute` just to access resolved data.

## Error Handling

### Centralized Approach with `withNavigationErrorHandler`

```typescript
import {bootstrapApplication} from '@angular/platform-browser';
import {provideRouter, withNavigationErrorHandler} from '@angular/router';
import {inject} from '@angular/core';
import {Router} from '@angular/router';
import {routes} from './app.routes';

bootstrapApplication(App, {
  providers: [
    provideRouter(
      routes,
      withNavigationErrorHandler((error) => {
        const router = inject(Router);
        if (error?.message) {
          console.error('Navigation error occurred:', error.message);
        }
        router.navigate(['/error']);
      }),
    ),
  ],
});
```

### Router Events Subscription

```typescript
import {Component, inject, signal} from '@angular/core';
import {Router, NavigationError} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';
import {map} from 'rxjs';

@Component({
  selector: 'app-root',
  template: `
    @if (errorMessage()) {
      <div class="error-banner">
        {{ errorMessage() }}
        <button (click)="retryNavigation()">Retry</button>
      </div>
    }
    <router-outlet />
  `,
})
export class App {
  private router = inject(Router);
  private lastFailedUrl = signal('');
  private navigationErrors = toSignal(
    this.router.events.pipe(
      map((event) => {
        if (event instanceof NavigationError) {
          this.lastFailedUrl.set(event.url);
          if (event.error) {
            console.error('Navigation error', event.error);
          }
          return 'Navigation failed. Please try again.';
        }
        return '';
      }),
    ),
    {initialValue: ''},
  );
  errorMessage = this.navigationErrors;
  retryNavigation() {
    if (this.lastFailedUrl()) {
      this.router.navigateByUrl(this.lastFailedUrl());
    }
  }
}
```

### Direct Resolver Error Handling

```typescript
import {inject} from '@angular/core';
import {ResolveFn, RedirectCommand, Router} from '@angular/router';
import {catchError, of, EMPTY} from 'rxjs';
import {UserStore} from './user-store';
import type {User} from './types';

export const userResolver: ResolveFn<User | RedirectCommand> = (route) => {
  const userStore = inject(UserStore);
  const router = inject(Router);
  const userId = route.paramMap.get('id')!;
  return userStore.getUser(userId).pipe(
    catchError((error) => {
      console.error('Failed to load user:', error);
      return of(new RedirectCommand(router.parseUrl('/users')));
    }),
  );
};
```

## Navigation Loading Feedback

Since navigation blocks during resolver execution, show loading indicators using router events:

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

## Best Practices

- Maintain lightweight resolvers focused on essential data only
- Always implement graceful error handling
- Apply caching strategies to improve performance
- Provide visual feedback during resolver execution since navigation is blocked
- Establish reasonable timeouts to prevent indefinite blocking
- Leverage TypeScript interfaces for type safety

## Parent-Child Resolver Data Access

Resolvers execute parent-to-child, making parent resolved data accessible to children:

```typescript
import { inject } from '@angular/core';
import { provideRouter, ActivatedRouteSnapshot } from '@angular/router';
import { userResolver } from './resolvers';
import { UserPosts } from './pages';
import { PostService } from './services';
import type { User } from './types';

provideRouter([
  {
    path: 'users/:id',
    resolve: { user: userResolver },
    children: [
      {
        path: 'posts',
        component: UserPosts,
        resolve: {
          posts: (route: ActivatedRouteSnapshot) => {
            const postService = inject(PostService);
            const user = route.parent?.data['user'] as User;
            const userId = user.id;
            return postService.getPostByUser(userId);
          },
        },
      },
    ],
  },
]);
```
