# Unhandled Errors in Angular
> Source: https://angular.dev/best-practices/error-handling

## Overview

As Angular applications execute, code may throw errors. If left unhandled, these errors lead to unexpected behavior and unresponsive UIs. This guide explains how Angular manages errors not explicitly caught by application code.

**Core Principle:** Errors should be surfaced to developers at the callsite whenever possible. This ensures the code initiating an operation has sufficient context to handle it appropriately.

## Unhandled Errors and ErrorHandler

Angular reports unhandled errors to the application's root `ErrorHandler`. When creating a custom `ErrorHandler`, provide it in your `ApplicationConfig` when calling `bootstrapApplication`.

### When Angular Catches Errors

Angular catches errors when:

1. **Framework-initiated code execution** -- Angular runs component constructors, lifecycle methods, and other framework code where developers cannot reasonably add try-catch blocks
2. **Asynchronous operations** -- Only when there is an explicit contract for Angular to wait for and use results, AND errors are not presented in return values or state

### When Angular Does NOT Catch Errors

- Errors inside APIs called directly by application code
- Service methods throwing errors when called from components (developer responsibility)
- Example: calling a service method requires developer-added `try...catch` blocks

### Specific API Behaviors

- `AsyncPipe` and `PendingTasks.run` forward errors to `ErrorHandler`
- `resource` presents errors in `status` and `error` properties (not forwarded)

### ErrorHandler Purpose

Errors reported to `ErrorHandler` are *unexpected* and potentially unrecoverable, indicating possible application state corruption. Applications should use `try` blocks or operators like `catchError` in RxJS where errors occur, rather than relying solely on `ErrorHandler` for error recovery.

### Example Implementation

```typescript
export class GlobalErrorHandler implements ErrorHandler {
  private readonly analyticsService = inject(AnalyticsService);
  private readonly router = inject(Router);

  handleError(error: any) {
    const url = this.router.url;
    const errorMessage = error?.message ?? 'unknown';

    this.analyticsService.trackEvent({
      eventName: 'exception',
      description: `Screen: ${url} | ${errorMessage}`,
    });

    console.error(GlobalErrorHandler.name, {error});
  }
}
```

## TestBed Error Handling

Angular's `TestBed` rethrows unexpected errors by default to prevent unintentionally missing framework-caught errors. For rare cases where tests verify the application remains responsive despite errors, configure:

```typescript
TestBed.configureTestingModule({rethrowApplicationErrors: false})
```

## Global Error Listeners

Errors reaching the global scope -- caught neither by application code nor the framework -- can cause crashes (non-browser environments) or go unreported (browser). Angular provides global listeners for both environments.

### Client-Side Rendering

`provideBrowserGlobalErrorListeners()` added to `ApplicationConfig`:
- Adds `'error'` and `'unhandledrejection'` listeners to browser window
- Forwards those errors to `ErrorHandler`
- Angular CLI includes this provider by default
- Recommended for most applications (or implement custom listeners)

### Server-Side and Hybrid Rendering

When using Angular with SSR, Angular automatically adds:
- `'unhandledRejection'` and `'uncaughtException'` listeners to server process
- Prevents server crashes
- Logs captured errors to console

**Important Note:** With Zone.js present, only `'unhandledRejection'` handler is added. Zone.js already forwards Application Zone errors to `ErrorHandler`, preventing them from reaching the server process.
