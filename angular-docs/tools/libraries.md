# Overview of Angular Libraries
> Source: https://angular.dev/tools/libraries

## Introduction

Angular libraries serve as reusable solutions for common application problems. Many applications need to solve the same general problems, such as presenting a unified user interface, presenting data, and allowing data entry.

Libraries differ fundamentally from applications -- they cannot run independently but must be imported and used within an application context.

## What Libraries Extend

Angular libraries augment the framework's core functionality. Adding reactive forms requires using `ng add @angular/forms`, then importing the `ReactiveFormsModule` into application code. Similar patterns apply to service workers for Progressive Web Apps and Angular Material for sophisticated UI components.

Any developer can utilize libraries published as npm packages by Angular or third-party creators.

## Important Distinction

Libraries are intended to be used by Angular applications. To add Angular features to non-Angular web applications, use Angular custom elements.

## Creating Your Own Libraries

### When to Package Features as Libraries

Developing your own libraries requires an architectural decision comparable to determining component or service scope. Custom libraries can be used locally within a workspace or published as npm packages to the npm registry, private npm Enterprise registries, or private package management systems.

### Benefits and Trade-offs

**Advantages:** Packaging features as libraries enforces decoupling from application business logic, helping prevent architectural mistakes that hinder future code reuse.

**Challenges:** Separating code into libraries introduces greater complexity than monolithic applications. It requires more of an investment in time and thought for managing, maintaining, and updating the library. This complexity can pay off when the library is being used in multiple applications.

For comprehensive guidance, see the Creating Libraries documentation.
