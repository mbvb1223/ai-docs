# Define Routes
> Source: https://angular.dev/guide/routing/define-routes

## Overview

Routes are fundamental building blocks for navigation in Angular applications. A route is an object that specifies which component renders for a given URL path, along with additional configuration options.

## Basic Route Structure

A basic route example:

```typescript
import {AdminPage} from './app-admin';

const adminPage = {
  path: 'admin',
  component: AdminPage,
};
```

When users visit `/admin`, the `AdminPage` component displays.

## Managing Routes

Most projects define routes in a dedicated file (e.g., `app.routes.ts`):

```typescript
import {Routes} from '@angular/router';
import {HomePage} from './home-page';
import {AdminPage} from './about-page';

export const routes: Routes = [
  {
    path: '',
    component: HomePage,
  },
  {
    path: 'admin',
    component: AdminPage,
  },
];
```

## Adding the Router

Include the router in your application using `provideRouter`:

```typescript
import {ApplicationConfig} from '@angular/core';
import {provideRouter} from '@angular/router';
import {routes} from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    // ...
  ],
};
```

## Route URL Paths

### Static Paths

Routes with fixed, unchanging paths:
- `/admin`
- `/blog`
- `/settings/account`

### Dynamic Paths with Parameters

Define parameterized routes using colon (`:`) prefix:

```typescript
import {Routes} from '@angular/router';
import {UserProfile} from './user-profile/user-profile';

const routes: Routes = [{path: 'user/:id', component: UserProfile}];
```

Valid parameter names contain letters, numbers, underscores, or hyphens. URLs like `/user/leeroy` and `/user/jenkins` match this pattern.

Multiple parameters are supported:

```typescript
const routes: Routes = [
  {path: 'user/:id/:social-media', component: SocialMediaFeed},
  {path: 'user/:id/', component: UserProfile},
];
```

**Note:** Parameters differ from query string data. See the route state guide for information about "query parameters."

### Wildcards

The double asterisk (`**`) catches all unmatched routes:

```typescript
import {Home} from './home/home';
import {UserProfile} from './user-profile';
import {NotFound} from './not-found';

const routes: Routes = [
  {path: 'home', component: Home},
  {path: 'user/:id', component: UserProfile},
  {path: '**', component: NotFound},
];
```

Wildcard routes belong at the end of the routes array.

## URL Matching Strategy

Angular uses first-match-wins logic. Define routes from most to least specific:

```typescript
const routes: Routes = [
  {path: '', component: Home},
  {path: 'users/new', component: NewUser},
  {path: 'users/:id', component: UserDetail},
  {path: 'users', component: Users},
  {path: '**', component: NotFound},
];
```

For `/users/new`, Angular stops at the second route and never evaluates later matching options.

## Route Loading Strategies

### Eager Loading

Components referenced directly in route configuration load immediately:

```typescript
import {Routes} from '@angular/router';
import {HomePage} from './components/home/home-page';
import {LoginPage} from './components/auth/login-page';

export const routes: Routes = [
  {
    path: '',
    component: HomePage,
  },
  {
    path: 'login',
    component: LoginPage,
  },
];
```

Eager loading includes all component code in the initial bundle, allowing seamless transitions but slower initial load.

### Lazy Loading

Use `loadComponent` and `loadChildren` to load code on demand:

```typescript
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'login',
    loadComponent: () => import('./components/auth/login-page'),
  },
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component'),
    loadChildren: () => import('./admin/admin.routes'),
  },
];
```

Lazy loading improves initial performance by splitting code into separate bundles loaded only when needed.

### Injection Context Lazy Loading

Loaders execute within the route's injection context, enabling context-aware loading:

```typescript
import {Routes} from '@angular/router';
import {inject} from '@angular/core';
import {FeatureFlags} from './feature-flags';

export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => {
      const flags = inject(FeatureFlags);
      return flags.isPremium
        ? import('./dashboard/premium-dashboard')
        : import('./dashboard/basic-dashboard');
    },
  },
];
```

### Choosing Eager vs. Lazy

Eager loading suits primary landing pages; other pages benefit from lazy loading. Multiple nested lazy routes can negatively impact performance.

## Redirects

Routes can redirect to other routes:

```typescript
import {Blog} from './home/blog';

const routes: Routes = [
  {
    path: 'articles',
    redirectTo: '/blog',
  },
  {
    path: 'blog',
    component: Blog,
  },
];
```

Redirects handle outdated links or bookmarks gracefully.

## Page Titles

Associate titles with routes; Angular automatically updates the page title:

```typescript
import {Routes} from '@angular/router';
import {Home} from './home';
import {About} from './about';

const routes: Routes = [
  {
    path: '',
    component: Home,
    title: 'Home Page',
  },
  {
    path: 'about',
    component: About,
    title: 'About Us',
  },
];
```

Titles can be dynamic using `ResolveFn`:

```typescript
const titleResolver: ResolveFn<string> = (route) => route.queryParams['id'];

const routes: Routes = [
  {
    path: 'products',
    component: Products,
    title: titleResolver,
  },
];
```

### Custom Title Strategy

Implement `TitleStrategy` for centralized title control:

```typescript
import {inject, Injectable} from '@angular/core';
import {Title} from '@angular/platform-browser';
import {TitleStrategy, RouterStateSnapshot} from '@angular/router';

@Injectable()
export class AppTitleStrategy extends TitleStrategy {
  private readonly title = inject(Title);

  updateTitle(snapshot: RouterStateSnapshot): void {
    const pageTitle = this.buildTitle(snapshot) || this.title.getTitle();
    this.title.setTitle(`MyAwesomeApp - ${pageTitle}`);
  }
}
```

Provide the custom strategy:

```typescript
import {provideRouter, TitleStrategy} from '@angular/router';
import {AppTitleStrategy} from './app-title.strategy';

export const appConfig = {
  providers: [
    provideRouter(routes),
    {provide: TitleStrategy, useClass: AppTitleStrategy}
  ],
};
```

## Route-Level Dependency Injection

Routes include a `providers` property for dependency injection:

```typescript
export const ROUTES: Route[] = [
  {
    path: 'admin',
    providers: [AdminService, {provide: ADMIN_API_KEY, useValue: '12345'}],
    children: [
      {path: 'users', component: AdminUsers},
      {path: 'teams', component: AdminTeams},
    ],
  },
];
```

Only routes within this section access `ADMIN_API_KEY` and `AdminService`.

## Route Data Association

Attach metadata to routes via the `data` property:

```typescript
import {Routes} from '@angular/router';
import {Home} from './home';
import {About} from './about';

const routes: Routes = [
  {
    path: 'about',
    component: About,
    data: {analyticsId: '456'},
  },
  {
    path: '',
    component: Home,
    data: {analyticsId: '123'},
  },
];
```

Read static data by injecting `ActivatedRoute`. Dynamic data uses data resolvers.

## Nested Routes

Child routes create sub-views that change based on URL:

```typescript
const routes: Routes = [
  {
    path: 'product/:id',
    component: Product,
    children: [
      {
        path: 'info',
        component: ProductInfo,
      },
      {
        path: 'reviews',
        component: ProductReviews,
      },
    ],
  },
];
```

Parent components must include `<router-outlet>` to display child routes.

## Next Steps

Learn how to display route contents with outlets and navigate between routes.
