# Other Common Routing Tasks
> Source: https://angular.dev/guide/routing/common-router-tasks

## Overview

This guide addresses frequent routing scenarios in Angular applications, covering practical implementation patterns for navigation, parameter passing, and error handling.

## Getting Route Information

### Passing Data Between Components

When users navigate through an application, you often need to transfer information between components. For example, in a shopping application where each grocery item has a unique identifier, an `EditGroceryItem` component needs to receive the item's ID to display the correct details.

### Implementation Steps

**1. Enable Component Input Binding**

Add the `withComponentInputBinding` feature to your router configuration:

```typescript
providers: [provideRouter(appRoutes, withComponentInputBinding())];
```

Alternatively, use the `bindToComponentInputs` option with `RouterModule.forRoot`.

**2. Create Component Inputs**

Update your component to include an `input()` property matching the route parameter name:

```typescript
id = input.required<string>();
hero = computed(() => this.service.getHero(id()));
```

**3. Handle Optional Values with Defaults**

When `withComponentInputBinding` is active, the router assigns `undefined` if no matching route data exists. Manage this by:

- Using the `transform` option on inputs
- Implementing `linkedSignal` for local state management

```typescript
id = input.required({
  transform: (maybeUndefined: string | undefined) => maybeUndefined ?? '0',
});
// OR
id = input<string | undefined>();
internalId = linkedSignal(() => this.id() ?? getDefaultId());
```

**Important Note:** This feature binds all route data as component inputs: static route data, resolved data, path parameters, matrix parameters, and query parameters. To access parent component route info, configure `paramsInheritanceStrategy: 'always'` in router settings.

## Displaying a 404 Page

Implement a catch-all route using a wildcard path to handle undefined routes:

```typescript
const routes: Routes = [
  {path: 'first-component', component: First},
  {path: 'second-component', component: Second},
  {path: '**', component: PageNotFound}, // Catch-all for 404
];
```

The router matches routes sequentially. When no earlier path matches the requested URL, the `**` wildcard route activates, displaying your 404 component.

## Link Parameters Array

Link parameter arrays contain two essential elements:

- Route path to the destination component
- Required and optional route parameters for the URL

### Basic Navigation

```html
<a [routerLink]="['/heroes']">Heroes</a>
```

### With Route Parameters

```html
<a [routerLink]="['/hero', hero.id]">
  <span class="badge">{{ hero.id }}</span>{{ hero.name }}
</a>
```

### With Optional Parameters

```html
<a [routerLink]="['/crisis-center', {foo: 'foo'}]">Crisis Center</a>
```

This syntax uses matrix parameters -- optional values tied to specific URL segments.

### Nested Route Navigation

For applications with child routes, construct arrays hierarchically:

```html
<a [routerLink]="['/crisis-center']">Crisis Center</a>
<a [routerLink]="['/crisis-center', 1]">Dragon Crisis</a>
<a [routerLink]="['/crisis-center', 2]">Shark Crisis</a>
```

The first array element identifies the parent route. Subsequent elements specify child routes and parameters. This pattern scales to multiple routing levels.

## LocationStrategy and Browser URL Styles

The router updates browser location and history when navigating to new components. Angular supports two URL formatting approaches:

| Strategy | Style | Example |
|----------|-------|---------|
| `PathLocationStrategy` | HTML5 pushState (default) | `localhost:3002/crisis-center` |
| `HashLocationStrategy` | Hash-based URLs | `localhost:3002/src/#/crisis-center` |

Modern browsers support `history.pushState`, enabling clean URLs without server requests. Older browsers require hash-based URLs to prevent page reloads.

Configure the location strategy through the router provider during application bootstrap. The default behavior uses `PathLocationStrategy`; override this during bootstrapping to use `HashLocationStrategy` if needed.
