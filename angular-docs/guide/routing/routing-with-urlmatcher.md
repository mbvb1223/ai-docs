# Creating Custom Route Matches
> Source: https://angular.dev/guide/routing/routing-with-urlmatcher

## Overview

Angular's Router supports sophisticated pattern matching capabilities for handling dynamic URLs. Beyond static routes, variable routes with parameters, and wildcard routes, developers can implement custom pattern matching using Angular's `UrlMatcher` interface for complex URL scenarios.

This guide demonstrates building a custom route matcher that identifies Twitter handles preceded by an `@` symbol in the URL.

## Objectives

Implement Angular's `UrlMatcher` to create a custom route matcher that recognizes specific URL patterns.

## Project Setup

### Step 1: Create Application

Using Angular CLI, generate a new project called `angular-custom-route-match`:

```bash
ng new angular-custom-route-match
```

When prompted, select `Y` for Angular routing and `CSS` for stylesheets.

### Step 2: Generate Profile Component

```bash
ng generate component profile
```

### Step 3: Update Templates

Update `profile.html`:

```html
<p>Hello {{ username() }}!</p>
```

Update `app.html`:

```html
<h2>Routing with Custom Matching</h2>
Navigate to <a routerLink="/@Angular">my profile</a>
<router-outlet />
```

## Configure Routes with Custom Matcher

### Import Required Modules

In `app.config.ts`, add imports:

```typescript
import {provideRouter, withComponentInputBinding} from '@angular/router';
import {routes} from './app.routes';
```

Add to providers array:

```typescript
provideRouter(routes, withComponentInputBinding())
```

### Define Custom URL Matcher

In `app.routes.ts`, implement the matcher:

```typescript
{
  matcher: (url) => {
    if (url.length === 1 && url[0].path.match(/^@[\w]+$/gm)) {
      return {
        consumed: url,
        posParams: {
          username: new UrlSegment(url[0].path.slice(1), {})
        }
      };
    }
    return null;
  },
  component: Profile,
}
```

### How the Matcher Works

The custom matcher function performs these tasks:

- Verifies the URL contains exactly one segment
- Uses regex to validate the username format (starting with `@` followed by word characters)
- Returns the complete URL with a `username` parameter extracted from the path (removing the `@` prefix)
- Returns `null` if no match occurs, allowing the router to continue evaluating other routes

**Note:** A custom URL matcher behaves like any other route definition. Define child routes or lazy loaded routes as you would with any other route.

## Reading Route Parameters

In `profile.ts`, create an input property to receive the username parameter:

```typescript
username = input.required<string>();
```

The `withComponentInputBinding` feature enables the Router to bind route information directly to component inputs.

## Testing the Implementation

### Run Development Server

```bash
ng serve
```

### Verify Functionality

1. Open `http://localhost:4200` in your browser
2. Observe the message "Navigate to my profile"
3. Click the profile link
4. The page displays "Hello, Angular!"

## Key Concepts

**UrlMatcher:** A function that returns a `UrlMatchResult` object when a URL pattern matches, or `null` when it does not. This allows custom pattern recognition beyond standard route definitions.

**UrlSegment:** Represents individual segments of a URL path, containing the path string and matrix parameters.

**withComponentInputBinding:** A feature that automatically maps route parameters to component input properties, eliminating manual subscription requirements.

## Next Steps

- Explore [Router API Reference](/api/router/Router)
- Learn [Common Router Tasks](/guide/routing/common-router-tasks)
