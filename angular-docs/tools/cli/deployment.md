# Angular Deployment Guide
> Source: https://angular.dev/tools/cli/deployment

## Overview

When your Angular application is ready for production, you have multiple deployment options available through the Angular CLI and various third-party platforms.

## Automatic Deployment with the CLI

The `ng deploy` command executes a deployment CLI builder associated with your project. Third-party builders implement deployment capabilities to different platforms, which you can add via `ng add`.

### Firebase Deployment Example

```bash
ng add @angular/fire
ng deploy
```

This interactive command prompts authentication and Firebase project selection before building and uploading production assets.

### Supported Deployment Platforms

| Platform | Setup Command |
|----------|---------------|
| Firebase Hosting | `ng add @angular/fire` |
| Vercel | `vercel init angular` |
| Netlify | `ng add @netlify-builder/deploy` |
| GitHub Pages | `ng add angular-cli-ghpages` |
| Amazon S3 | `ng add @jefiozie/ngx-aws-deploy` |

For self-managed servers without existing builders, you can either create a custom builder or manually deploy your application.

## Manual Deployment to Remote Servers

### Build Process

Create a production build using the default production configuration:

```bash
ng build
```

By default, `ng build` outputs artifacts to `dist/my-app/`, though this is configurable via the `outputPath` option in the `@angular-devkit/build-angular:browser` builder. Copy this directory to your web server or CDN.

## Server Configuration Requirements

### Single-Page Application (SPA) Fallback

Client-side rendered Angular applications require specific server configuration. For applications using Angular Router, configure your server to return `index.html` when requested files don't exist.

**Why this matters:** Deep links like `http://my-app.test/users/42` bypass Angular's client-side router when directly accessed or refreshed. Without fallback configuration, servers return 404 errors instead of loading the application.

**Implementation:** Configure your server's 404 page or fallback route to serve `index.html`. Once loaded, Angular Router reads the URL and displays the correct route.

### Cross-Origin Resource Sharing (CORS)

Network requests to different servers trigger CORS restrictions enforced by browsers. The server receiving requests must explicitly permit them -- the client application cannot override this. Configure your server per [enable-cors.org](https://enable-cors.org/server.html) guidelines.

## Production Optimizations

The production configuration enables automatic optimizations:

| Feature | Purpose |
|---------|---------|
| Ahead-of-Time (AOT) Compilation | Pre-compiles component templates |
| Production Mode | Optimizes runtime performance |
| Bundling | Concatenates files into minimum deployable set |
| Minification | Removes whitespace, comments, optional tokens |
| Mangling | Renames identifiers to shorter arbitrary names |
| Dead Code Elimination | Removes unreferenced modules and unused code |

### Development-Only Code Removal

When serving locally with `ng serve`, Angular enables development features:

- Extra safety checks like `expression-changed-after-checked` detection
- Detailed error messages
- Debugging utilities (global `ng` variable, Angular DevTools support)

The production build removes this development-only code automatically to optimize bundle size for end users.

## Deploy URL Configuration

The `--deploy-url` flag specifies the base path for resolving relative asset URLs at compile time:

```bash
ng build --deploy-url /my/assets
```

This option overlaps functionally with `<base href>`, which can be defined at runtime. Prefer `<base href>` where possible, as `--deploy-url` requires hard-coding at build time.
