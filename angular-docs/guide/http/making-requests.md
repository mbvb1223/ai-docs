# Making HTTP Requests
> Source: https://angular.dev/guide/http/making-requests

Angular's `HttpClient` provides methods for all HTTP verbs to load data and apply server mutations. Each method returns an RxJS `Observable` that executes the request upon subscription.

**Key Point:** Observables created by `HttpClient` may be subscribed any number of times and will make a new backend request for each subscription.

## Fetching JSON Data

The `HttpClient.get()` method retrieves data from a backend endpoint:

```typescript
http.get<Config>('/api/config').subscribe((config) => {
  // process the configuration.
});
```

The generic type parameter specifies the expected response data structure. This is a type assertion only -- `HttpClient` does not validate actual server responses match this type.

## Non-JSON Response Types

Control response format using the `responseType` option:

| Value | Returns |
|-------|---------|
| `'json'` (default) | Typed JSON data |
| `'text'` | String data |
| `'arraybuffer'` | ArrayBuffer with raw bytes |
| `'blob'` | Blob instance |

Example retrieving image bytes:

```typescript
http.get('/images/dog.jpg', {responseType: 'arraybuffer'})
  .subscribe((buffer) => {
    console.log('The image is ' + buffer.byteLength + ' bytes large');
  });
```

**Important:** The `responseType` value must use literal typing (not string variables) to affect return types correctly.

## Mutating Server State

The `HttpClient.post()` method sends request bodies for mutations:

```typescript
http.post<Config>('/api/config', newConfig).subscribe((config) => {
  console.log('Updated config:', config);
});
```

Request body serialization depends on data type:

| Body Type | Serialization |
|-----------|---------------|
| string | Plain text |
| number, boolean, array, object | JSON |
| ArrayBuffer | Raw buffer data |
| Blob | Raw data with content type |
| FormData | multipart/form-data |
| HttpParams / URLSearchParams | application/x-www-form-urlencoded |

**Critical:** Mutation requests must be subscribed to actually execute.

## Setting URL Parameters

Use the `params` option for query strings. Object literals provide simplicity:

```typescript
http.get('/api/config', {
  params: {filter: 'all'},
}).subscribe((config) => {
  // ...
});
```

For complex parameter handling, use `HttpParams`:

```typescript
const baseParams = new HttpParams().set('filter', 'all');
http.get('/api/config', {
  params: baseParams.set('details', 'enabled'),
}).subscribe((config) => {
  // ...
});
```

`HttpParams` instances are immutable -- mutation methods return new instances with changes applied.

### Custom Parameter Encoding

Implement `HttpParameterCodec` for custom encoding logic:

```typescript
export class CustomHttpParamEncoder implements HttpParameterCodec {
  encodeKey(key: string): string {
    return encodeURIComponent(key);
  }
  encodeValue(value: string): string {
    return encodeURIComponent(value);
  }
  decodeKey(key: string): string {
    return decodeURIComponent(key);
  }
  decodeValue(value: string): string {
    return decodeURIComponent(value);
  }
}

const params = new HttpParams({ encoder: new CustomHttpParamEncoder() })
  .set('email', 'dev+alerts@example.com');
```

## Setting Request Headers

Add headers via the `headers` option:

```typescript
http.get('/api/config', {
  headers: {
    'X-Debug-Level': 'verbose',
  },
}).subscribe((config) => {
  // ...
});
```

Using `HttpHeaders` for advanced control:

```typescript
const baseHeaders = new HttpHeaders().set('X-Debug-Level', 'minimal');
http.get<Config>('/api/config', {
  headers: baseHeaders.set('X-Debug-Level', 'verbose'),
}).subscribe((config) => {
  // ...
});
```

Like `HttpParams`, `HttpHeaders` instances are immutable.

## Response Event Observation

Access the full response object (not just body) with `observe: 'response'`:

```typescript
http.get<Config>('/api/config', {observe: 'response'})
  .subscribe((res) => {
    console.log('Response status:', res.status);
    console.log('Body:', res.body);
  });
```

## Raw Progress Events

Enable progress tracking with `reportProgress: true` and `observe: 'events'`:

```typescript
http.post('/api/upload', myData, {
  reportProgress: true,
  observe: 'events',
}).subscribe((event) => {
  switch (event.type) {
    case HttpEventType.UploadProgress:
      console.log('Uploaded ' + event.loaded + ' out of ' + event.total);
      break;
    case HttpEventType.Response:
      console.log('Finished uploading!');
      break;
  }
});
```

**Note:** The fetch backend does not report upload progress.

Event types include: `Sent`, `UploadProgress`, `ResponseHeader`, `DownloadProgress`, `Response`, and `User`.

## Error Handling

`HttpClient` captures all error types in `HttpErrorResponse`. Network/timeout errors have status `0`; backend errors have the server's status code.

Use RxJS operators for error management:

```typescript
http.get('/api/config', { timeout: 3000 }).subscribe({
  next: (config) => {
    console.log('Config fetched successfully:', config);
  },
  error: (err) => {
    // Handle timeout or other errors
  },
});
```

The `catchError` operator transforms errors into UI values; `retry()` auto-resubscribes under conditions.

### Timeouts

Set request timeouts (in milliseconds) with the `timeout` option:

```typescript
http.get('/api/config', { timeout: 3000 }).subscribe({
  next: (config) => console.log(config),
  error: (err) => { /* Handle timeout */ },
});
```

Timeouts apply only to the HTTP request, not the entire handling chain.

## Advanced Fetch Options

When using `withFetch()`, additional options optimize performance and user experience.

### Keep-alive Connections

Allow requests to outlive the initiating page:

```typescript
http.post('/api/analytics', analyticsData, {
  keepalive: true,
}).subscribe();
```

### HTTP Caching Control

The `cache` option manages browser cache interaction:

```typescript
// Use cached response regardless of freshness
http.get('/api/config', { cache: 'force-cache' }).subscribe();

// Always fetch from network, bypass cache
http.get('/api/live-data', { cache: 'no-cache' }).subscribe();

// Use cached response only
http.get('/api/static-data', { cache: 'only-if-cached' }).subscribe();
```

### Request Priority

Indicate importance for Core Web Vitals optimization:

```typescript
http.get('/api/user-profile', { priority: 'high' }).subscribe();
http.get('/api/recommendations', { priority: 'low' }).subscribe();
http.get('/api/settings', { priority: 'auto' }).subscribe();
```

Priority values: `'high'` (critical), `'low'` (non-critical), `'auto'` (browser-determined).

### Request Mode

Control cross-origin request handling:

```typescript
http.get('/api/local-data', { mode: 'same-origin' }).subscribe();
http.get('https://api.external.com/data', { mode: 'cors' }).subscribe();
http.get('https://external-api.com/data', { mode: 'no-cors' }).subscribe();
```

Modes: `'same-origin'`, `'cors'` (default), `'no-cors'`.

### Redirect Handling

Specify redirect behavior:

```typescript
http.get('/api/resource', { redirect: 'follow' }).subscribe();
http.get('/api/resource', { redirect: 'manual' }).subscribe();
http.get('/api/resource', { redirect: 'error' }).subscribe();
```

Options: `'follow'` (default), `'manual'`, `'error'`.

### Credentials Handling

Control credential inclusion in cross-origin requests:

```typescript
http.get('https://api.example.com/data', {
  credentials: 'include'
}).subscribe();

http.get('https://api.example.com/data', {
  credentials: 'omit'
}).subscribe();

http.get('/api/user-data', {
  credentials: 'same-origin'
}).subscribe();
```

**Important:** `withCredentials: true` overrides the `credentials` option, always forcing `'include'`.

Values: `'omit'`, `'same-origin'` (default), `'include'`.

### Referrer and Referrer Policy

Control referrer information sent with requests:

```typescript
http.get('/api/data', {
  referrer: 'https://example.com/page'
}).subscribe();

http.get('/api/analytics', {
  referrerPolicy: 'origin'
}).subscribe();

http.get('/api/data', {
  referrerPolicy: 'no-referrer'
}).subscribe();
```

Referrer policy options: `'no-referrer'`, `'no-referrer-when-downgrade'`, `'origin'`, `'origin-when-cross-origin'`, `'same-origin'`, `'strict-origin'`, `'strict-origin-when-cross-origin'`, `'unsafe-url'`.

### Integrity Verification

Verify response content using cryptographic hashes:

```typescript
http.get('/api/script.js', {
  integrity: 'sha256-ABC123...',
  responseType: 'text',
}).subscribe((script) => {
  // Script content is verified
});
```

## Understanding HTTP Observables

`HttpClient` produces "cold" Observables -- no request executes until subscription. Each subscription triggers a new backend request. Multiple subscriptions to the same Observable create independent requests.

Unsubscribing cancels in-progress requests. When used with async pipes or `switchMap`, this automatically cleans up stale requests.

Responses typically complete automatically after returning. Memory leaks are generally not a concern, though subscriptions should be cleaned up when components are destroyed to prevent error callbacks on destroyed components.

**Recommendation:** Use the `async` pipe or `toSignal` to ensure proper subscription disposal.

## Best Practices

Create reusable, injectable services encapsulating data access logic rather than injecting `HttpClient` directly into components:

```typescript
@Injectable({providedIn: 'root'})
export class UserService {
  private http = inject(HttpClient);
  getUser(id: string): Observable<User> {
    return this.http.get<User>(`/api/user/${id}`);
  }
}
```

Combine `@if` with the async pipe in templates to render UI only after data loads:

```typescript
@Component({
  imports: [AsyncPipe],
  template: `
    @if (user$ | async; as user) {
      <p>Name: {{ user.name }}</p>
    }
  `,
})
export class UserProfile {
  userId = input.required<string>();
  user$!: Observable<User>;
  private userService = inject(UserService);

  constructor() {
    effect(() => {
      this.user$ = this.userService.getUser(this.userId());
    });
  }
}
```
