# Router Reference
> Source: https://angular.dev/guide/routing/router-reference

## Overview

This guide covers core router concepts and terminology for Angular's routing system.

## Router Events

During navigation, the `Router` emits events through `Router.events`. Here are the key events:

| Event | Description |
|-------|-------------|
| `NavigationStart` | Triggered when navigation begins |
| `RouteConfigLoadStart` | Triggered before lazy loading a route configuration |
| `RouteConfigLoadEnd` | Triggered after a route is lazy loaded |
| `RoutesRecognized` | Triggered when the Router parses the URL and recognizes routes |
| `GuardsCheckStart` | Triggered when the Router starts the Guards phase |
| `ChildActivationStart` | Triggered when the Router begins activating a route's children |
| `ActivationStart` | Triggered when the Router begins activating a route |
| `GuardsCheckEnd` | Triggered when the Guards phase completes successfully |
| `ResolveStart` | Triggered when the Router begins the Resolve phase |
| `ResolveEnd` | Triggered when the Resolve phase completes successfully |
| `ChildActivationEnd` | Triggered when child route activation completes |
| `ActivationEnd` | Triggered when route activation completes |
| `NavigationEnd` | Triggered when navigation ends successfully |
| `NavigationCancel` | Triggered when navigation is canceled (guard returns false or redirects) |
| `NavigationError` | Triggered when navigation fails due to an unexpected error |
| `Scroll` | Represents a scrolling event |

**Note:** Enable `withDebugTracing` to log these events to the console.

## Router Terminology

Key router terms and definitions:

| Term | Definition |
|------|------------|
| `Router` | Displays the application component for the active URL and manages navigation |
| `provideRouter` | Provides necessary service providers for application view navigation |
| `RouterModule` | An NgModule providing service providers and directives for navigation |
| `Routes` | An array of Route definitions, each mapping a URL path to a component |
| `Route` | Defines how the router navigates to a component based on a URL pattern |
| `RouterOutlet` | The directive (`<router-outlet>`) marking where the router displays views |
| `RouterLink` | Directive binding clickable HTML elements to routes |
| `RouterLinkActive` | Directive for adding/removing classes when associated routes become active |
| `ActivatedRoute` | Service providing route-specific information (parameters, data, queries, fragments) |
| `RouterState` | Current router state including the activated routes tree |
| Link parameters array | An array interpreted as a routing instruction for `RouterLink` or `Router.navigate()` |
| Routing component | An Angular component with `RouterOutlet` that displays route-based views |

## Base href

The router uses browser `history.pushState` for navigation, enabling custom in-application URL paths like `localhost:4200/crisis-center`.

### Adding Base href

Include a `<base href>` element in your `index.html`:

```html
<base href="/" />
```

Place it immediately after the `<head>` tag. This element helps the browser load resources when deep linking into the application.

### HTML5 URLs and Base href Structure

URLs have these components:

```
foo://example.com:8042/over/there?name=ferret#nose
 \_/ \____________/ \________/ \_________/ \__/
  |       |            |            |         |
scheme  authority    path          query   fragment
```

### Alternative Configuration

If you cannot add the `<base>` element directly, you can:

1. Provide an `APP_BASE_HREF` value to the router
2. Use root URLs (with an authority) for all web resources (CSS, images, scripts, templates)

**Important guidelines:**

- The `<base href>` path should end with "/"
- If `<base href>` includes a query part, it is only used when the link path is empty and lacks a query
- Root URLs ignore the `<base href>` value
- Fragments in `<base href>` are never persisted

Refer to RFC 3986 section 5.2.2 for complete information on URI construction.

### HashLocationStrategy

Use hash-based routing by providing `useHash: true`:

```typescript
providers: [provideRouter(appRoutes, withHashLocation())];
```

For `RouterModule.forRoot()`, configure with:

```typescript
RouterModule.forRoot(routes, {useHash: true})
```
