# Reactive Data Fetching with httpResource
> Source: https://angular.dev/guide/http/http-resource

`httpResource` serves as a reactive wrapper around `HttpClient`, exposing request status and response data as signals. This enables integration with Angular's reactive APIs like `computed`, `effect`, and `linkedSignal`. As a wrapper, it inherits all `HttpClient` features including interceptors.

**Status**: `httpResource` is experimental and subject to change before stabilization.

## Basic Usage

### Simple URL-Based Requests

Define an HTTP resource by providing a reactive function that returns a URL:

```typescript
userId = input.required<string>();
user = httpResource(() => `/api/user/${userId()}`);
```

The resource automatically triggers new requests whenever dependent signals change. Pending requests are cancelled when dependencies update.

**Key Distinction**: Unlike `HttpClient`, which initiates requests upon Observable subscription, `httpResource` executes requests eagerly.

### Advanced Request Configuration

For complex scenarios, define a request object with reactive properties:

```typescript
user = httpResource(() => ({
  url: `/api/user/${userId()}`,
  method: 'GET',
  headers: { 'X-Special': 'true' },
  params: { 'fast': 'yes' },
  reportProgress: true,
  transferCache: true,
  keepalive: true,
  mode: 'cors',
  redirect: 'error',
  priority: 'high',
  cache: 'force-cache',
  credentials: 'include',
  referrer: 'no-referrer',
  integrity: 'sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GhEXAMPLEKEY=',
  referrerPolicy: 'no-referrer'
}));
```

**Recommendation**: Use `httpResource` for read operations only; prefer direct `HttpClient` APIs for mutations (`POST`, `PUT`).

## Template Integration

Control conditional rendering using resource signals:

```typescript
@if(user.hasValue()) {
  <user-details [user]="user.value()">
} @else if (user.error()) {
  <div>Could not load user information</div>
} @else if (user.isLoading()) {
  <div>Loading user info...</div>
}
```

**Important**: Always guard `value()` access with `hasValue()` to prevent runtime errors when the resource is in an error state.

## Response Types

By default, `httpResource` parses responses as JSON. Alternative response types are available:

```typescript
httpResource.text(() => ({ ... }));        // Returns string
httpResource.blob(() => ({ ... }));        // Returns Blob
httpResource.arrayBuffer(() => ({ ... })); // Returns ArrayBuffer
```

## Response Parsing and Validation

Integrate schema validation libraries by specifying a `parse` option. The parse function's return type determines the resource's `value` type.

### Zod Integration Example

```typescript
const starWarsPersonSchema = z.object({
  name: z.string(),
  height: z.number({coerce: true}),
  edited: z.string().datetime(),
  films: z.array(z.string())
});

export class CharacterViewer {
  id = signal(1);
  swPersonResource = httpResource(
    () => `https://swapi.info/api/people/${this.id()}`,
    { parse: starWarsPersonSchema.parse }
  );
}
```

## Testing

`httpResource` testing uses identical APIs to `HttpClient` testing. Ensure `provideHttpClient()` and `provideHttpClientTesting()` are included in test configuration.

### Unit Test Example

```typescript
TestBed.configureTestingModule({
  providers: [provideHttpClient(), provideHttpClientTesting()]
});

const id = signal(0);
const mockBackend = TestBed.inject(HttpTestingController);
const response = httpResource(() => `/data/${id()}`, {
  injector: TestBed.inject(Injector)
});

TestBed.tick();
const firstRequest = mockBackend.expectOne('/data/0');
firstRequest.flush(0);

await TestBed.inject(ApplicationRef).whenStable();
expect(response.value()).toEqual(0);
```

## Setup Requirements

Ensure `provideHttpClient` is included in application providers. Refer to the [HttpClient setup guide](setup.md) for configuration details.
