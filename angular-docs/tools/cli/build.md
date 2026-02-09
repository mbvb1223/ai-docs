# Building Angular Apps
> Source: https://angular.dev/tools/cli/build

## Overview

The `ng build` command compiles TypeScript to JavaScript while optimizing, bundling, and minifying output appropriately for Angular applications and libraries.

## Build Builders

Angular CLI provides four primary builders for build targets:

| Builder | Purpose |
|---------|---------|
| `@angular-devkit/build-angular:application` | Creates client-side bundles, Node servers, and build-time prerendered routes using esbuild |
| `@angular-devkit/build-angular:browser-esbuild` | Bundles client-side applications for browsers with esbuild |
| `@angular-devkit/build-angular:browser` | Bundles client-side applications for browsers with webpack |
| `@angular-devkit/build-angular:ng-packagr` | Builds Angular libraries following Angular Package Format standards |

New applications default to the `application` builder, while libraries use `ng-packagr` by default.

## Output Directory

Build results output to `dist/${PROJECT_NAME}` by default.

## Size Budget Configuration

Size budgets establish thresholds preventing application parts from exceeding defined limits. Configure budgets in `angular.json` under the `budgets` section for each environment:

```json
{
  "configurations": {
    "production": {
      "budgets": [
        {
          "type": "initial",
          "maximumWarning": "250kb",
          "maximumError": "500kb"
        }
      ]
    }
  }
}
```

### Size Value Formats

- `123` or `123b` = bytes
- `123kb` = kilobytes
- `123mb` = megabytes
- `12%` = percentage relative to baseline

### Budget Types and Properties

**Budget Types:**
- `bundle` - specific bundle size
- `initial` - JavaScript and CSS for bootstrapping (defaults: 500kb warning, 1mb error)
- `allScript` - all scripts combined
- `all` - entire application
- `anyComponentStyle` - individual component stylesheets (defaults: 2kb warning, 4kb error)
- `anyScript` - single script file
- `any` - any file

**Budget Properties:**
- `name` - bundle name (for type=bundle)
- `baseline` - size for comparison
- `maximumWarning` - warning threshold relative to baseline
- `maximumError` - error threshold relative to baseline
- `minimumWarning` - minimum warning threshold
- `minimumError` - minimum error threshold
- `warning` - threshold for both min & max
- `error` - threshold for both min & max

## CommonJS Dependencies Configuration

The framework recommends native ECMAScript modules (ESM) throughout applications and dependencies. ESM provides better static analysis, enabling more powerful bundle optimizations.

Angular CLI supports CommonJS imports and automatic bundling, but CommonJS modules prevent effective bundler and minifier optimization, resulting in larger bundles.

Disable CommonJS warnings by adding modules to `allowedCommonJsDependencies` in `angular.json`:

```json
"build": {
  "builder": "@angular-devkit/build-angular:browser",
  "options": {
    "allowedCommonJsDependencies": ["lodash"]
  }
}
```

## Browser Compatibility Configuration

Angular CLI uses Browserslist to ensure browser version compatibility. The system automatically transforms JavaScript and CSS features based on supported browser targets but does not automatically add polyfills for missing Web APIs.

Configure polyfills using the `polyfills` option in `angular.json`.

By default, Angular CLI's browserslist configuration targets browsers officially supported by Angular's current major version.

Override internal configuration by running `ng generate config browserslist`, which creates a `.browserslistrc` file matching Angular's supported browsers.

**Important:** Only reduce the browser set -- avoid expanding it, as Angular itself may lack broader compatibility despite application code being compatible.

## Tailwind CSS Integration

Angular supports Tailwind CSS, a utility-first framework. Documentation on integration is available in separate Tailwind-specific guides.

## Critical CSS Inlining

Angular can inline critical CSS to improve First Contentful Paint (FCP) performance. This optimization is enabled by default but can be disabled in style customization options.

The process extracts CSS needed for initial viewport rendering and inlines it directly into HTML, allowing faster display while remaining CSS loads asynchronously. Angular CLI uses Beasties for analyzing application HTML and styles.
