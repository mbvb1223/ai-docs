# Angular Migrations
> Source: https://angular.dev/reference/migrations

## Overview

This is the Angular migrations reference documentation, serving as a hub for various Angular code modernization guides. These migrations help you update your codebase to use newer Angular patterns and APIs.

## Available Migrations

### 1. Standalone

Standalone components provide a simplified way to build Angular applications. Standalone components specify their dependencies directly instead of getting them through NgModules.

### 2. Control Flow Syntax

Built-in Control Flow Syntax allows you to use more ergonomic syntax which is close to JavaScript and has better type checking. It replaces the need to import `CommonModule` for directives like `*ngFor`, `*ngIf`, and `*ngSwitch`.

### 3. inject() Function

Angular's `inject` function offers more accurate types and better compatibility with standard decorators, compared to constructor-based injection.

### 4. Lazy-loaded Routes

Converting eagerly loaded component routes to lazy loaded ones enables bundle splitting, reducing JavaScript at initial page load.

### 5. New `input()` API

Migrate existing `@Input` fields to the signal input API, now production-ready.

### 6. New `output()` Function

Convert existing `@Output` custom events to the new output function, now production-ready.

### 7. Queries as Signal

Transition decorator query fields to the improved signal queries API.

### 8. Cleanup Unused Imports

Remove unused imports across your project.

### 9. Self-closing Tags

Update component templates to use self-closing tags where possible.

### 10. NgClass to Class Bindings

Replace `NgClass` directives with class bindings over the `NgClass` directives when possible.

### 11. NgStyle to Style Bindings

Replace `NgStyle` with style bindings where applicable.

### 12. RouterTestingModule Migration

Convert `RouterTestingModule` to `RouterModule` in TestBed configurations with `provideLocationMocks()`.

### 13. CommonModule to Standalone Imports

Replace `CommonModule` imports with individual directives and pipes.
