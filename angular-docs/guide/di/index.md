# Dependency Injection in Angular
> Source: https://angular.dev/guide/di

Dependency Injection (DI) is a design pattern used to organize and share code across an application. It enables cleaner code architecture by allowing developers to inject features into different application components rather than creating dependencies internally.

## Key Benefits

1. **Code Maintainability**: Supports cleaner separation of concerns which enables easier refactoring and reducing code duplication
2. **Scalability**: Allows modular functionality reuse across multiple contexts
3. **Testing**: Facilitates use of test doubles when real implementations are not practical

## Core Concepts

### What is a Dependency?

A dependency represents any object, value, function, or service that a class requires but does not create itself. This establishes relationships between application parts.

### Two-Way Interaction

DI systems operate on two principles:

- **Providing**: Making values available for injection
- **Injecting**: Requesting those values as dependencies

### Common Dependency Types

- Configuration values (environment constants, API URLs, feature flags)
- Factories (functions creating objects based on runtime conditions)
- Services (classes providing shared functionality and business logic)

## Services in Angular

An Angular service is a TypeScript class decorated with `@Injectable` that becomes available for injection as a dependency.

### Typical Service Categories

- Data clients abstracting server communication
- State management across components
- Authentication and authorization handling
- Logging and error management
- Event handling following observer patterns
- Reusable utility functions

### Service Example

```typescript
import {Injectable} from '@angular/core';

@Injectable({providedIn: 'root'})
export class AnalyticsLogger {
  trackEvent(category: string, value: string) {
    console.log('Analytics event logged:', {
      category,
      value,
      timestamp: new Date().toISOString(),
    });
  }
}
```

## Dependency Injection with `inject()`

Angular provides the `inject()` function for retrieving dependencies during class construction.

### Valid Usage Locations

- Class field initializers
- Constructor bodies
- Service classes
- Route guards

### Navigation Example

```typescript
import {Component, inject} from '@angular/core';
import {Router} from '@angular/router';
import {AnalyticsLogger} from './analytics-logger';

@Component({
  selector: 'app-navbar',
  template: `<a href="#" (click)="navigateToDetail($event)">Detail Page</a>`,
})
export class Navbar {
  private router = inject(Router);
  private analytics = inject(AnalyticsLogger);

  navigateToDetail(event: Event) {
    event.preventDefault();
    this.analytics.trackEvent('navigation', '/details');
    this.router.navigate(['/details']);
  }
}
```

## Injection Context

Angular defines "injection context" as any location where `inject()` can be called. While component, directive, and service construction represent common scenarios, additional contexts exist for specialized use cases. See [Dependency Injection Context](di-context.md) for more details.

## Next Steps

- [Creating and using services](creating-injectable-service.md) - Service creation via Angular CLI or manual implementation, understanding the `providedIn: 'root'` pattern, and injecting services into components and other services
- [Dependency Providers](providers.md) - Defining providers, InjectionToken, and advanced provider patterns
- [Hierarchical Dependency Injection](hierarchical-di.md) - Understanding injector hierarchies and resolution modifiers
- [DI in Action](di-in-action.md) - Additional dependency injection features and patterns

**Note**: The `providedIn: 'root'` configuration represents the recommended approach for most services, making them available application-wide as singletons.
