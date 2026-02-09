# Hybrid Rendering
> Source: https://angular.dev/guide/hybrid-rendering

> **Note:** This page redirects to the main SSR guide. Hybrid rendering is covered as part of Angular's server-side rendering documentation. See [index.md](./index.md) for the full SSR and hybrid rendering guide.

## Overview

Hybrid rendering enables developers to leverage the benefits of server-side rendering (SSR), pre-rendering (also known as "static site generation" or SSG) and client-side rendering (CSR) to optimize your Angular application. This approach provides fine-grained control over rendering strategies for different application parts.

## Setting Up Hybrid Rendering

### New Projects

Create a new project with SSR support:

```bash
ng new --ssr
```

### Existing Projects

Add SSR to an established project:

```bash
ng add @angular/ssr
```

By default, Angular prerenders the entire application and generates a server file. To create a fully static site, set `outputMode` to `static`. To enable full SSR, update server routes to use `RenderMode.Server`.

## Server Routing Configuration

Create route configurations in a file like `app.routes.server.ts`:

```typescript
import {RenderMode, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: '',
    renderMode: RenderMode.Client,
  },
  {
    path: 'about',
    renderMode: RenderMode.Prerender,
  },
  {
    path: 'profile',
    renderMode: RenderMode.Server,
  },
  {
    path: '**',
    renderMode: RenderMode.Server,
  },
];
```

Add this configuration to your application using `provideServerRendering` and `withRoutes`:

```typescript
import {provideServerRendering, withRoutes} from '@angular/ssr';
import {serverRoutes} from './app.routes.server';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(withRoutes(serverRoutes)),
  ],
};
```

## Rendering Modes

| Mode | Description |
|------|-------------|
| **Server (SSR)** | Server renders for each request, sending fully populated HTML |
| **Client (CSR)** | Browser renders the application (Angular default) |
| **Prerender (SSG)** | Build-time rendering generating static HTML files |

### Choosing a Rendering Mode

**Client-Side Rendering (CSR)**
- Simplest development model
- Poorest performance characteristics
- JavaScript must download, parse, and execute before content renders
- May negatively impact SEO
- Minimal server overhead

**Server-Side Rendering (SSR)**
- Faster page loads than CSR
- Excellent SEO
- Requires server-compatible code (no browser-specific APIs)
- Increases server hosting costs
- Eliminates additional network requests during rendering

**Build-Time Prerendering (SSG)**
- Fastest page loads
- Requires all rendering information available at build time
- Cannot include user-specific data
- Excellent SEO
- May significantly increase build times
- Can be deployed via CDN or static file servers
- Minimal per-request server overhead

For the complete guide including server configuration, caching, platform-specific implementations, and more, see the [full SSR guide](./index.md).
