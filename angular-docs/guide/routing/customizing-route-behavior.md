# Customizing Route Behavior
> Source: https://angular.dev/guide/routing/customizing-route-behavior

## Overview

Angular Router offers extension points for customizing how routes function. Before implementing custom strategies, ensure the default router behavior does not meet your needs. Custom implementations work best for component state preservation, strategic lazy loading, external URL integration, and dynamic matching beyond simple patterns.

## Router Configuration Options

The `withRouterConfig` function accepts `RouterConfigOptions` to adjust Router behavior:

### Canceled Navigation Resolution

The `canceledNavigationResolution` setting controls browser history restoration when navigation is canceled. Default `'replace'` reverts to the pre-navigation URL, while `'computed'` keeps the history index synchronized with Angular navigation.

### Same-URL Navigation Handling

`onSameUrlNavigation` determines behavior when users navigate to the current URL. Default `'ignore'` skips processing; `'reload'` re-runs guards and resolvers. Individual navigations can override this setting.

### Parameter Inheritance Strategy

`paramsInheritanceStrategy` defines how route parameters flow from parent routes. Default `'emptyOnly'` inherits only when the child path is empty; `'always'` ensures parameters are available throughout the route tree.

### URL Update Timing

`urlUpdateStrategy` controls when the address bar updates. Default `'deferred'` waits for successful navigation; `'eager'` updates immediately when navigation starts.

### Query Parameter Handling

`defaultQueryParamsHandling` sets fallback behavior for `Router.createUrlTree`. Options include `'replace'` (default), `'merge'`, and `'preserve'`.

## Route Reuse Strategy

Route reuse strategy determines whether Angular destroys and recreates components during navigation or preserves them. Custom strategies benefit applications needing form state preservation, expensive data retention, scroll position maintenance, or tab-like interfaces.

### Custom Implementation

Implementing `RouteReuseStrategy` requires five core methods:

- **shouldDetach**: Determines if a route should be stored for reuse
- **store**: Saves the detached route handle
- **shouldAttach**: Checks if a stored route should be reattached
- **retrieve**: Returns the previously stored route handle
- **shouldReuseRoute**: Decides whether to reuse the current route instance

Routes opt into reuse through metadata: `data: {reuse: true}`. Register strategies via dependency injection: `{provide: RouteReuseStrategy, useClass: CustomRouteReuseStrategy}`.

### Destroying Detached Handles

Use the `destroyDetachedRouteHandle` function to manually clean up detached route handles when discarding them without reattachment, preventing memory leaks.

### Experimental Auto-Cleanup

The `withExperimentalAutoCleanupInjectors` feature automatically destroys injectors of detached routes no longer stored by the strategy. Custom strategies not extending `BaseRouteReuseStrategy` must implement `shouldDestroyInjector` to indicate which routes should have injectors destroyed.

## Preloading Strategy

Preloading strategies determine when Angular loads lazy-loaded route modules in the background, eliminating delays when users first navigate to lazy routes.

### Built-in Strategies

- **NoPreloading**: Default; modules load only when users navigate
- **PreloadAllModules**: Loads all lazy modules after initial navigation

### Custom Preloading Strategy

Custom strategies implement `PreloadingStrategy` with a single `preload` method. Routes opt in through metadata: `data: {preload: true}`.

Example selective strategy checks route metadata and only preloads marked routes. Routes indicate preference without automatic preloading.

### Performance Considerations

Preloading impacts network usage and memory consumption. Timing matters -- immediate preloading might compete with critical resources. Browser resource limits affect concurrent connections. Service workers can provide fine-grained control over caching and network requests.

## URL Handling Strategy

The `UrlHandlingStrategy` class controls which URLs Angular processes versus ignores. This becomes essential when migrating applications incrementally or sharing URL space with other frameworks.

Custom strategies implement three methods:

- **shouldProcessUrl**: Determines if Angular handles the URL
- **extract**: Returns the URL portion Angular should process
- **merge**: Combines URL fragment with the rest

Register via dependency injection: `{provide: UrlHandlingStrategy, useClass: CustomUrlHandlingStrategy}`.

This pattern creates clear boundaries -- Angular handles specific paths while ignoring others, useful for legacy application migration.

## Custom Route Matchers

Custom matchers enable sophisticated matching logic based on runtime conditions, complex patterns, or business rules where standard path matching falls short.

### Basic Implementation

A matcher function receives URL segments and returns either a match result with consumed segments and parameters, or null for no match. Matchers run before Angular evaluates the path property.

### Use Cases

**Version-based routing**: Match patterns like `/v1/docs` or `/v2.1/docs`, extracting version and section parameters for documentation sites.

**Locale-aware routing**: Extract locale codes from URLs (e.g., `/en/`, `/es/`) and route to appropriate components while making locale available as a parameter.

**Complex business logic**: Implement product matching where different URL patterns apply based on type -- books use ISBN, electronics use SKU format, clothing includes brand information.

### Performance Considerations

Custom matchers run for every navigation until a match succeeds. Keep logic focused and efficient -- return early when matches are impossible, avoid expensive operations, and consider caching repeated patterns. Reserve custom matchers for scenarios where standard matching genuinely falls short.
