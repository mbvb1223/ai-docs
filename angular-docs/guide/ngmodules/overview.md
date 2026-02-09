# NgModules Overview
> Source: https://angular.dev/guide/ngmodules/overview

## Key Definition

An NgModule is a class marked by the `@NgModule` decorator. This decorator accepts metadata that tells Angular how to compile component templates and configure dependency injection.

## Main Responsibilities

NgModules have two primary functions:

1. Declaring components, directives, and pipes that belong to the NgModule
2. Adding providers to the injector for components, directives, and pipes that import the NgModule

## Declarations

The `declarations` property identifies which components, directives, and pipes are owned by a specific NgModule. This property accepts arrays and nested arrays of these elements.

**Important constraint**: If Angular discovers any components, directives, or pipes declared in more than one NgModule, it reports an error.

All components, directives, or pipes must explicitly include `standalone: false` to be declared in an NgModule.

## Imports

Components declared in an NgModule may depend on other components, directives, and pipes. These dependencies are specified in the `imports` property. The imports array accepts other NgModules and standalone components, directives, and pipes.

## Exports

An NgModule can export its declared components, directives, and pipes such that they're available to other components and NgModules.

NgModules can export not only their own declarations but also any imported components, directives, pipes, and other NgModules.

## NgModule Providers

An NgModule can specify `providers` for injected dependencies. These providers are available to:

- Any standalone component, directive, or pipe that imports the NgModule
- The declarations and providers of any other NgModule that imports the NgModule

## The forRoot and forChild Pattern

Some NgModules define static `forRoot` methods accepting configuration and returning provider arrays. The name indicates these providers load exclusively at the application root during bootstrap. This approach eagerly loads dependencies, increasing initial page bundle size.

Similarly, some NgModules define `forChild` methods for providers intended for components within the application hierarchy.

## Bootstrapping Applications

The `@NgModule` decorator accepts an optional `bootstrap` array containing one or more components. The `bootstrapModule` method from `platformBrowser` or `platformServer` renders listed components matching CSS selectors on the page.

Components in `bootstrap` are automatically included in NgModule declarations. When bootstrapping from an NgModule, all collected `providers` from this module and imported modules are eagerly loaded application-wide.

## Important Note

The Angular team recommends using standalone components instead of `NgModule` for all new code. Use this guide to understand existing code built with `@NgModule`.
