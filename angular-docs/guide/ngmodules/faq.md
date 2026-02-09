# NgModules FAQ
> Source: https://angular.dev/guide/ngmodules/faq
>
> Note: This page redirects to the main NgModules guide. The FAQ content has been consolidated into the main NgModules documentation.

## Frequently Asked Questions about NgModules

### What is an NgModule?

An NgModule is a class decorated with `@NgModule()` that accepts metadata telling Angular how to compile component templates and configure dependency injection.

```typescript
import {NgModule} from '@angular/core';

@NgModule({
  // Metadata goes here
})
export class CustomMenuModule {}
```

### What are the key properties of `@NgModule`?

- **declarations**: Components, directives, and pipes that belong to this module
- **imports**: Other modules, standalone components, directives, and pipes needed by this module's declarations
- **exports**: Declarations made available to importing modules
- **providers**: Services available to this module's declarations and importing modules
- **bootstrap**: The root component(s) for application startup

### Can a component belong to multiple NgModules?

No. Angular reports an error if components, directives, or pipes appear in the declarations of more than one NgModule.

### Do I need `standalone: false` for NgModule declarations?

Yes. Components must be explicitly marked as `standalone: false` to be declared in an NgModule:

```typescript
@Component({
  standalone: false,
  /* ... */
})
export class CustomMenu { }
```

### What is the difference between `forRoot` and `forChild`?

- `forRoot()` indicates providers should be added exclusively at the application root during bootstrap
- `forChild()` indicates providers are intended for components within the application hierarchy, typically in feature modules

### Should I use NgModules for new code?

The Angular team recommends using standalone components instead of `@NgModule()` for new code. NgModules remain supported for existing codebases.

For the complete NgModules documentation, see the [NgModules guide](./index.md).
