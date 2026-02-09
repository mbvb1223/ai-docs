# Angular Routing
> Source: https://angular.dev/guide/routing

## Overview

Angular Routing enables developers to manage navigation in single-page applications (SPAs) by controlling which content displays based on the URL without requiring full-page reloads.

## Core Purpose

Routing helps you change what the user sees in a single-page app. Angular Router (`@angular/router`) is the official navigation library, included by default in all Angular CLI projects.

## Why Routing is Necessary in SPAs

### Traditional Web Navigation

In conventional web applications, navigating to a new URL triggers a server request, which returns an HTML page that replaces the entire current page.

### SPA Approach

Single-page applications operate differently:

- The browser makes only one initial request to the web server for `index.html`
- A client-side router subsequently controls content display based on URL changes
- Navigation updates page content in place without full-page reloads
- Users experience faster navigation and reduced server load

## How Angular Manages Routing

Angular routing comprises three primary components:

### 1. Routes

Routes define which component displays when users visit specific URLs. They establish the mapping between URL paths and corresponding components.

### 2. Outlets

Outlets function as placeholders in templates that dynamically load and render components based on the currently active route.

### 3. Links

Links provide navigation mechanisms enabling users to move between different routes without triggering page reloads.

## Additional Routing Functionality

Angular Router offers comprehensive features beyond basic navigation:

- Nested routes for hierarchical navigation structures
- Programmatic navigation triggered by application logic
- Route parameters, query strings, and wildcard patterns
- Activated route information via `ActivatedRoute`
- View transition effects for enhanced UX
- Navigation guards for access control and route protection

## Next Steps

For implementation details, developers should review the guide on defining routes using Angular Router.
