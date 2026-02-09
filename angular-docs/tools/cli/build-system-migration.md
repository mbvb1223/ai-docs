# Angular Application Build System Migration Guide
> Source: https://angular.dev/tools/cli/build-system-migration

## Overview

Angular v17+ introduces a modern build system offering significant improvements over the webpack-based approach. The system features ESM output format, faster builds, integration with esbuild and Vite, built-in SSR capabilities, and automatic stylesheet hot replacement.

**Key Point:** The existing webpack-based build system remains stable and fully supported. Migration is optional.

## For New Applications

New Angular projects automatically use the `application` builder by default.

## For Existing Applications

### Automated Migration (Recommended)

Starting with v18, `ng update` prompts users about migration. This can also be triggered manually:

```bash
ng update @angular/cli --name use-application-builder
```

The automated process:
- Converts `browser` or `browser-esbuild` targets to `application`
- Removes previous SSR builders
- Merges `tsconfig.server.json` with `tsconfig.app.json`
- Adds `"esModuleInterop": true` for ESM compliance
- Updates server code for new bootstrapping patterns
- Removes webpack-specific stylesheet syntax
- Converts to `@angular/build` dependency when applicable

### Manual Migration Options

**Option 1: `browser-esbuild` Builder**

Minimal changes required. Simply update `angular.json`:

```json
{
  "architect": {
    "build": {
      "builder": "@angular-devkit/build-angular:browser-esbuild"
    }
  }
}
```

**Option 2: `application` Builder**

More comprehensive but recommended. Update the builder and adjust options:

```json
{
  "architect": {
    "build": {
      "builder": "@angular-devkit/build-angular:application"
    }
  }
}
```

#### Required Configuration Changes

When migrating to `application`:

- Rename `main` to `browser`
- Convert `polyfills` to array format
- Remove `buildOptimizer` (covered by `optimization`)
- Remove `resourcesOutputPath` (now always `media`)
- Remove `vendorChunk` and `commonChunk` (no longer needed)
- Remove `deployUrl` (use `<base href>` instead)
- Rename `ngswConfigPath` to `serviceWorker`

### Important ESM Note

Remember to remove any CommonJS assumptions in the application server code if using SSR such as `require`, `__filename`, `__dirname`, or other constructs from the CommonJS module scope. All application code should be ESM compatible.

## Building and Development

### Standard Build

```bash
ng build
```

### Development Server

No configuration changes needed:

```bash
ng serve
```

The development server automatically detects and uses the new build system. All previous command-line options remain functional.

**Note:** Minor Flash of Unstyled Content (FOUC) may appear on startup as the server defers stylesheet processing to improve rebuild times.

## Hot Module Replacement (HMR)

Automatic HMR support includes:
- Global stylesheets (`styles` build option)
- Component stylesheets (inline and file-based)
- Component templates (inline and file-based)

Disable HMR if needed:

```bash
ng serve --no-hmr
```

## Vite as Development Server

Vite is integrated for development server functionality only, not production builds. It's bundled as a low-dependency package providing comprehensive development capabilities. Configuration occurs automatically within the Angular CLI `dev-server` builder.

## Prebundling

Vite's prebundling optimizes development server performance by analyzing third-party dependencies. Enabled by default.

### Configuration Examples

Exclude specific dependencies:

```json
{
  "serve": {
    "builder": "@angular/build:dev-server",
    "options": {
      "prebundle": {
        "exclude": ["some-dep"]
      }
    }
  }
}
```

Disable entirely:

```json
{
  "serve": {
    "builder": "@angular/build:dev-server",
    "options": {
      "prebundle": false
    }
  }
}
```

## New Features

**Important:** These features are incompatible with the `karma` builder by default. Enable via `builderMode: "application"` option (developer preview).

### Build-Time Value Replacement (`define`)

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

Environment variables:

```bash
export MY_API="http://example.com"
ng build --define API_HOST=\'$MY_API\'
```

**TypeScript Support:** Add type definitions to prevent errors:

```typescript
// src/types.d.ts
declare const SOME_NUMBER: number;
declare const ANOTHER: string;
```

**Limitation:** Cannot replace identifiers within Angular metadata (decorators).

### File Extension Loader Customization

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

Example usage:

```typescript
import contents from './some-file.svg';
console.log(contents); // <svg>...</svg>
```

Type definition:

```typescript
declare module '*.svg' {
  const content: string;
  export default content;
}
```

### Import Attribute Loader Customization

Per-file control using import attributes:

```typescript
import contents from './file.svg' with {loader: 'text'};
```

Requires TypeScript `module: "esnext"` setting.

**Type Checking Workaround:**

```typescript
// @ts-expect-error
import contents from './file.svg' with {loader: 'text'};
```

Examples by loader type:

**Text:**

```typescript
// @ts-expect-error
import svgContent from './image.svg' with {loader: 'text'};
```

**File:**

```typescript
// @ts-expect-error
import imagePath from './image.webp' with {loader: 'file'};
console.log(imagePath); // media/image-ULK2SIIB.webp
```

**Base64:**

```typescript
// @ts-expect-error
import logo from './logo.png' with {loader: 'base64'};
```

**Data URL:**

```typescript
// @ts-expect-error
import icon from './icon.svg' with {loader: 'dataurl'};
```

### Import/Export Conditions

Automatically applied conditions for conditional exports:

- `production` - enabled for optimized builds
- `development` - enabled for non-optimized builds
- `browser` - enabled for browser output

**Optimization Determination:** Set by `optimization` option; `ng build` optimizes by default, `ng serve` doesn't.

**Usage with Subpath Imports:**

```typescript
import {verboseLogging} from '#logger';
```

Configure in `package.json`:

```json
{
  "imports": {
    "#logger": {
      "development": "./src/logging/debug.ts",
      "default": "./src/logging/noop.ts"
    }
  }
}
```

**Server/Browser Switching:**

```json
{
  "imports": {
    "#crashReporter": {
      "browser": "./src/browser-logger.ts",
      "default": "./src/server-logger.ts"
    }
  }
}
```

## Known Issues

### Web Worker Type-Checking

Workers use standard syntax (`new Worker(new URL('<file>', import.meta.url))`) but lack type-checking. TypeScript code is supported but unvalidated. Nested workers are not processed.

### ESM Default vs. Namespace Imports

TypeScript allows non-compliant namespace imports of default exports. The build system warns about potential runtime errors.

**Solution:** Enable `esModuleInterop` in `tsconfig.json`:

```typescript
// Before (causes warning/error)
import * as moment from 'moment';

// After (correct)
import moment from 'moment';
```

### Order-Dependent Side-Effectful Imports

Side-effectful imports in multiple lazy modules may execute out of order. This is rare but caused by an underlying bundler defect.

**Recommendation:** Avoiding the use of modules with non-local side effects (outside of polyfills) is recommended whenever possible regardless of the build system being used.

### Output Location Changes

Default output location changed to `dist/<project-name>/browser` instead of `dist/<project-name>`. Reconfigure if needed via workspace config output path settings.

## Bug Reports

Report issues at [GitHub](https://github.com/angular/angular-cli/issues) with minimal reproductions when possible.
