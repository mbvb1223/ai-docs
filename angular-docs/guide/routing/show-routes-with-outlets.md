# Show Routes with Outlets
> Source: https://angular.dev/guide/routing/show-routes-with-outlets

## Overview

The `RouterOutlet` directive serves as a placeholder indicating where the router should display the component corresponding to the current URL.

## Basic Structure

```html
<app-header />
<!-- Angular inserts your route content here -->
<router-outlet />
<app-footer />
```

```typescript
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  imports: [RouterOutlet],
  templateUrl: './app.html',
  styleUrl: './app.css',
})
export class App {}
```

## Route Configuration Example

```typescript
import {Routes} from '@angular/router';
import {Home} from './home';
import {Products} from './products';

const routes: Routes = [
  {
    path: '',
    component: Home,
    title: 'Home Page',
  },
  {
    path: 'products',
    component: Products,
    title: 'Our Products',
  },
];
```

### Rendering Behavior

When users navigate to `/products`, Angular renders:

```html
<app-header />
<app-products />
<app-footer />
```

Returning to the home page results in:

```html
<app-header />
<app-home />
<app-footer />
```

The `<router-outlet>` element remains in the DOM as a reference point for future navigations, with routed content inserted as a sibling element.

## Nesting Routes with Child Routes

Child routes enable partial page updates when URLs change rather than full page refreshes. These routes require a second `<router-outlet>` in addition to the primary one.

### Child Route Template Example

```html
<h1>Settings</h1>
<nav>
  <ul>
    <li><a routerLink="profile">Profile</a></li>
    <li><a routerLink="security">Security</a></li>
  </ul>
</nav>
<router-outlet />
```

### Child Route Configuration

```typescript
const routes: Routes = [
  {
    path: 'settings',
    component: Settings,
    children: [
      {
        path: 'profile',
        component: Profile,
      },
      {
        path: 'security',
        component: Security,
      },
    ],
  },
];
```

Each child route requires both a `path` and `component`, placed within a `children` array inside the parent route.

## Secondary Routes with Named Outlets

Multiple outlets on a single page can be distinguished by assigning unique names.

```html
<app-header />
<router-outlet />
<router-outlet name="read-more" />
<router-outlet name="additional-actions" />
<app-footer />
```

Each outlet must have a unique name, which cannot be set or changed dynamically. The default outlet name is `'primary'`.

Route configuration matches outlet names:

```typescript
{
  path: 'user/:id',
  component: UserDetails,
  outlet: 'additional-actions'
}
```

## Outlet Lifecycle Events

Four lifecycle events can be emitted by router outlets:

| Event | Description |
|-------|-------------|
| `activate` | Fires when a new component instantiates |
| `deactivate` | Fires when a component is destroyed |
| `attach` | Fires when `RouteReuseStrategy` instructs attachment |
| `detach` | Fires when `RouteReuseStrategy` instructs detachment |

```html
<router-outlet
  (activate)="onActivate($event)"
  (deactivate)="onDeactivate($event)"
  (attach)="onAttach($event)"
  (detach)="onDetach($event)"/>
```

## Passing Contextual Data to Routed Components

The `routerOutletData` input allows passing outlet-specific configuration without modifying route definitions. Routed components access this data as a signal using the `ROUTER_OUTLET_DATA` injection token.

### Dashboard Component

```typescript
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  imports: [RouterOutlet],
  template: `
    <h2>Dashboard</h2>
    <router-outlet [routerOutletData]="{layout: 'sidebar'}" />
  `,
})
export class Dashboard {}
```

### Routed Component Receiving Data

```typescript
import {Component, inject} from '@angular/core';
import {ROUTER_OUTLET_DATA} from '@angular/router';

@Component({
  selector: 'app-stats',
  template: `<p>Stats view (layout: {{ outletData().layout }})</p>`,
})
export class Stats {
  outletData = inject(ROUTER_OUTLET_DATA) as Signal<{layout: string}>;
}
```

When Angular activates the Stats component in that outlet, it receives `{ layout: 'sidebar' }` as injected data.

**Note:** When `routerOutletData` is unset, the injected value defaults to `null`.

## Related Resources

- [API Reference: RouterOutlet](/api/router/RouterOutlet)
- [Navigate to Routes Guide](/guide/routing/navigate-to-routes)
