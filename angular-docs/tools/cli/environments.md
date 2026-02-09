# Build Environments in Angular CLI
> Source: https://angular.dev/tools/cli/environments

## Overview

Angular CLI enables developers to define multiple named build configurations for projects, such as `development` and `staging`, each with distinct default settings. These configurations allow the build, serve, and test commands to substitute files with appropriate versions for target environments.

## Angular CLI Configurations

The `@angular-devkit/build-angular:browser` builder supports a `configurations` object that overrides specific builder options based on command-line specifications.

**Example configuration structure:**

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser",
          "options": {
            "sourceMap": false
          },
          "configurations": {
            "debug": {
              "sourceMap": true
            }
          }
        }
      }
    }
  }
}
```

**Selecting configurations:**

Use the `--configuration` flag to specify which configuration to apply:

```bash
ng build --configuration debug
```

Multiple configurations can be chained with commas, applying sequentially:

```bash
ng build --configuration debug,production,customer-facing
```

## Setting Up Environment-Specific Defaults

### Generate Environment Files

Start by generating the environments directory structure:

```bash
ng generate environments
```

This creates `src/environments/` with:
- `environment.ts` (default/production)
- `environment.development.ts`
- `environment.staging.ts`

**Example directory structure:**

```
my-app/src/environments
├── environment.development.ts
├── environment.staging.ts
└── environment.ts
```

### Configure Base Environment

The default `environment.ts` file:

```typescript
export const environment = {
  production: true,
  apiUrl: 'http://my-prod-url',
};
```

### Create Target-Specific Configurations

For development (`environment.development.ts`):

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://my-dev-url',
};
```

## Using Environment Variables in Components

Components must import from the original environments file:

```typescript
import {environment} from './environments/environment';
```

**Example usage:**

```typescript
import {environment} from './../environments/environment';

// Fetches from different URLs based on build configuration
fetch(environment.apiUrl);
```

## File Replacement Configuration

The `angular.json` file contains `fileReplacements` sections that swap files during builds:

```json
"configurations": {
  "development": {
    "fileReplacements": [
      {
        "replace": "src/environments/environment.ts",
        "with": "src/environments/environment.development.ts"
      }
    ]
  }
}
```

When running `ng build --configuration development`, the base environment file is replaced with the development-specific version.

### Adding Staging Environment

Create `src/environments/environment.staging.ts`, then add to `angular.json`:

```json
"configurations": {
  "development": { },
  "production": { },
  "staging": {
    "fileReplacements": [
      {
        "replace": "src/environments/environment.ts",
        "with": "src/environments/environment.staging.ts"
      }
    ]
  }
}
```

Build with staging configuration:

```bash
ng build --configuration staging
```

## Configuring ng serve

Configure the serve command to use specific build targets:

```json
"serve": {
  "builder": "@angular-devkit/build-angular:dev-server",
  "options": { },
  "configurations": {
    "development": {
      "buildTarget": "my-app:build:development"
    },
    "production": {
      "buildTarget": "my-app:build:production"
    }
  },
  "defaultConfiguration": "development"
}
```

The `defaultConfiguration` property specifies which configuration applies when none is explicitly selected.
