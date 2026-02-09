# HttpClient Interceptors
> Source: https://angular.dev/guide/http/interceptors

Angular's `HttpClient` supports interceptors, a middleware pattern enabling common request/response patterns to be abstracted away from individual requests.

Angular offers two interceptor types: **functional** (recommended) and **DI-based**. Functional interceptors provide more predictable behavior in complex setups.

## What Interceptors Do

Interceptors enable patterns like:

- Adding authentication headers to API requests
- Implementing exponential backoff retry logic
- Caching responses with time or mutation-based invalidation
- Customizing response parsing
- Logging server response metrics
- Displaying loading spinners during network operations
- Batching requests within timeframes
- Auto-failing requests after configurable timeouts
- Server polling and result refreshing

## Defining Functional Interceptors

Basic interceptor structure accepts an `HttpRequest` and `next` handler representing the processing chain:

```typescript
export function loggingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  console.log(req.url);
  return next(req);
}
```

## Configuration

Register interceptors via `withInterceptors` during `HttpClient` setup:

```typescript
bootstrapApplication(App, {
  providers: [provideHttpClient(withInterceptors([loggingInterceptor, cachingInterceptor]))],
});
```

Interceptors execute in declaration order, forming a processing chain.

## Intercepting Response Events

Transform the `Observable` stream to access response data. Check `.type` to identify final responses:

```typescript
export function loggingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response) {
        console.log(req.url, 'returned a response with status', event.status);
      }
    }),
  );
}
```

Interceptors naturally associate responses with their outgoing requests, because they transform the response stream in a closure that captures the request object.

## Modifying Requests

`HttpRequest` and `HttpResponse` instances are immutable. Modifications use `.clone()` with property specifications:

```typescript
const reqWithHeader = req.clone({
  headers: req.headers.set('X-New-Header', 'new header value'),
});
```

This immutability ensures interceptor idempotence across retries.

**Critical:** Request/response bodies are not protected from deep mutations. Interceptors handling bodies should account for multiple executions.

## Dependency Injection in Interceptors

Interceptors run in their registering injector's context and can use Angular's `inject()` API:

```typescript
export function authInterceptor(req: HttpRequest<unknown>, next: HttpHandlerFn) {
  const authToken = inject(AuthService).getAuthToken();
  const newReq = req.clone({
    headers: req.headers.append('X-Authentication-Token', authToken),
  });
  return next(newReq);
}
```

## Request and Response Metadata

`HttpRequest` includes a `.context` object storing metadata as a typed map using `HttpContextToken` keys.

### Defining Context Tokens

```typescript
export const CACHING_ENABLED = new HttpContextToken<boolean>(() => true);
```

### Reading Tokens in Interceptors

```typescript
export function cachingInterceptor(req: HttpRequest<unknown>, next: HttpHandlerFn): Observable<HttpEvent<unknown>> {
  if (req.context.get(CACHING_ENABLED)) {
    // apply caching logic
    return ...;
  } else {
    return next(req);
  }
}
```

### Setting Context Tokens

```typescript
const data$ = http.get('/sensitive/data', {
  context: new HttpContext().set(CACHING_ENABLED, false),
});
```

The request context is **mutable** -- state persists across retries, enabling cross-retry communication.

## Synthetic Responses

Interceptors can construct responses without invoking `next`, useful for caching or alternate mechanisms:

```typescript
const resp = new HttpResponse({
  body: 'response body',
});
```

## Working with Redirect Information

When using `withFetch`, responses include a `redirected` property indicating redirect occurrence:

```typescript
export function redirectTrackingInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    tap((event) => {
      if (event.type === HttpEventType.Response && event.redirected) {
        console.log('Request to', req.url, 'was redirected to', event.url);
      }
    }),
  );
}
```

## Working with Response Types

The `type` property indicates browser response handling per CORS policies:

- `'basic'` -- Same-origin with all headers accessible
- `'cors'` -- Cross-origin with proper CORS configuration
- `'opaque'` -- Cross-origin without CORS; limited header/body access
- `'opaqueredirect'` -- Redirected request in no-cors mode
- `'error'` -- Network failure

```typescript
export function responseTypeInterceptor(
  req: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  return next(req).pipe(
    map((event) => {
      if (event.type === HttpEventType.Response) {
        switch (event.responseType) {
          case 'opaque':
            console.warn('Limited response data due to CORS policy');
            break;
          case 'cors':
          case 'basic':
            // Full access to response data
            break;
          case 'error':
            console.error('Network error in response');
            break;
        }
      }
    }),
  );
}
```

## DI-Based Interceptors

Angular also supports injectable class interceptors implementing `HttpInterceptor`:

```typescript
@Injectable()
export class LoggingInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, handler: HttpHandler): Observable<HttpEvent<any>> {
    console.log('Request URL: ' + req.url);
    return handler.handle(req);
  }
}
```

Register via dependency injection multi-provider:

```typescript
bootstrapApplication(App, {
  providers: [
    provideHttpClient(withInterceptorsFromDi()),
    {provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true},
  ],
});
```

DI-based interceptors execute in registration order, which can be difficult to predict in hierarchical configurations.
