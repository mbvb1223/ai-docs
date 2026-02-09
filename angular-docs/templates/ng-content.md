# Slotting Child Content with ng-content
> Source: https://angular.dev/guide/templates/ng-content

## Overview

The `<ng-content>` element is a special Angular feature that enables components to accept and render markup or template fragments from parent components. Notably, it does not render as a real DOM element itself.

## Basic Example

### BaseButton Component

```typescript
// ./base-button/base-button.ts
import { Component } from '@angular/core';

@Component({
  selector: 'button[baseButton]',
  template: `<ng-content />`,
})
export class BaseButton {}
```

The component uses an attribute selector `button[baseButton]` and includes a simple template containing only the `<ng-content />` element.

### Usage in Parent Component

```typescript
// ./app.ts
import { Component } from '@angular/core';
import { BaseButton } from './base-button';

@Component({
  selector: 'app-root',
  imports: [BaseButton],
  template: `<button baseButton>Next <span class="icon arrow-right"></span></button>`,
})
export class App {}
```

The parent component supplies custom content (text and icon) that gets projected into the child component's `<ng-content>` placeholder.

## Key Concepts

- **Content Projection**: `<ng-content>` allows parent components to pass markup into child components
- **No DOM Rendering**: The `<ng-content>` tag itself does not create a DOM element; it serves as an insertion point
- **Flexibility**: Enables building reusable components that accept varying content structures

## Additional Resources

For comprehensive coverage of advanced `<ng-content>` patterns and techniques, consult the "Content projection in-depth guide".
