# Testing Routing and Navigation
> Source: https://angular.dev/guide/routing/testing

## Overview

Testing routing and navigation ensures applications behave correctly when users move between different routes. This guide covers various strategies for testing routing functionality.

## Prerequisites

To follow this guide, familiarity with these tools is recommended:

- **Vitest** - A JavaScript testing framework providing `describe`, `it`, and `expect` syntax
- **Angular Testing Utilities** - Including `TestBed` and `ComponentFixture`
- **RouterTestingHarness** - A specialized test harness for routed components with built-in navigation capabilities

## Testing Scenarios

### Route Parameters

Components often depend on route parameters from the URL to display or fetch data. Here is how to test a component that reads user IDs from routes:

```typescript
// user-profile.spec.ts
import {TestBed} from '@angular/core/testing';
import {Router} from '@angular/router';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter} from '@angular/router';
import {UserProfile} from './user-profile';

describe('UserProfile', () => {
  it('should display user ID from route parameters', async () => {
    TestBed.configureTestingModule({
      imports: [UserProfile],
      providers: [provideRouter([{path: 'user/:id', component: UserProfile}])],
    });
    const harness = await RouterTestingHarness.create();
    await harness.navigateByUrl('/user/123', UserProfile);
    expect(harness.routeNativeElement?.textContent).toContain('User Profile: 123');
  });
});
```

The corresponding component implementation:

```typescript
// user-profile.ts
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  template: '<h1>User Profile: {{userId}}</h1>',
})
export class UserProfile {
  private route = inject(ActivatedRoute);
  userId: string | null = this.route.snapshot.paramMap.get('id');
}
```

### Route Guards

Guards control route access based on conditions like authentication. This example tests an authentication guard:

```typescript
// auth.guard.spec.ts
import {vi, type Mocked} from 'vitest';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter, Router} from '@angular/router';
import {authGuard} from './auth.guard';
import {AuthStore} from './auth-store';
import {Component} from '@angular/core';
import {TestBed} from '@angular/core/testing';

@Component({template: '<h1>Protected Page</h1>'})
class Protected {}

@Component({template: '<h1>Login Page</h1>'})
class Login {}

describe('authGuard', () => {
  let authStore: Mocked<AuthStore>;
  let harness: RouterTestingHarness;

  async function setup(isAuthenticated: boolean) {
    authStore = {
      isAuthenticated: vi.fn().mockReturnValue(isAuthenticated)
    } as Mocked<AuthStore>;

    TestBed.configureTestingModule({
      providers: [
        {provide: AuthStore, useValue: authStore},
        provideRouter([
          {path: 'protected', component: Protected, canActivate: [authGuard]},
          {path: 'login', component: Login},
        ]),
      ],
    });
    harness = await RouterTestingHarness.create();
  }

  it('allows navigation when user is authenticated', async () => {
    await setup(true);
    await harness.navigateByUrl('/protected', Protected);
    expect(harness.routeNativeElement?.textContent).toContain('Protected Page');
  });

  it('redirects to login when user is not authenticated', async () => {
    await setup(false);
    await harness.navigateByUrl('/protected', Login);
    expect(harness.routeNativeElement?.textContent).toContain('Login Page');
  });
});
```

The guard implementation:

```typescript
// auth.guard.ts
import {inject} from '@angular/core';
import {CanActivateFn, Router} from '@angular/router';
import {AuthStore} from './auth-store';

export const authGuard: CanActivateFn = () => {
  const authStore = inject(AuthStore);
  const router = inject(Router);
  return authStore.isAuthenticated() ? true : router.parseUrl('/login');
};
```

### Router Outlets

Testing router outlets involves verifying that different components display for different routes:

```typescript
// app.spec.ts
import {TestBed} from '@angular/core/testing';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter} from '@angular/router';
import {Component} from '@angular/core';
import {App} from './app';

@Component({
  template: '<h1>Home Page</h1>',
})
class MockHome {}

@Component({
  template: '<h1>About Page</h1>',
})
class MockAbout {}

describe('App Router Outlet', () => {
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [App],
      providers: [
        provideRouter([
          {path: '', component: MockHome},
          {path: 'about', component: MockAbout},
        ]),
      ],
    });
    harness = await RouterTestingHarness.create();
  });

  it('should display home component for default route', async () => {
    await harness.navigateByUrl('');
    expect(harness.routeNativeElement?.textContent).toContain('Home Page');
  });

  it('should display about component for about route', async () => {
    await harness.navigateByUrl('/about');
    expect(harness.routeNativeElement?.textContent).toContain('About Page');
  });
});
```

The app component:

```typescript
// app.ts
import {Component} from '@angular/core';
import {RouterOutlet, RouterLink} from '@angular/router';

@Component({
  imports: [RouterOutlet, RouterLink],
  template: `
    <nav>
      <a routerLink="/">Home</a>
      <a routerLink="/about">About</a>
    </nav>
    <router-outlet />
  `,
})
export class App {}
```

### Nested Routes

Testing nested routes requires verifying that both parent and child components render correctly:

```typescript
// nested-routes.spec.ts
import {TestBed} from '@angular/core/testing';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter} from '@angular/router';
import {Parent, Child} from './nested-components';

describe('Nested Routes', () => {
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [Parent, Child],
      providers: [
        provideRouter([
          {
            path: 'parent',
            component: Parent,
            children: [{path: 'child', component: Child}],
          },
        ]),
      ],
    });
    harness = await RouterTestingHarness.create();
  });

  it('should render parent and child components for nested route', async () => {
    await harness.navigateByUrl('/parent/child');
    expect(harness.routeNativeElement?.textContent).toContain('Parent Component');
    expect(harness.routeNativeElement?.textContent).toContain('Child Component');
  });
});
```

Corresponding components:

```typescript
// nested.ts
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  imports: [RouterOutlet],
  template: `
    <h1>Parent Component</h1>
    <router-outlet />
  `,
})
export class Parent {}

@Component({
  template: '<h2>Child Component</h2>',
})
export class Child {}
```

### Query Parameters and Fragments

Query parameters and URL fragments provide optional URL data that does not trigger route changes but affects component behavior:

```typescript
// search.spec.ts
import {TestBed} from '@angular/core/testing';
import {Router, provideRouter} from '@angular/router';
import {RouterTestingHarness} from '@angular/router/testing';
import {Search} from './search';

describe('Search', () => {
  let component: Search;
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [Search],
      providers: [provideRouter([{path: 'search', component: Search}])],
    });
    harness = await RouterTestingHarness.create();
  });

  it('should read search term from query parameters', async () => {
    component = await harness.navigateByUrl('/search?q=angular', Search);
    expect(component.searchTerm()).toBe('angular');
  });
});
```

The search component:

```typescript
// search.ts
import {Component, inject, computed} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';

@Component({
  template: '<div>Search term: {{searchTerm()}}</div>',
})
export class Search {
  private route = inject(ActivatedRoute);
  private queryParams = toSignal(this.route.queryParams, {initialValue: {}});
  searchTerm = computed(() => this.queryParams()['q'] || null);
}
```

## Best Practices for Router Testing

1. **Use RouterTestingHarness** - This provides a cleaner API than creating test host components, offering direct component access and built-in navigation.

2. **Handle external dependencies thoughtfully** - Prefer real implementations for more realistic tests. Use fakes that approximate real behavior; reserve mocks as a last resort.

3. **Test navigation state** - Verify both the navigation action and resulting application state, including URL changes and component rendering.

4. **Handle asynchronous operations** - Router navigation is asynchronous. Use `async/await` to properly manage timing in tests.

5. **Test error scenarios** - Include tests for invalid routes, failed navigation, and guard rejections to ensure graceful error handling.

6. **Do not mock Angular Router** - Provide real route configurations and use the harness for navigation. This approach is more robust and catches real issues when the router updates, since mocks can hide breaking changes.
