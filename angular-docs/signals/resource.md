# Async Reactivity with Resources

> Source: https://angular.dev/guide/signals/resource

## Overview

The `resource` function provides a mechanism to incorporate asynchronous data into Angular's signal-based reactive system while maintaining synchronous access patterns.

A Resource gives you a way to incorporate async data into your application's signal-based code and still allow you to access its data synchronously.

**Status:** The `resource` API is currently experimental and subject to change before stabilization.

## Core Concept

While signal APIs (`signal`, `computed`, `input`) operate synchronously, applications frequently require asynchronous data handling. Resources bridge this gap, particularly for server data fetching scenarios.

## Basic Usage

```typescript
import {resource, Signal} from '@angular/core';

const userId: Signal<string> = getUserId();
const userResource = resource({
  // Define a reactive computation.
  // The params value recomputes whenever any read signals change.
  params: () => ({id: userId()}),
  // Define an async loader that retrieves data.
  // The resource calls this function every time the `params` value changes.
  loader: ({params}) => fetchUser(params),
});

// Create a computed signal based on the result of the resource's loader function.
const firstName = computed(() => {
  if (userResource.hasValue()) {
    // `hasValue` serves 2 purposes:
    // - It acts as type guard to strip `undefined` from the type
    // - It protects against reading a throwing `value` when the resource is in error state
    return userResource.value().firstName;
  }
  // fallback in case the resource value is `undefined` or if the resource is in error state
  return undefined;
});
```

## Resource Options

The `resource` function accepts a `ResourceOptions` object with two primary properties:

### params

A reactive computation producing parameter values. Mirrors `computed` behavior -- whenever signals read during execution change, the resource generates a new parameter value.

### loader

An asynchronous function (ResourceLoader) that retrieves state. The resource invokes this function whenever the `params` computation produces a new value, passing that value as an argument.

Resources expose a `value` signal containing the loader's results.

---

## Resource Loaders

A ResourceLoader accepts a single parameter -- a `ResourceLoaderParams` object -- and returns a value.

### ResourceLoaderParams Properties

| Property | Description |
|----------|-------------|
| `params` | The value of the resource's `params` computation |
| `previous` | An object with a `status` property, containing the previous ResourceStatus |
| `abortSignal` | An AbortSignal for request cancellation (see below) |

When the `params` computation returns `undefined`, the loader doesn't execute and the resource status becomes `'idle'`.

### Aborting Requests

Resources automatically abort outstanding operations when `params` changes during loading. Leverage the `abortSignal` parameter to respond to cancellations:

```typescript
const userId: Signal<string> = getUserId();
const userResource = resource({
  params: () => ({id: userId()}),
  loader: ({params, abortSignal}): Promise<User> => {
    // fetch cancels any outstanding HTTP requests when the given `AbortSignal`
    // indicates that the request has been aborted.
    return fetch(`users/${params.id}`, {signal: abortSignal});
  },
});
```

The native `fetch` function accepts `AbortSignal` for request termination. See MDN documentation for comprehensive `AbortSignal` details.

### Reloading

Programmatically trigger a resource's loader using the `reload` method:

```typescript
const userId: Signal<string> = getUserId();
const userResource = resource({
  params: () => ({id: userId()}),
  loader: ({params}) => fetchUser(params),
});
// ...
userResource.reload();
```

---

## Resource Status

The resource object provides multiple signal properties for monitoring asynchronous loader status.

| Property | Description |
|----------|-------------|
| `value` | The most recent value, or `undefined` if none received |
| `hasValue` | Boolean indicating whether the resource possesses a value |
| `error` | The most recent error, or `undefined` if none occurred |
| `isLoading` | Boolean indicating whether the loader is currently executing |
| `status` | Specific ResourceStatus string constant (see table below) |

### ResourceStatus Values

| Status | value() | Description |
|--------|---------|-------------|
| `'idle'` | `undefined` | No valid request; loader hasn't executed |
| `'error'` | `undefined` | The loader encountered an error |
| `'loading'` | `undefined` | Loader running due to `params` value change |
| `'reloading'` | Previous value | Loader running from `reload()` method call |
| `'resolved'` | Resolved value | Loader completed successfully |
| `'local'` | Locally set value | Value set via `.set()` or `.update()` |

Use status information to conditionally render UI elements such as loading indicators and error messages.

---

## HTTP Integration: httpResource

The `httpResource` wrapper around `HttpClient` exposes request status and response as signals. It processes requests through the Angular HTTP stack, including interceptors, providing a convenient signal-based alternative for HTTP operations.
