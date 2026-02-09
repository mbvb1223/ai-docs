# NgModules
> Source: https://angular.dev/guide/ngmodules

## Overview

An NgModule is a class decorated with `@NgModule()` that accepts metadata telling Angular how to compile component templates and configure dependency injection.

```typescript
import {NgModule} from '@angular/core';

@NgModule({
  // Metadata goes here
})
export class CustomMenuModule {}
```

**Key Responsibility**: NgModules handle declaring components, directives, and pipes that belong to the module, plus adding providers to the injector.

**Important**: The Angular team recommends using standalone components instead of `@NgModule()` for new code. This guide helps understand existing code built with NgModules.

## Declarations

The `declarations` property specifies which components, directives, and pipes belong to an NgModule.

```typescript
@NgModule({
  declarations: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule {}
```

### Important Rules

- Angular reports an error if components, directives, or pipes appear in multiple NgModules
- Components must be explicitly marked as `standalone: false` to be declared in an NgModule

```typescript
@Component({
  standalone: false,
  /* ... */
})
export class CustomMenu { }
```

### Nested Arrays

The declarations property accepts arrays containing other arrays:

```typescript
const MENU_COMPONENTS = [CustomMenu, CustomMenuItem];
const WIDGETS = [MENU_COMPONENTS, CustomSlider];

@NgModule({
  declarations: [WIDGETS, CustomCheckbox],
})
export class CustomMenuModule {}
```

## Imports

Components declared in an NgModule may depend on other components, directives, and pipes. Add these to the `imports` property:

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule {}
```

The imports array accepts other NgModules, standalone components, directives, and pipes.

## Exports

An NgModule can export its declared components, directives, and pipes, making them available to other components and NgModules.

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  exports: [CustomMenu, CustomMenuItem],
})
export class CustomMenuModule {}
```

Exports aren't limited to declarations -- an NgModule can also export any imported components, directives, pipes, and NgModules:

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  exports: [CustomMenu, CustomMenuItem, PopupTrigger],
})
export class CustomMenuModule {}
```

## NgModule Providers

An NgModule can specify `providers` for injected dependencies. These are available to:

- Any standalone component, directive, or pipe that imports the NgModule
- The declarations and providers of other NgModules that import this NgModule

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  providers: [OverlayManager],
})
export class CustomMenuModule {}

@NgModule({
  imports: [CustomMenuModule],
  declarations: [UserProfile],
  providers: [UserDataClient],
})
export class UserProfileModule {}
```

### Injection Availability

- `CustomMenu` and `CustomMenuItem` can inject `OverlayManager` (declared in the providing module)
- `UserProfile` can inject `OverlayManager` (its module imports the providing module)
- `UserDataClient` can inject `OverlayManager` (its module imports the providing module)

### The forRoot and forChild Pattern

Some NgModules define static `forRoot()` methods accepting configuration and returning provider arrays. This convention indicates providers should be added exclusively at the application root during bootstrap.

```typescript
bootstrapApplication(MyApplicationRoot, {
  providers: [CustomMenuModule.forRoot(/* config */)],
});
```

Similarly, `forChild()` indicates providers are intended for components within the application hierarchy:

```typescript
@Component({
  providers: [CustomMenuModule.forChild(/* config */)],
})
export class UserProfile { }
```

## Bootstrapping an Application

The `@NgModule()` decorator accepts an optional `bootstrap` array containing one or more components. Use `bootstrapModule()` to start an application:

```typescript
import {platformBrowser} from '@angular/platform-browser';

@NgModule({
  bootstrap: [MyApplication],
})
export class MyApplicationModule {}

platformBrowser().bootstrapModule(MyApplicationModule);
```

**Note**: Components listed in `bootstrap` are automatically included in the NgModule's declarations. When bootstrapped, collected providers from this module and all imported modules are eagerly loaded and available application-wide.
