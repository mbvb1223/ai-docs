# Server-Side and Hybrid Rendering in Angular
> Source: https://angular.dev/guide/ssr

## Overview

Angular applications default to client-side rendering (CSR), which delivers a lightweight initial payload but results in slower load times and higher device resource demands. Hybrid rendering strategies that combine server-side rendering (SSR), pre-rendering (static site generation), and CSR offer significant performance improvements.

## What is Hybrid Rendering?

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

### Configuring Routes

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

For the App Shell pattern, use `withAppShell`:

```typescript
import {provideServerRendering, withRoutes, withAppShell} from '@angular/ssr';
import {AppShell} from './app-shell';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering(withRoutes(serverRoutes), withAppShell(AppShell)),
  ],
};
```

### Rendering Modes

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

> **NOTE:** With Angular Service Worker, the first request is server-rendered; subsequent requests are handled client-side.

## Setting Headers and Status Codes

Configure custom headers and status codes for server routes:

```typescript
import {RenderMode, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'profile',
    renderMode: RenderMode.Server,
    headers: {
      'X-My-Custom-Header': 'some-value',
    },
    status: 201,
  },
];
```

## Redirects

- **SSR:** Redirects use standard HTTP redirects (301, 302)
- **SSG:** Redirects use soft redirects with `<meta http-equiv="refresh">` tags

## Customizing Build-Time Prerendering

### Parameterized Routes

Specify which parameters generate separate prerendered documents using `getPrerenderParams`:

```typescript
import {RenderMode, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'post/:id',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      const dataService = inject(PostService);
      const ids = await dataService.getIds();
      return ids.map((id) => ({id}));
    },
  },
  {
    path: 'post/:id/**',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      return [
        {id: '1', '**': 'foo/3'},
        {id: '2', '**': 'bar/4'},
      ];
    },
  },
];
```

> **IMPORTANT:** When using `inject` inside `getPrerenderParams`, it must be invoked synchronously -- not within asynchronous callbacks or following `await` statements. Use `runInInjectionContext` if needed.

### Fallback Strategies

Specify fallback behavior for unprerendered paths:

```typescript
import {RenderMode, PrerenderFallback, ServerRoute} from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'post/:id',
    renderMode: RenderMode.Prerender,
    fallback: PrerenderFallback.Client,
    async getPrerenderParams() {
      return [{id: 1}, {id: 2}, {id: 3}];
    },
  },
];
```

Available fallback strategies: **Server** (default), **Client**, or **None**.

## Server-Compatible Components

Browser-specific APIs like `window`, `document`, `navigator`, and `location` are unavailable on the server. Use lifecycle hooks to restrict browser-only code:

```typescript
import {Component, viewChild, afterNextRender} from '@angular/core';

@Component({
  selector: 'my-cmp',
  template: `<span #content>{{ ... }}</span>`,
})
export class MyComponent {
  contentRef = viewChild.required<ElementRef>('content');

  constructor() {
    afterNextRender(() => {
      console.log('content height: ' + this.contentRef().nativeElement.scrollHeight);
    });
  }
}
```

> **NOTE:** Prefer platform-specific providers over runtime checks with `isPlatformBrowser` or `isPlatformServer`.

> **IMPORTANT:** Avoid using `isPlatformBrowser` in templates to render different content across platforms -- this causes hydration mismatches and layout shifts. Instead, use `afterNextRender` for browser-specific initialization and maintain consistent rendered content.

## Setting Providers on the Server

Server-side provider values persist across requests until the server restarts. Use factory providers with `useFactory` to generate new values for each request:

```typescript
// Provider that creates a new value per request
{provide: SomeToken, useFactory: () => new SomeValue()}
```

## Providing Platform-Specific Implementations

Create separate service implementations for browser and server:

```typescript
export abstract class AnalyticsService {
  abstract trackEvent(name: string): void;
}

@Injectable()
export class BrowserAnalyticsService implements AnalyticsService {
  trackEvent(name: string): void {
    // Browser-based analytics
  }
}

@Injectable()
export class ServerAnalyticsService implements AnalyticsService {
  trackEvent(name: string): void {
    // Server-side tracking
  }
}
```

Register in application configuration (`app.config.ts`):

```typescript
export const appConfig: ApplicationConfig = {
  providers: [{provide: AnalyticsService, useClass: BrowserAnalyticsService}],
};
```

Override in server configuration (`app.config.server.ts`):

```typescript
const serverConfig: ApplicationConfig = {
  providers: [{provide: AnalyticsService, useClass: ServerAnalyticsService}],
};
```

## Accessing Document via Dependency Injection

Avoid directly referencing `document`. Use the `DOCUMENT` token instead:

```typescript
import {Injectable, inject, DOCUMENT} from '@angular/core';

@Injectable({providedIn: 'root'})
export class CanonicalLinkService {
  private readonly document = inject(DOCUMENT);

  setCanonical(href: string): void {
    const link = this.document.createElement('link');
    link.rel = 'canonical';
    link.href = href;
    this.document.head.appendChild(link);
  }
}
```

For meta tag management, use the `Meta` service.

## Accessing Request and Response via Dependency Injection

Key tokens for SSR environments:

- **`REQUEST`**: Access to current request object (Web API `Request` type)
- **`RESPONSE_INIT`**: Response initialization options for setting headers and status codes
- **`REQUEST_CONTEXT`**: Additional context passed as second parameter to `handle` function

```typescript
import {inject, REQUEST} from '@angular/core';

@Component({
  selector: 'app-my-component',
  template: `<h1>My Component</h1>`,
})
export class MyComponent {
  constructor() {
    const request = inject(REQUEST);
    console.log(request?.url);
  }
}
```

> **IMPORTANT:** These tokens are `null` during builds, browser rendering (CSR), static site generation (SSG), and route extraction in development.

## Generate a Fully Static Application

Set `outputMode` to `static` in `angular.json` to create a fully static site without requiring a Node.js server:

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "options": {
            "outputMode": "static"
          }
        }
      }
    }
  }
}
```

## Caching Data with HttpClient

`HttpClient` automatically caches outgoing network requests on the server and transfers this data to the browser for reuse during hydration. Caching stops once the application becomes stable in the browser.

### Configuring Caching Options

Customize caching behavior with `withHttpTransferCacheOptions`:

```typescript
import {bootstrapApplication} from '@angular/platform-browser';
import {provideClientHydration, withHttpTransferCacheOptions} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [
    provideClientHydration(
      withHttpTransferCacheOptions({
        includeHeaders: ['ETag', 'Cache-Control'],
        filter: (req) => !req.url.includes('/api/profile'),
        includePostRequests: true,
        includeRequestsWithAuthHeaders: false,
      }),
    ),
  ],
});
```

### Configuration Options

- **`includeHeaders`**: Specifies which response headers to cache (none by default). Avoid sensitive authentication tokens.
- **`includePostRequests`**: Enables caching for `POST` requests (default: false). Use only with idempotent read operations like GraphQL queries.
- **`includeRequestsWithAuthHeaders`**: Determines whether requests with `Authorization` or `Proxy-Authorization` headers are cached (default: false to prevent caching user-specific responses).

### Per-Request Overrides

Override caching for specific requests:

```typescript
http.get('/api/profile', {transferCache: {includeHeaders: ['CustomHeader']}});
```

### Disabling Caching

**Globally** using `withNoHttpTransferCache`:

```typescript
import {bootstrapApplication, provideClientHydration, withNoHttpTransferCache} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withNoHttpTransferCache())],
});
```

**Using filter** to selectively disable for specific endpoints:

```typescript
withHttpTransferCacheOptions({
  filter: (req) => !req.url.includes('/api/sensitive-data'),
})
```

**Individually** on requests:

```typescript
httpClient.get('/api/sensitive-data', {transferCache: false});
```

> **NOTE:** Use `HTTP_TRANSFER_CACHE_ORIGIN_MAP` token when your application uses different HTTP origins on server and client to establish origin mapping for proper cache reuse.

## Configuring a Server

### Node.js

Use `@angular/ssr/node` for Node.js environments:

```typescript
import {
  AngularNodeAppEngine,
  createNodeRequestHandler,
  writeResponseToNodeResponse,
} from '@angular/ssr/node';
import express from 'express';

const app = express();
const angularApp = new AngularNodeAppEngine();

app.use('*', (req, res, next) => {
  angularApp
    .handle(req)
    .then((response) => {
      if (response) {
        writeResponseToNodeResponse(response, res);
      } else {
        next();
      }
    })
    .catch(next);
});

export const reqHandler = createNodeRequestHandler(app);
```

### Non-Node.js Platforms

Use `@angular/ssr` with Web API standards:

```typescript
import {AngularAppEngine, createRequestHandler} from '@angular/ssr';

const angularApp = new AngularAppEngine();

export const reqHandler = createRequestHandler(async (req: Request) => {
  const res: Response | null = await angularApp.render(req);
  // ...
});
```
