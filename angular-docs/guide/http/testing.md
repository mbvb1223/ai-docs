# HTTP Client Testing
> Source: https://angular.dev/guide/http/testing

Testing HTTP requests requires mocking the backend to simulate server interactions. Angular's `@angular/common/http/testing` library provides tools to capture requests, make assertions, and mock responses without hitting a real network.

The testing pattern follows this sequence: the app executes code and makes requests first, then tests verify those requests occurred, assert their properties, and flush responses.

## Setup for Testing

Configure `TestBed` with two essential providers:

```typescript
TestBed.configureTestingModule({
  providers: [
    provideHttpClient(),
    provideHttpClientTesting(),
  ],
});
const httpTesting = TestBed.inject(HttpTestingController);
```

**Critical:** Provide `provideHttpClient()` *before* `provideHttpClientTesting()`. Reversing this order can break tests, as the testing provider overwrites parts of the HTTP client setup.

## Expecting and Answering Requests

### Basic Pattern

```typescript
TestBed.configureTestingModule({
  providers: [ConfigService, provideHttpClient(), provideHttpClientTesting()],
});

const httpTesting = TestBed.inject(HttpTestingController);
const service = TestBed.inject(ConfigService);
const config$ = service.getConfig<Config>();

// Subscribe to trigger the HTTP request
const configPromise = firstValueFrom(config$);

// Verify the request was made
const req = httpTesting.expectOne('/api/config', 'Request to load configuration');

// Assert request properties
expect(req.request.method).toBe('GET');

// Deliver the mock response
req.flush(DEFAULT_CONFIG);

// Verify the response was processed
expect(await configPromise).toEqual(DEFAULT_CONFIG);

// Assert no additional requests were made
httpTesting.verify();
```

### Alternative Matching Syntax

```typescript
const req = httpTesting.expectOne(
  {
    method: 'GET',
    url: '/api/config',
  },
  'Request to load the configuration',
);
```

**Note:** `expectOne()` fails if multiple requests match the criteria. Query parameters are included in URL matching.

### Verification Pattern

Move verification into `afterEach()` for consistency:

```typescript
afterEach(() => {
  TestBed.inject(HttpTestingController).verify();
});
```

## Handling Multiple Requests

Use `match()` instead of `expectOne()` when responses must handle duplicate requests:

```typescript
const allGetRequests = httpTesting.match({method: 'GET'});
for (const req of allGetRequests) {
  // Handle each request individually
}
```

Unlike `expectOne()`, matched requests are removed from future matching, and you assume responsibility for flushing and verifying them.

## Advanced Matching

### Predicate Functions

Custom matching logic uses predicate functions:

```typescript
// Find requests with a body
const requestsWithBody = httpTesting.expectOne((req) => req.body !== null);
```

### Negative Assertions

Assert that no requests match specific criteria:

```typescript
// Verify no non-GET requests (mutations) were issued
httpTesting.expectNone((req) => req.method !== 'GET');
```

## Error Handling

### Backend Errors

Simulate server responses with non-successful status codes:

```typescript
const req = httpTesting.expectOne('/api/config');
req.flush('Failed!', {status: 500, statusText: 'Internal Server Error'});
// Test your error handling logic
```

### Network Errors

Simulate connection failures using `error()`:

```typescript
const req = httpTesting.expectOne('/api/config');
req.error(new ProgressEvent('network error!'));
// Test your network error handling
```

## Testing Interceptors

### Functional Interceptors

```typescript
export function authInterceptor(
  request: HttpRequest<unknown>,
  next: HttpHandlerFn,
): Observable<HttpEvent<unknown>> {
  const authService = inject(AuthService);
  const clonedRequest = request.clone({
    headers: request.headers.append(
      'X-Authentication-Token',
      authService.getAuthToken(),
    ),
  });
  return next(clonedRequest);
}

TestBed.configureTestingModule({
  providers: [
    AuthService,
    provideHttpClient(withInterceptors([authInterceptor])),
    provideHttpClientTesting(),
  ],
});

const req = httpTesting.expectOne('/api/config');
expect(req.request.headers.get('X-Authentication-Token')).toEqual(
  service.getAuthToken(),
);
```

### Class-Based Interceptors

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  private authService = inject(AuthService);

  intercept(request: HttpRequest<unknown>, next: HttpHandler):
    Observable<HttpEvent<unknown>> {
    const clonedRequest = request.clone({
      headers: request.headers.append(
        'X-Authentication-Token',
        this.authService.getAuthToken(),
      ),
    });
    return next.handle(clonedRequest);
  }
}

TestBed.configureTestingModule({
  providers: [
    AuthService,
    provideHttpClient(withInterceptorsFromDi()),
    provideHttpClientTesting(),
    {provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true},
  ],
});
```

Both approaches allow inspection of modified requests through `HttpTestingController` to verify interceptor functionality.
