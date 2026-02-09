# Creating Angular Libraries
> Source: https://angular.dev/tools/libraries/creating-libraries

## Overview

This guide explains how to create and publish Angular libraries to extend Angular functionality. Libraries are appropriate when you need to solve the same problem across multiple applications or share solutions with other developers.

## Getting Started

### Initial Setup

Generate a new library skeleton using Angular CLI:

```bash
ng new my-workspace --no-create-application
cd my-workspace
ng generate library my-lib
```

The `ng generate` command creates a `projects/my-lib` folder containing a component and library structure.

### Naming Conventions

When choosing a library name for npm publication:

- **Avoid** the `ng-` prefix (reserved for Angular framework)
- **Prefer** the `ngx-` prefix to indicate Angular compatibility
- This convention helps differentiate between libraries for different JavaScript frameworks

### Build Commands

Execute these commands to build, test, and lint:

```bash
ng build my-lib --configuration development
ng test my-lib
ng lint my-lib
```

Libraries use `ng-packagr` builder instead of the default webpack builder, ensuring AOT compilation.

### Public API Definition

Define your library's public API in the `public-api.ts` file. All exports from this file become public when imported into applications. Supply documentation (README) for installation and maintenance.

## Refactoring Application Code into Libraries

### Key Considerations

**Component Design:**
- Design components as stateless, avoiding reliance on external variables
- If state is needed, determine whether it belongs to the application or library

**Lifecycle Management:**
- Observables subscribed to internally must be cleaned up during component lifecycle
- Components should expose interactions through inputs (context) and outputs (events)

**Dependency Management:**
- Check internal dependencies for classes and interfaces
- Migrate custom classes, interfaces, and services alongside components
- If the library depends on external libraries (e.g., Angular Material), configure those dependencies

**Service Providers:**
- Services should declare their own providers rather than relying on NgModule or component declarations
- This enables tree-shaking, allowing the compiler to exclude unused services
- Expose optional services using lightweight token design patterns
- Expose global service providers through `provideXYZ()` functions

## Schematics Integration

Libraries can include reusable schematics that integrate with Angular CLI:

**Types of Schematics:**
- Installation schematic enabling `ng add`
- Generation schematics for scaffolding artifacts (components, services, tests)
- Update schematics for dependency updates and breaking change migrations

Choose schematics when customization complexity is high; use dynamic components for simpler, unchanging scenarios.

## Library Publishing

### Build for Distribution

Build libraries using the `production` configuration:

```bash
ng build my-lib
cd dist/my-lib
npm publish
```

Angular CLI uses `ng-packagr` to create npm-compatible packages.

### Asset Management

Include additional assets like theming files, Sass mixins, or documentation in the distributable. Manually add these to the conditional `"exports"` field in `package.json`:

```json
"exports": {
  ".": {
    "sass": "./_index.scss"
  },
  "./theming": {
    "sass": "./_theming.scss"
  }
}
```

### Peer Dependencies

List `@angular/*` dependencies as peer dependencies, not regular dependencies. This ensures all modules use the same Angular instance and prevents application breakage.

## Using Your Own Library

### Building Requirements

Build the library before using it in applications:

```bash
ng build my-lib
```

Import from the library by name:

```typescript
import {myExport} from 'my-lib';
```

### Incremental Builds

Enable background rebuilding with the `--watch` flag:

```bash
ng build my-lib --watch
```

This significantly improves development experience by emitting only amended files.

### Local Linking for Development

Configure the consuming application's `angular.json`:

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "options": {
            "preserveSymlinks": true
          },
          "configurations": {
            "development": {
              "sourceMap": {
                "scripts": true,
                "styles": true,
                "vendor": true
              }
            }
          }
        },
        "serve": {
          "options": {
            "prebundle": {
              "exclude": ["my-lib"]
            }
          }
        }
      }
    }
  }
}
```

**Configuration Details:**
- `preserveSymlinks`: Follows symlinks instead of resolving to original location
- `sourceMap.vendor`: Enables easier debugging of linked code
- `prebundle.exclude`: Ensures proper watching and rebuilding of linked source

## Distribution Formats

| Format | Details |
|--------|---------|
| **Partial-Ivy (recommended)** | Portable code consumable by Ivy applications from Angular v12+ |
| **Full-Ivy** | Contains private Angular Ivy instructions; requires exact Angular version matching |

For npm publication, use partial-Ivy format by setting `"compilationMode": "partial"` in `tsconfig.prod.json`.

## Version Compatibility

The Angular version in applications must be equal to or greater than the version used in dependent libraries. Angular does not support earlier application versions than library versions.

Compiling with `"compilationMode": "partial"` ensures stability across Angular versions for npm distribution.

## Consuming Partial-Ivy Code Outside Angular CLI

For non-CLI applications, use the Angular linker as a Babel plugin:

```javascript
import linkerPlugin from '@angular/compiler-cli/linker/babel';

export default {
  module: {
    rules: [
      {
        test: /\.m?js$/,
        use: {
          loader: 'babel-loader',
          options: {
            plugins: [linkerPlugin],
            compact: false,
            cacheDirectory: true,
          },
        },
      },
    ],
  },
};
```

The Angular CLI integrates this plugin automatically.

## Important Build System Notes

- Applications use `@angular-devkit/build-angular` (webpack-based)
- Libraries use `ng-packagr`
- These systems support different features and produce different outputs
- TypeScript path mappings should point to **built libraries**, not source `.ts` files
