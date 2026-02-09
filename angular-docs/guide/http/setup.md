# Setting up HttpClient
> Source: https://angular.dev/guide/http/setup

Before using `HttpClient` in an Angular application, you must configure it through dependency injection.

## Providing HttpClient Through Dependency Injection

### Standalone Configuration

`HttpClient` is provided via the `provideHttpClient` helper function, typically included in the application providers within `app.config.ts`:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient()],
};
```

### NgModule-Based Configuration

For applications using NgModule-based bootstrap, include `provideHttpClient` in your app's NgModule providers:

```typescript
@NgModule({
  providers: [provideHttpClient()],
  // ... other application configuration
})
export class AppModule {}
```

### Injecting HttpClient

Once configured, inject `HttpClient` as a dependency in components, services, or other classes:

```typescript
@Injectable({providedIn: 'root'})
export class ConfigService {
  private http = inject(HttpClient);
  // This service can now make HTTP requests via `this.http`.
}
```

## Configuring Features of HttpClient

`provideHttpClient` accepts optional feature configurations to enable or customize client behavior.

### withFetch

```typescript
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(withFetch())],
};
```

By default, `HttpClient` uses the `XMLHttpRequest` API. The `withFetch` feature switches to the modern `fetch` API instead. While more modern and available in some environments where `XMLHttpRequest` isn't supported, `fetch` has limitations such as not producing upload progress events.

### withInterceptors(...)

Configures functional interceptor functions to process `HttpClient` requests. See the [interceptor guide](interceptors.md) for detailed information.

### withInterceptorsFromDi()

Includes legacy class-based interceptors in the `HttpClient` configuration. Functional interceptors (through `withInterceptors`) offer more predictable ordering and are recommended over DI-based interceptors.

### withRequestsMadeViaParent()

By default, `provideHttpClient` overrides any parent injector's `HttpClient` configuration. Adding this feature makes `HttpClient` pass requests to the parent injector's instance after passing through local interceptors. This is useful for adding interceptors in child injectors while still using parent interceptor chains.

**Critical:** You must configure an `HttpClient` instance in a parent injector, or a runtime error will occur.

### withJsonpSupport()

Enables the `.jsonp()` method on `HttpClient` for GET requests via the JSONP convention for cross-domain data loading. CORS is preferred when possible.

### withXsrfConfiguration(...)

Allows customization of `HttpClient`'s built-in XSRF security functionality. Refer to the security guide for details.

### withNoXsrfProtection()

Disables `HttpClient`'s built-in XSRF security functionality. See the security guide for information.

## HttpClientModule-Based Configuration

Legacy NgModule-based applications can use these modules from `@angular/common/http`:

| NgModule | provideHttpClient() Equivalent |
|----------|--------------------------------|
| `HttpClientModule` | `provideHttpClient(withInterceptorsFromDi())` |
| `HttpClientJsonpModule` | `withJsonpSupport()` |
| `HttpClientXsrfModule.withOptions(...)` | `withXsrfConfiguration(...)` |
| `HttpClientXsrfModule.disable()` | `withNoXsrfProtection()` |

### Caution with Multiple Injectors

When `HttpClientModule` appears in multiple injectors, interceptor behavior becomes poorly defined and depends on configuration ordering. Prefer `provideHttpClient` for multi-injector setups, which offers more stable behavior through the `withRequestsMadeViaParent` feature.
