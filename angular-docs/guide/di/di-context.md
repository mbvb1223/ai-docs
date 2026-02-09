# Injection Context
> Source: https://angular.dev/guide/di/dependency-injection-context

The dependency injection (DI) system in Angular relies internally on a runtime context where the current injector is available. This means injectors only work when code executes within such a context.

## When Injection Context is Available

The injection context is accessible in these situations:

- During construction (via the `constructor`) of a class instantiated by the DI system, such as an `@Injectable` or `@Component`
- In field initializers for such classes
- In the factory function specified for `useFactory` of a `Provider` or `@Injectable`
- In the `factory` function specified for an `InjectionToken`
- Within a stack frame that runs in an injection context

Understanding when you are in an injection context allows you to use the `inject` function to inject instances.

## Stack Frame in Context

Some APIs are designed to run in an injection context. Router guards exemplify this, allowing use of `inject` within the guard function to access services.

### Example: CanActivateFn

```typescript
const canActivateTeam: CanActivateFn = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot,
) => {
  return inject(PermissionsService).canActivate(
    inject(UserToken),
    route.params.id
  );
};
```

## Run Within an Injection Context

When you need to execute a function in an injection context without already being in one, use `runInInjectionContext`. This requires access to an injector, such as `EnvironmentInjector`:

```typescript
@Injectable({
  providedIn: 'root',
})
export class HeroService {
  private environmentInjector = inject(EnvironmentInjector);

  someMethod() {
    runInInjectionContext(this.environmentInjector, () => {
      inject(SomeService); // Do what you need with the injected service
    });
  }
}
```

Note: `inject` returns an instance only if the injector can resolve the required token.

## Asserts the Context

Angular provides `assertInInjectionContext` to confirm the current context is an injection context and throw a clear error if not. Pass a reference to the calling function so the error message points to the correct API entry point.

### Example: Helper Function

```typescript
import { ElementRef, assertInInjectionContext, inject } from '@angular/core';

export function injectNativeElement<T extends Element>(): T {
  assertInInjectionContext(injectNativeElement);
  return inject(ElementRef).nativeElement;
}
```

### Usage Example

```typescript
import { Component, inject } from '@angular/core';
import { injectNativeElement } from './dom-helpers';

@Component({
  /* ... */
})
export class PreviewCard {
  readonly hostEl = injectNativeElement<HTMLElement>();
  // Field initializer runs in an injection context.

  onAction() {
    const anotherRef = injectNativeElement<HTMLElement>();
    // Fails: runs outside an injection context.
  }
}
```

## Using DI Outside of a Context

Calling `inject` or `assertInInjectionContext` outside of an injection context throws error NG0203.
