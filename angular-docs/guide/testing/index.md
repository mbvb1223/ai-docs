# Testing
> Source: https://angular.dev/guide/testing

## Overview

Angular testing helps verify applications work as expected. Unit tests catch bugs early, ensure code quality, and enable safe refactoring. The Angular CLI uses **Vitest** as the default testing framework for new projects.

> Testing your Angular application helps you check that it is working as you expect.

## Setup for Testing

The CLI automatically includes Vitest and jsdom (a DOM emulation library) in new projects. Vitest executes tests in a Node.js environment, avoiding browser overhead while simulating DOM interactions.

### Running Tests

Execute the test command:

```bash
ng test
```

This builds the application in watch mode and launches Vitest. The test runner automatically re-executes when files change.

### Sample Output

```
✓ src/app/app.spec.ts (3)
 ✓ AppComponent should create the app
 ✓ AppComponent should have as title 'my-app'
 ✓ AppComponent should render title

Test Files  1 passed (1)
Tests  3 passed (3)
Start at  18:18:01
Duration  2.46s
```

## Configuration

### Angular.json Test Options

Configure test behavior through `angular.json`:

- **include**: File glob patterns (default: `['**/*.spec.ts', '**/*.test.ts']`)
- **exclude**: Patterns to skip
- **setupFiles**: Global setup files executed before tests
- **providersFile**: Path to file exporting test providers
- **coverage**: Enable/disable code coverage reporting
- **browsers**: Real browser names for testing (requires provider installation)

### Global Test Providers

Create a providers file to inject Angular dependencies globally:

```typescript
// src/test-providers.ts
import {Provider} from '@angular/core';
import {provideHttpClient} from '@angular/common/http';
import {provideHttpClientTesting} from '@angular/common/http/testing';

const testProviders: Provider[] = [
  provideHttpClient(),
  provideHttpClientTesting()
];

export default testProviders;
```

Reference in `angular.json`:

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "providersFile": "src/test-providers.ts"
          }
        }
      }
    }
  }
}
```

### Advanced Vitest Configuration

For custom configurations, create a Vitest config file and reference it:

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "runnerConfig": "vitest-base.config.ts"
          }
        }
      }
    }
  }
}
```

Generate a base config:

```bash
ng generate config vitest
```

## Code Coverage

Generate coverage reports with:

```bash
ng test --coverage
```

Reports appear in the `coverage/` directory.

## Browser Testing

Run tests in actual browsers instead of Node.js:

### Install Browser Providers

**Playwright** (Chromium, Firefox, WebKit):

```bash
npm install --save-dev @vitest/browser-playwright playwright
```

**WebdriverIO** (Chrome, Firefox, Safari, Edge):

```bash
npm install --save-dev @vitest/browser-webdriverio webdriverio
```

### Execute in Browser

```bash
# Headed mode (Playwright)
ng test --browsers=chromium

# Headless mode
ng test --browsers=chromiumHeadless

# WebdriverIO examples
ng test --browsers=chrome
ng test --browsers=chromeHeadless
```

Tests run headed by default; headless mode activates when `CI` environment variable is set.

## Continuous Integration

Run tests non-interactively in CI pipelines:

```bash
ng test --no-watch --no-progress
```

Most CI systems set `CI=true`, which `ng test` automatically detects for single-run mode.

## Additional Testing Resources

- **Code Coverage**: Measuring test coverage and setting minimum thresholds
- [Testing Services](./services.md): Unit testing Angular services
- [Component Testing Basics](./components-basics.md): Fundamental component test patterns
- [Component Scenarios](./components-scenarios.md): Real-world testing situations
- [Attribute Directives](./attribute-directives.md): Testing custom directives
- [Pipes](./pipes.md): Unit testing pipes
- **Routing & Navigation**: Testing route behavior
- **Debugging Tests**: Troubleshooting common issues
- [Utility APIs](./utility-apis.md): Angular testing framework features
- **Component Harnesses**: Reusable test helpers
