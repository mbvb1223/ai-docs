# Defining Dependency Providers
> Source: https://angular.dev/guide/di/dependency-injection-providers (redirects to https://angular.dev/guide/di/defining-dependency-providers)

Angular provides two primary approaches for making services available for injection:

1. **Automatic provision** -- Using `providedIn` in the `@Injectable` decorator or factory functions in `InjectionToken` configuration
2. **Manual provision** -- Using the `providers` array in components, directives, routes, or application configuration

## Automatic Provision for Non-Class Dependencies

### Understanding InjectionToken

An `InjectionToken` serves as a unique identifier for Angular's DI system, enabling storage and retrieval of any value type -- not just classes. The string parameter is purely descriptive for debugging; Angular identifies tokens by object reference.

```typescript
import {InjectionToken} from '@angular/core';

export const API_URL = new InjectionToken<string>('api.url');
export const LOGGER = new InjectionToken<(msg: string) => void>('logger.function');

export interface Config {
  apiUrl: string;
  timeout: number;
}
export const CONFIG_TOKEN = new InjectionToken<Config>('app.config');
```

### InjectionToken with Factory Functions

Tokens with factory functions default to `providedIn: 'root'` and enable global availability without manual provider registration:

```typescript
export interface AppConfig {
  apiUrl: string;
  version: string;
  features: Record<string, boolean>;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config', {
  providedIn: 'root',
  factory: () => ({
    apiUrl: 'https://api.example.com',
    version: '1.0.0',
    features: {
      darkMode: true,
      analytics: false,
    },
  }),
});

@Component({
  selector: 'app-header',
  template: `<h1>Version: {{ config.version }}</h1>`,
})
export class Header {
  config = inject(APP_CONFIG);
}
```

### Practical Factory Patterns

Factory functions excel for logging with dependencies, browser APIs, and feature flags:

```typescript
export type LoggerFn = (level: string, message: string) => void;

export const LOGGER_FN = new InjectionToken<LoggerFn>('logger.function', {
  providedIn: 'root',
  factory: () => {
    const config = inject(APP_CONFIG);
    return (level: string, message: string) => {
      if (config.features.logging !== false) {
        console[level](`[${new Date().toISOString()}] ${message}`);
      }
    };
  },
});

export const LOCAL_STORAGE = new InjectionToken<Storage>('localStorage', {
  providedIn: 'root',
  factory: () => window.localStorage,
});

export const FEATURE_FLAGS = new InjectionToken<Map<string, boolean>>('feature.flags', {
  providedIn: 'root',
  factory: () => {
    const flags = new Map<string, boolean>();
    const urlParams = new URLSearchParams(window.location.search);
    flags.set('betaFeatures', urlParams.get('beta') === 'true');
    return flags;
  },
});
```

**Advantages:**

- No manual provider configuration needed
- Tree-shakeable and included only when used
- Full TypeScript type safety
- Can inject dependencies via `inject()`

## Manual Provider Configuration

Manual configuration becomes necessary when:

- Services lack `providedIn` configuration
- Component-specific instances are required
- Runtime configuration is needed
- Non-class values must be provided

### Services Without providedIn

```typescript
@Injectable()
export class LocalDataStore {
  private data: string[] = [];
  addData(item: string) {
    this.data.push(item);
  }
}

@Component({
  selector: 'app-example',
  providers: [LocalDataStore],
  template: `...`,
})
export class Example {
  dataStore = inject(LocalDataStore);
}
```

### Component-Specific Instances

Services with `providedIn: 'root'` can be overridden at the component level, creating isolated instances tied to component lifecycle:

```typescript
@Injectable({providedIn: 'root'})
export class DataStore {
  private data: ListItem[] = [];
}

@Component({
  selector: 'app-isolated',
  providers: [DataStore],
  template: `...`,
})
export class Isolated {
  dataStore = inject(DataStore); // Component-specific instance
}
```

## Injector Hierarchy

Angular's DI system mirrors the component tree. When requesting a dependency, Angular traverses upward through the injector hierarchy until finding a provider. This enables:

- **Scoped instances** -- Different application areas have separate service instances
- **Provider override** -- Child components override parent providers
- **Memory efficiency** -- Services instantiate only where needed

Any element with a component or directive can provide values to descendants.

## Provider Declaration

Provider configuration objects define key-value pairs:

- **Key** -- Unique identifier (provider token)
- **Value** -- The dependency returned when requested

### Shorthand vs. Full Syntax

```typescript
// Shorthand
providers: [LocalService]

// Full syntax
providers: [
  { provide: LocalService, useClass: LocalService }
]
```

### Provider Identifiers

#### Class Names

```typescript
providers: [{provide: LocalService, useClass: LocalService}]
```

#### Injection Tokens

```typescript
export const DATA_SERVICE_TOKEN = new InjectionToken<DataService>('DataService');

@Component({
  selector: 'app-example',
  providers: [{provide: DATA_SERVICE_TOKEN, useClass: LocalDataService}],
})
export class Example {
  private dataService = inject(DATA_SERVICE_TOKEN);
}
```

**Important:** TypeScript interfaces cannot serve as DI identifiers because they are compile-time constructs. Use `InjectionToken` instead.

## Provider Value Types

### useClass

Provides a JavaScript class, supporting multiple implementations:

```typescript
@Injectable()
export class Logger {
  log(message: string) {
    console.log(message);
  }
}

@Injectable()
export class BetterLogger extends Logger {
  override log(message: string) {
    super.log(`[${new Date().toISOString()}] ${message}`);
  }
}

@Component({
  selector: 'app-example',
  providers: [
    {provide: Logger, useClass: BetterLogger},
  ],
})
export class Example {
  private logger = inject(Logger);
}
```

### useValue

Provides static JavaScript values:

```typescript
providers: [
  {provide: API_URL_TOKEN, useValue: 'https://api.example.com'},
  {provide: MAX_RETRIES_TOKEN, useValue: 3},
  {provide: FEATURE_FLAGS_TOKEN, useValue: {darkMode: true, beta: false}},
];
```

**Application Configuration Example:**

```typescript
export interface AppConfig {
  apiUrl: string;
  appTitle: string;
  features: {
    darkMode: boolean;
    analytics: boolean;
  };
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

const appConfig: AppConfig = {
  apiUrl: 'https://api.example.com',
  appTitle: 'My Application',
  features: {
    darkMode: true,
    analytics: false,
  },
};

bootstrapApplication(AppComponent, {
  providers: [{provide: APP_CONFIG, useValue: appConfig}],
});

@Component({
  selector: 'app-header',
  template: `<h1>{{ title }}</h1>`,
})
export class Header {
  private config = inject(APP_CONFIG);
  title = this.config.appTitle;
}
```

### useFactory

Factory functions generate values with dependency injection support:

```typescript
export const loggerFactory = (config: AppConfig) => {
  return new LoggerService(config.logLevel, config.endpoint);
};

providers: [
  {
    provide: LoggerService,
    useFactory: loggerFactory,
    deps: [APP_CONFIG],
  },
];
```

**Optional Dependencies:**

```typescript
import {Optional} from '@angular/core';

providers: [
  {
    provide: MyService,
    useFactory: (required: RequiredService, optional?: OptionalService) => {
      return new MyService(required, optional || new DefaultService());
    },
    deps: [RequiredService, [new Optional(), OptionalService]],
  },
];
```

**Configuration-Based API Client:**

```typescript
class ApiClient {
  constructor(
    private http: HttpClient,
    private baseUrl: string,
    private rateLimitMs: number,
  ) {}

  async fetchData(endpoint: string) {
    await this.applyRateLimit();
    return this.http.get(`${this.baseUrl}/${endpoint}`);
  }

  private async applyRateLimit() {
    return new Promise((resolve) => setTimeout(resolve, this.rateLimitMs));
  }
}

const apiClientFactory = () => {
  const http = inject(HttpClient);
  const userService = inject(UserService);
  const baseUrl = userService.getApiBaseUrl();
  const rateLimitMs = userService.getRateLimit();
  return new ApiClient(http, baseUrl, rateLimitMs);
};

export const apiClientProvider = {
  provide: ApiClient,
  useFactory: apiClientFactory,
};

@Component({
  selector: 'app-dashboard',
  providers: [apiClientProvider],
})
export class Dashboard {
  private apiClient = inject(ApiClient);
}
```

### useExisting

Creates an alias ensuring both tokens return the same singleton instance:

```typescript
providers: [
  NewLogger,
  {provide: OldLogger, useExisting: NewLogger},
];
```

**Note:** Unlike `useClass` which creates separate instances, `useExisting` maintains singleton behavior.

### Multiple Providers

The `multi: true` flag aggregates multiple provider contributions into an array:

```typescript
export const INTERCEPTOR_TOKEN = new InjectionToken<Interceptor[]>('interceptors');

providers: [
  {provide: INTERCEPTOR_TOKEN, useClass: AuthInterceptor, multi: true},
  {provide: INTERCEPTOR_TOKEN, useClass: LoggingInterceptor, multi: true},
  {provide: INTERCEPTOR_TOKEN, useClass: RetryInterceptor, multi: true},
];
```

Injecting `INTERCEPTOR_TOKEN` returns an array containing all registered interceptors.

## Provider Registration Locations

### Application Bootstrap

Use application-level providers via `bootstrapApplication` for:

- Services shared across multiple features
- True singletons available application-wide
- Global configuration and error handling

```typescript
bootstrapApplication(App, {
  providers: [
    {provide: API_BASE_URL, useValue: 'https://api.example.com'},
    {provide: INTERCEPTOR_TOKEN, useClass: AuthInterceptor, multi: true},
    LoggingService,
    {provide: ErrorHandler, useClass: GlobalErrorHandler},
  ],
});
```

**Benefits:** Single instance, global availability, simplified setup.

**Drawbacks:** Always included in bundle, harder per-feature customization, reduced test isolation.

#### Bootstrap vs. providedIn: 'root'

Provide at bootstrap when:

- The provider has side-effects
- Configuration is required
- Using Angular's `provideSomething` functions (`provideRouter`, `provideHttpClient`)

### Component or Directive Providers

Use for:

- Component-specific state and validation
- Isolated instances per component tree
- Reusable components with independent services

```typescript
@Component({
  selector: 'app-advanced-form',
  providers: [
    FormValidationService,
    {provide: FORM_CONFIG, useValue: {strictMode: true}},
  ],
})
export class AdvancedForm {}

@Component({
  selector: 'app-modal',
  providers: [ModalStateService],
})
export class Modal {}
```

**Benefits:** Better encapsulation, easier component testing, multiple configurations.

**Drawbacks:** New instances per component, no shared state, higher memory usage.

### Route Providers

Use for feature-specific and lazy-loaded services:

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    providers: [
      AdminService,
      {provide: FEATURE_FLAGS, useValue: {adminMode: true}},
    ],
    loadChildren: () => import('./admin/admin.routes'),
  },
  {
    path: 'shop',
    providers: [
      ShoppingCartService,
      PaymentService,
    ],
    loadChildren: () => import('./shop/shop.routes'),
  },
];
```

## Library Author Patterns

### The Provide Pattern

Export provider configuration functions for cleaner APIs:

```typescript
export interface AnalyticsConfig {
  trackingId: string;
  enableDebugMode?: boolean;
  anonymizeIp?: boolean;
}

const ANALYTICS_CONFIG = new InjectionToken<AnalyticsConfig>('analytics.config');

export class AnalyticsService {
  private config = inject(ANALYTICS_CONFIG);
  track(event: string, properties?: any) {
    // Implementation using config
  }
}

export function provideAnalytics(config: AnalyticsConfig): Provider[] {
  return [
    {provide: ANALYTICS_CONFIG, useValue: config},
    AnalyticsService
  ];
}

// Consumer usage
bootstrapApplication(App, {
  providers: [
    provideAnalytics({
      trackingId: 'GA-12345',
      enableDebugMode: !environment.production,
    }),
  ],
});
```

### Advanced Patterns with Feature Options

```typescript
export enum HttpFeatures {
  Interceptors = 'interceptors',
  Caching = 'caching',
  Retry = 'retry',
}

export interface HttpConfig {
  baseUrl?: string;
  timeout?: number;
  headers?: Record<string, string>;
}

export interface RetryConfig {
  maxAttempts: number;
  delayMs: number;
}

const HTTP_CONFIG = new InjectionToken<HttpConfig>('http.config');
const RETRY_CONFIG = new InjectionToken<RetryConfig>('retry.config');
const HTTP_FEATURES = new InjectionToken<Set<HttpFeatures>>('http.features');

class HttpClientService {
  private config = inject(HTTP_CONFIG, {optional: true});
  private features = inject(HTTP_FEATURES);
  get(url: string) {
    // Use config and check features
  }
}

class RetryInterceptor {
  private config = inject(RETRY_CONFIG);
  // Retry logic
}

export function provideHttpClient(config?: HttpConfig, ...features: HttpFeature[]): Provider[] {
  const providers: Provider[] = [
    {provide: HTTP_CONFIG, useValue: config || {}},
    {provide: HTTP_FEATURES, useValue: new Set(features.map((f) => f.kind))},
    HttpClientService,
  ];

  features.forEach((feature) => {
    providers.push(...feature.providers);
  });

  return providers;
}

export interface HttpFeature {
  kind: HttpFeatures;
  providers: Provider[];
}

export function withInterceptors(...interceptors: any[]): HttpFeature {
  return {
    kind: HttpFeatures.Interceptors,
    providers: interceptors.map((interceptor) => ({
      provide: INTERCEPTOR_TOKEN,
      useClass: interceptor,
      multi: true,
    })),
  };
}

export function withCaching(): HttpFeature {
  return {
    kind: HttpFeatures.Caching,
    providers: [CacheInterceptor],
  };
}

export function withRetry(config: RetryConfig): HttpFeature {
  return {
    kind: HttpFeatures.Retry,
    providers: [
      {provide: RETRY_CONFIG, useValue: config},
      RetryInterceptor
    ],
  };
}

// Consumer usage
bootstrapApplication(App, {
  providers: [
    provideHttpClient(
      {baseUrl: 'https://api.example.com'},
      withInterceptors(AuthInterceptor, LoggingInterceptor),
      withCaching(),
      withRetry({maxAttempts: 3, delayMs: 1000}),
    ),
  ],
});
```

### Benefits of Provider Functions

Provider functions offer encapsulation, type safety, flexibility through composition, future-proofing, and consistency with Angular's own patterns. This approach is considered best practice for configurable library services.
