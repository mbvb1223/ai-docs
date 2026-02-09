# Read Route State
> Source: https://angular.dev/guide/routing/read-route-state

## Overview

Angular Router enables developers to access and utilize information associated with routes for creating responsive, context-aware components. This guide covers accessing route data, parameters, and detecting active routes.

## ActivatedRoute Service

`ActivatedRoute` from `@angular/router` provides comprehensive information about the current route.

### Basic Usage

```typescript
import {Component} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product',
})
export class Product {
  private activatedRoute = inject(ActivatedRoute);
  constructor() {
    console.log(this.activatedRoute);
  }
}
```

### Common Properties

| Property | Details |
|----------|---------|
| `url` | Observable of route paths as string arrays |
| `data` | Observable containing route data and resolved values |
| `params` | Observable with route-specific parameters |
| `queryParams` | Observable with query parameters available to all routes |

See the [ActivatedRoute API documentation](/api/router/ActivatedRoute) for complete details.

## Route Snapshots

Route snapshots represent the router state at a specific moment. They are static and will not reflect future changes -- useful for one-time access to route information.

### Accessing Snapshots

```typescript
import {ActivatedRoute, ActivatedRouteSnapshot} from '@angular/router';

@Component({/*...*/})
export class UserProfile {
  readonly userId: string;
  private route = inject(ActivatedRoute);

  constructor() {
    // Example URL: https://www.angular.dev/users/123?role=admin&status=active#contact
    this.userId = this.route.snapshot.paramMap.get('id');

    const snapshot = this.route.snapshot;
    console.log({
      url: snapshot.url,
      params: snapshot.params,  // {id: '123'}
      queryParams: snapshot.queryParams, // {role: 'admin', status: 'active'}
    });
  }
}
```

Refer to [ActivatedRoute](/api/router/ActivatedRoute) and [ActivatedRouteSnapshot](/api/router/ActivatedRouteSnapshot) API docs.

## Reading Route Parameters

### Route Parameters

Route parameters pass data through URLs for displaying specific content based on identifiers.

**Define parameters** using colon prefix:

```typescript
import {Routes} from '@angular/router';
import {Product} from './product';

const routes: Routes = [{path: 'product/:id', component: Product}];
```

**Access parameters** by subscribing to `route.params`:

```typescript
import {Component, inject, signal} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-product-detail',
  template: `<h1>Product Details: {{ productId() }}</h1>`,
})
export class ProductDetail {
  productId = signal('');
  private activatedRoute = inject(ActivatedRoute);

  constructor() {
    this.activatedRoute.params.subscribe((params) => {
      this.productId.set(params['id']);
    });
  }
}
```

### Query Parameters

Query parameters provide flexible optional data without affecting route structure. Ideal for filtering, sorting, pagination, and state management.

**Navigation with query parameters:**

```typescript
// Single parameter: /products?category=electronics
router.navigate(['/products'], {
  queryParams: {category: 'electronics'},
});

// Multiple parameters: /products?category=electronics&sort=price&page=1
router.navigate(['/products'], {
  queryParams: {
    category: 'electronics',
    sort: 'price',
    page: 1,
  },
});
```

**Access query parameters:**

```typescript
import {ActivatedRoute, Router} from '@angular/router';

@Component({
  selector: 'app-product-list',
  template: `
    <div>
      <select (change)="updateSort($event)">
        <option value="price">Price</option>
        <option value="name">Name</option>
      </select>
      <!-- Products list -->
    </div>
  `,
})
export class ProductList {
  private route = inject(ActivatedRoute);
  private router = inject(Router);

  constructor() {
    this.route.queryParams.subscribe((params) => {
      const sort = params['sort'] || 'price';
      const page = Number(params['page']) || 1;
      this.loadProducts(sort, page);
    });
  }

  updateSort(event: Event) {
    const sort = (event.target as HTMLSelectElement).value;
    this.router.navigate([], {
      queryParams: {sort},
      queryParamsHandling: 'merge', // Preserve other parameters
    });
  }
}
```

See [QueryParamsHandling API](/api/router/QueryParamsHandling) for details.

### Matrix Parameters

Matrix parameters are optional, segment-specific parameters using semicolons (`;`) instead of query strings. They are scoped to individual path segments.

**URL format:** `/path;key=value` or `/path;key1=value1;key2=value2`

**Navigation with matrix parameters:**

```typescript
this.router.navigate(['/awesome-products', {view: 'grid', filter: 'new'}]);
// Results in: /awesome-products;view=grid;filter=new
```

**Accessing via ActivatedRoute:**

```typescript
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component(/* ... */)
export class AwesomeProducts {
  private route = inject(ActivatedRoute);

  constructor() {
    this.route.params.subscribe((params) => {
      const view = params['view'];    // 'grid'
      const filter = params['filter']; // 'new'
    });
  }
}
```

**Note:** Matrix parameters also work with [`withComponentInputBinding`](/api/router/withComponentInputBinding).

## RouterLinkActive: Detecting Active Routes

The `RouterLinkActive` directive dynamically styles navigation elements based on the active route -- common for highlighting current navigation items.

### Basic Usage

```html
<nav>
  <a
    class="button"
    routerLink="/about"
    routerLinkActive="active-button"
    ariaCurrentWhenActive="page"
  >
    About
  </a>
  |
  <a
    class="button"
    routerLink="/settings"
    routerLinkActive="active-button"
    ariaCurrentWhenActive="page"
  >
    Settings
  </a>
</nav>
```

Angular applies `active-button` and sets `ariaCurrentWhenActive` to `page` when URLs match.

### Multiple Classes

```html
<!-- Space-separated string -->
<a routerLink="/user/bob" routerLinkActive="class1 class2">Bob</a>

<!-- Array syntax -->
<a routerLink="/user/bob" [routerLinkActive]="['class1', 'class2']">Bob</a>
```

### Route Matching Strategy

By default, `RouterLinkActive` considers ancestors as matches:

```html
<a [routerLink]="['/user/jane']" routerLinkActive="active-link">User</a>
<a [routerLink]="['/user/jane/role/admin']" routerLinkActive="active-link">Role</a>
```

At `/user/jane/role/admin`, both links receive `active-link`.

### Exact Matching

For exact matches only, use `routerLinkActiveOptions`:

```html
<a
  [routerLink]="['/user/jane']"
  routerLinkActive="active-link"
  [routerLinkActiveOptions]="{exact: true}">
  User
</a>
<a
  [routerLink]="['/user/jane/role/admin']"
  routerLinkActive="active-link"
  [routerLinkActiveOptions]="{exact: true}">
  Role
</a>
```

**Equivalent matching options:**

```typescript
// `exact: true` equals:
{
  paths: 'exact',
  fragment: 'ignored',
  matrixParams: 'ignored',
  queryParams: 'exact',
}

// `exact: false` equals:
{
  paths: 'subset',
  fragment: 'ignored',
  matrixParams: 'ignored',
  queryParams: 'subset',
}
```

See [IsActiveMatchOptions API](/api/router/IsActiveMatchOptions).

### Applying to Ancestors

Apply `RouterLinkActive` to ancestor elements:

```html
<div routerLinkActive="active-link" [routerLinkActiveOptions]="{exact: true}">
  <a routerLink="/user/jim">Jim</a>
  <a routerLink="/user/bob">Bob</a>
</div>
```

Refer to [RouterLinkActive API](/api/router/RouterLinkActive).

## Checking if a URL is Active

The `isActive` function returns a computed signal tracking whether a URL is currently active. It updates automatically with router state changes.

```typescript
import {Component, inject} from '@angular/core';
import {isActive, Router} from '@angular/router';

@Component({
  template: `
    <div [class.active]="isSettingsActive()">
      <h2>Settings</h2>
    </div>
  `,
})
export class Panel {
  private router = inject(Router);

  isSettingsActive = isActive('/settings', this.router, {
    paths: 'subset',
    queryParams: 'ignored',
    fragment: 'ignored',
    matrixParams: 'ignored',
  });
}
```

## Additional Resources

- [ActivatedRoute API](/api/router/ActivatedRoute)
- [ActivatedRouteSnapshot API](/api/router/ActivatedRouteSnapshot)
- [RouterLinkActive API](/api/router/RouterLinkActive)
- [QueryParamsHandling API](/api/router/QueryParamsHandling)
- [IsActiveMatchOptions API](/api/router/IsActiveMatchOptions)
