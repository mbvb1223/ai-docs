# Hierarchical Dependency Injection
> Source: https://angular.dev/guide/di/hierarchical-dependency-injection

Angular implements a sophisticated hierarchical dependency injection system with two main injector hierarchies working in concert: the `EnvironmentInjector` hierarchy and the `ElementInjector` hierarchy.

## Two Injector Hierarchies

### EnvironmentInjector Hierarchy

Configured through `@Injectable()` decorators (using `providedIn` property) or the `providers` array in `ApplicationConfig`. This hierarchy manages application-wide services and is particularly useful for tree-shaking during optimization.

### ElementInjector Hierarchy

Created implicitly at each DOM element. Initially empty unless explicitly configured via the `providers` or `viewProviders` properties on `@Component()` or `@Directive()` decorators.

## The Injector Stack

Angular maintains a multi-level injector stack:

1. **NullInjector** (top) - Throws errors unless `@Optional()` is used
2. **Platform Injector** - Contains platform-specific dependencies
3. **Root EnvironmentInjector** - Configured by `ApplicationConfig` passed to `bootstrapApplication()`
4. **Child Injectors** - Created as needed for components and directives

## Resolution Process

When resolving dependencies, Angular follows a two-phase approach:

1. **ElementInjector Phase** - Searches up the component/directive hierarchy
2. **EnvironmentInjector Phase** - Searches the application-wide provider configuration

The search terminates at the first matching provider. If no provider is found at any level, an error is thrown (unless `@Optional()` is applied).

## Resolution Modifiers

Angular provides four key modifiers to customize dependency resolution:

### @Optional()

Allows unresolved dependencies to return `null` instead of throwing errors.

### @Self()

Limits search to only the current component's `ElementInjector`, ignoring parent providers.

### @SkipSelf()

Begins search in the parent injector, bypassing the current component's providers.

### @Host()

Designates a component as the final stopping point in the injector tree search.

These modifiers can be combined (except `@Self` with `@SkipSelf`, and `@Host` with `@Self`).

## Service Visibility: `providers` vs `viewProviders`

### `providers` array

Makes services visible to the component and all its descendants in the injector tree.

### `viewProviders` array

Restricts service visibility to the component's template view only, excluding projected content (child components via `<ng-content>`).

## Key Principles

- First matching provider wins; Angular does not continue searching once a provider is found
- `@Injectable()` with `providedIn: 'root'` enables tree-shaking benefits over `ApplicationConfig` registration
- Application-level `ApplicationConfig` providers override `@Injectable()` configurations
- Component instances and their associated services are destroyed together

This hierarchical structure provides developers fine-grained control over service scope, lifetime, and visibility throughout Angular applications.
