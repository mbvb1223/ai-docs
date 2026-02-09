# Creating and Using Services
> Source: https://angular.dev/guide/di/creating-injectable-service (redirects from https://angular.dev/guide/di/creating-and-using-services)

Services are reusable code pieces shared across Angular applications, typically handling data fetching, business logic, or functionality needed by multiple components.

## Creating a Service

### Using Angular CLI

Generate a service with:

```bash
ng generate service CUSTOM_NAME
```

This creates a dedicated `CUSTOM_NAME.ts` file in your `src` directory.

### Manual Creation

Add the `@Injectable()` decorator to a TypeScript class to make it injectable:

```typescript
// src/app/basic-data-store.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class BasicDataStore {
  private data: string[] = [];

  addData(item: string): void {
    this.data.push(item);
  }

  getData(): string[] {
    return [...this.data];
  }
}
```

## How Services Become Available

Using `@Injectable({ providedIn: 'root' })` means Angular will:

- **Create a single instance** (singleton) for your entire application
- **Make it universally available** without additional configuration
- **Enable tree-shaking** so unused services don't bloat your bundle

This represents the recommended approach for most scenarios.

## Injecting a Service

Once created with `providedIn: 'root'`, inject services anywhere using the `inject()` function from `@angular/core`.

### Into a Component

```typescript
import { Component, inject } from '@angular/core';
import { BasicDataStore } from './basic-data-store';

@Component({
  selector: 'app-example',
  template: `
    <div>
      <p>{{ dataStore.getData() }}</p>
      <button (click)="dataStore.addData('More data')">
        Add more data
      </button>
    </div>
  `,
})
export class Example {
  dataStore = inject(BasicDataStore);
}
```

### Into Another Service

```typescript
import { inject, Injectable } from '@angular/core';
import { AdvancedDataStore } from './advanced-data-store';

@Injectable({
  providedIn: 'root',
})
export class BasicDataStore {
  private advancedDataStore = inject(AdvancedDataStore);
  private data: string[] = [];

  addData(item: string): void {
    this.data.push(item);
  }

  getData(): string[] {
    return [...this.data, ...this.advancedDataStore.getData()];
  }
}
```

## Next Steps

While `providedIn: 'root'` covers most cases, Angular provides additional patterns for specialized scenarios:

- **Component-specific instances** -- Isolated service instances per component
- **Manual configuration** -- Services requiring runtime setup
- **Factory providers** -- Dynamic service creation based on conditions
- **Value providers** -- Configuration objects or constants

Explore these advanced patterns in the guide on [defining dependency providers](providers.md).
