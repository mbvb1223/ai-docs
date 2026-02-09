# ESBuild-based Build System
> Source: https://angular.dev/tools/cli/esbuild
> Note: This URL redirects to the Angular homepage. The esbuild build system documentation is covered under the build system migration guide. The content below is derived from the Angular build system migration documentation.

## Overview

Angular v17+ introduces a modern build system powered by esbuild, offering significant improvements over the webpack-based approach. The system features ESM output format, faster builds, integration with esbuild and Vite, built-in SSR capabilities, and automatic stylesheet hot replacement.

The existing webpack-based build system remains stable and fully supported. Migration is optional.

## Build Builders Using esbuild

Angular CLI provides two esbuild-powered builders:

| Builder | Purpose |
|---------|---------|
| `@angular-devkit/build-angular:application` | Creates client-side bundles, Node servers, and build-time prerendered routes using esbuild |
| `@angular-devkit/build-angular:browser-esbuild` | Bundles client-side applications for browsers with esbuild |

New applications default to the `application` builder.

## Using the `browser-esbuild` Builder

For minimal migration from webpack, update `angular.json`:

```json
{
  "architect": {
    "build": {
      "builder": "@angular-devkit/build-angular:browser-esbuild"
    }
  }
}
```

This is a drop-in replacement for the `@angular-devkit/build-angular:browser` builder that uses esbuild under the hood.

## Using the `application` Builder

The recommended builder for new and migrated projects:

```json
{
  "architect": {
    "build": {
      "builder": "@angular-devkit/build-angular:application"
    }
  }
}
```

### Required Configuration Changes

When migrating to the `application` builder:

- Rename `main` to `browser`
- Convert `polyfills` to array format
- Remove `buildOptimizer` (covered by `optimization`)
- Remove `resourcesOutputPath` (now always `media`)
- Remove `vendorChunk` and `commonChunk` (no longer needed)
- Remove `deployUrl` (use `<base href>` instead)
- Rename `ngswConfigPath` to `serviceWorker`

## Automated Migration

Starting with Angular v18, `ng update` prompts users about migration. This can also be triggered manually:

```bash
ng update @angular/cli --name use-application-builder
```

## Vite Integration

Vite is integrated as the development server when using esbuild builders. It provides:
- Hot Module Replacement (HMR) for stylesheets and templates
- Prebundling for faster development server startup
- Automatic configuration within the Angular CLI `dev-server` builder

Vite is used for development server functionality only, not production builds.

## Build-Time Value Replacement (`define`)

Replace identifiers at build time:

```json
{
  "build": {
    "builder": "@angular/build:application",
    "options": {
      "define": {
        "SOME_NUMBER": "5",
        "ANOTHER": "'this is a string literal'",
        "REFERENCE": "globalThis.someValue"
      }
    }
  }
}
```

Command-line usage:

```bash
ng build --define SOME_NUMBER=5 --define "ANOTHER='value'"
```

**TypeScript Support:** Add type definitions to prevent errors:

```typescript
// src/types.d.ts
declare const SOME_NUMBER: number;
declare const ANOTHER: string;
```

## File Extension Loader Customization

Control how files with specific extensions load:

```json
{
  "build": {
    "builder": "@angular/build:application",
    "options": {
      "loader": {
        ".svg": "text"
      }
    }
  }
}
```

Available loaders:
- `text` - inlines as string (default export)
- `binary` - inlines as Uint8Array
- `file` - emits file, provides runtime location
- `dataurl` - inlines as data URL
- `base64` - Base64-encoded string
- `empty` - excludes from bundles

## Import/Export Conditions

Automatically applied conditions for conditional exports:

- `production` - enabled for optimized builds
- `development` - enabled for non-optimized builds
- `browser` - enabled for browser output

## Known Issues

### Web Worker Type-Checking
Workers use standard syntax but lack type-checking. TypeScript code is supported but unvalidated.

### ESM Default vs. Namespace Imports
Enable `esModuleInterop` in `tsconfig.json` to resolve import compatibility issues:

```typescript
// Before (causes warning/error)
import * as moment from 'moment';

// After (correct)
import moment from 'moment';
```

### Output Location Changes
Default output location changed to `dist/<project-name>/browser` instead of `dist/<project-name>`.

## Further Reading

See the [Build System Migration Guide](./build-system-migration.md) for the complete migration documentation.
