# Serving Angular Apps for Development
> Source: https://angular.dev/tools/cli/serve

## Overview

The `ng serve` command compiles and serves your Angular CLI application locally. It automatically rebuilds and live reloads on file changes. Stop the server with `Ctrl+C`.

The command executes the `serve` target builder specified in `angular.json`. The default builder is `@angular/build:dev-server`.

## Builder Configuration

Check your project's `serve` target in `angular.json`:

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "serve": {
          "builder": "@angular/build:dev-server"
        },
        "build": { },
        "test": { }
      }
    }
  }
}
```

## Proxying to a Backend Server

Use proxy configuration to redirect specific URLs to a backend server via the `--proxy-config` option.

### Setup Steps

1. Create `proxy.conf.json` in your project's `src/` folder

2. Add proxy rules:

```json
{
  "/api/**": {
    "target": "http://localhost:3000",
    "secure": false
  }
}
```

3. Update `angular.json`:

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "serve": {
          "builder": "@angular/build:dev-server",
          "options": {
            "proxyConfig": "src/proxy.conf.json"
          }
        }
      }
    }
  }
}
```

4. Run `ng serve`

**Important:** Restart `ng serve` to apply proxy configuration changes.

## Path Matching Behavior

### @angular/build:dev-server (Vite-based)

- `/api` matches only `/api`
- `/api/*` matches `/api/users` but not `/api/users/123`
- `/api/**` matches `/api/users` and `/api/users/123`

### @angular-devkit/build-angular:dev-server (Webpack-based)

- `/api` matches `/api` and all sub-paths (equivalent to `/api/**`)
