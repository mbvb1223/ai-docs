# Advanced Component Configuration
> Source: https://angular.dev/guide/components/advanced-configuration

## Overview

This guide covers advanced Angular component configuration options available through the `@Component` decorator. It assumes familiarity with Angular essentials.

## ChangeDetectionStrategy

The `@Component` decorator accepts a `changeDetection` option controlling how Angular monitors components for updates.

### Default Strategy

`ChangeDetectionStrategy.Default` is the standard approach. Angular checks whether a component's DOM requires updates whenever application activity occurs, including user interactions, network responses, and timer events.

### OnPush Strategy

`ChangeDetectionStrategy.OnPush` reduces checking overhead. The framework only updates a component's DOM when:

- A component input receives changes from template binding
- An event listener within the component executes
- The component is explicitly marked for checking via `ChangeDetectorRef.markForCheck()` or utilities like `AsyncPipe`

When an OnPush component is checked, Angular also checks all ancestor components up the application tree.

## PreserveWhitespaces

By default, Angular removes and collapses extra whitespace in templates, typically from newlines and indentation. Set `preserveWhitespaces` to `true` in component metadata to maintain whitespace.

## Custom Element Schemas

Angular throws an error when encountering unknown HTML elements. Include `CUSTOM_ELEMENTS_SCHEMA` in the `schemas` property to disable this validation:

```typescript
import {Component, CUSTOM_ELEMENTS_SCHEMA} from '@angular/core';

@Component({
  ...,
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  template: '<some-unknown-component />'
})
export class ComponentWithCustomElements { }
```

Angular does not support any other schemas currently.
