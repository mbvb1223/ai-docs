# Route Transition Animations
> Source: https://angular.dev/guide/animations/route-animations
> (Redirects to: https://angular.dev/guide/routing/route-transition-animations)

## Overview

Route transition animations enhance user experience by providing smooth visual transitions when navigating between different views in Angular applications. The Angular Router includes built-in support for the browser's View Transitions API, enabling seamless animations between route changes in supported browsers.

> **Note:** The Router's native View Transitions integration is currently in developer preview and represents a relatively new browser feature with limited cross-browser support.

## How View Transitions Work

View transitions leverage the browser's native `document.startViewTransition` API to create smooth animations between different application states through a four-step process:

1. **Capturing the current state** -- The browser takes a screenshot of the current page
2. **Executing the DOM update** -- Your callback function runs to update the DOM
3. **Capturing the new state** -- The browser captures the updated page state
4. **Playing the transition** -- The browser animates between the old and new states

Basic API structure:

```javascript
document.startViewTransition(async () => {
  await updateTheDOMSomehow();
});
```

For additional details, consult the [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions).

## Router Integration with View Transitions

Angular Router integrates view transitions into the navigation lifecycle by:

1. **Completing navigation preparation** -- Route matching, lazy loading, guards, and resolvers execute
2. **Initiating the view transition** -- Router calls `startViewTransition` when routes are ready for activation
3. **Updating the DOM** -- Router activates new routes and deactivates old ones within the transition callback
4. **Finalizing the transition** -- The transition Promise resolves when Angular completes rendering

The Router's view transition integration functions as progressive enhancement. When browsers lack View Transitions API support, the Router performs normal DOM updates without animation, ensuring cross-browser compatibility.

## Enabling View Transitions in the Router

Add the `withViewTransitions` feature to your router configuration. Angular supports both standalone and NgModule bootstrap approaches.

### Standalone Bootstrap

```typescript
import {bootstrapApplication} from '@angular/platform-browser';
import {provideRouter, withViewTransitions} from '@angular/router';
import {routes} from './app.routes';

bootstrapApplication(MyApp, {
  providers: [provideRouter(routes, withViewTransitions())],
});
```

### NgModule Bootstrap

```typescript
import {NgModule} from '@angular/core';
import {RouterModule} from '@angular/router';

@NgModule({
  imports: [RouterModule.forRoot(routes, {enableViewTransitions: true})],
})
export class AppRouting {}
```

## Customizing Transitions with CSS

Customize view transitions using CSS to create unique animation effects. The browser creates separate transition elements that you can target with CSS selectors.

To create custom transitions:

1. **Add view-transition-name** -- Assign unique names to elements you want to animate
2. **Define global animations** -- Create CSS animations in your global styles
3. **Target transition pseudo-elements** -- Use `::view-transition-old()` and `::view-transition-new()` selectors

Example adding rotation effect to a counter element:

```css
/* Define keyframe animations */
@keyframes rotate-out {
  to {
    transform: rotate(90deg);
  }
}

@keyframes rotate-in {
  from {
    transform: rotate(-90deg);
  }
}

/* Target view transition pseudo-elements */
::view-transition-old(count),
::view-transition-new(count) {
  animation-duration: 200ms;
  animation-name: -ua-view-transition-fade-in, rotate-in;
}

::view-transition-old(count) {
  animation-name: -ua-view-transition-fade-out, rotate-out;
}
```

> **Important:** Define view transition animations in global styles files, not in component styles. Angular's view encapsulation scopes component styles, preventing them from targeting transition pseudo-elements correctly.

## Advanced Transition Control with onViewTransitionCreated

The `withViewTransitions` feature accepts an options object with an `onViewTransitionCreated` callback for advanced control. This callback:

- Runs in an injection context
- Receives a `ViewTransitionInfo` object containing the `ViewTransition` instance, the `ActivatedRouteSnapshot` for the route being navigated from, and the `ActivatedRouteSnapshot` for the route being navigated to

Use this callback to customize transition behavior based on navigation context. Example skipping transitions for specific navigation types:

```typescript
import {inject} from '@angular/core';
import {Router, withViewTransitions, isActive} from '@angular/router';

withViewTransitions({
  onViewTransitionCreated: ({transition}) => {
    const router = inject(Router);
    const targetUrl = router.currentNavigation()!.finalUrl!;

    // Skip transition if only fragment or query params change
    const config = {
      paths: 'exact',
      matrixParams: 'exact',
      fragment: 'ignored',
      queryParams: 'ignored',
    };
    const isTargetRouteCurrent = isActive(targetUrl, router, config);

    if (isTargetRouteCurrent()) {
      transition.skipTransition();
    }
  },
});
```

This example skips the view transition when navigation only changes the URL fragment or query parameters, such as anchor links within the same page.

## Examples from Chrome Explainer Adapted to Angular

### Transitioning Elements Don't Need to Be the Same DOM Element

Elements can transition smoothly between different DOM elements as long as they share the same `view-transition-name`.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#transitioning_elements_dont_need_to_be_the_same_dom_element)
- [Angular Example on StackBlitz](https://stackblitz.com/edit/stackblitz-starters-dh8npr?file=src%2Fmain.ts)

### Custom Entry and Exit Animations

Create unique animations for elements entering and leaving the viewport during route transitions.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#custom_entry_and_exit_transitions)
- [Angular Example on StackBlitz](https://stackblitz.com/edit/stackblitz-starters-8kly3o)

### Async DOM Updates and Waiting for Content

Angular Router prioritizes immediate transitions over waiting for additional content to load.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#async_dom_updates_and_waiting_for_content)

> **Note:** Angular Router does not provide a way to delay view transitions. This design choice prevents pages from becoming non-interactive while waiting for additional content. As Chrome documentation notes: "During this time, the page is frozen, so delays here should be kept to a minimum."

### Handle Multiple View Transition Styles with View Transition Types

Use view transition types to apply different animation styles based on navigation context.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#view-transition-types)
- [Angular Example on StackBlitz](https://stackblitz.com/edit/stackblitz-starters-vxzcam)

### Handle Multiple View Transition Styles with CSS Class (Deprecated)

This approach uses CSS classes on the transition root element to control animation styles.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#changing-on-navigation-type)
- [Angular Example on StackBlitz](https://stackblitz.com/edit/stackblitz-starters-nmnzzg?file=src%2Fmain.ts)

### Transitioning Without Freezing Other Animations

Maintain other page animations during view transitions to create more dynamic user experiences.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#transitioning-without-freezing)
- [Angular Example on StackBlitz](https://stackblitz.com/edit/stackblitz-starters-76kgww)

### Animating with JavaScript

Control view transitions programmatically using JavaScript APIs for complex animation scenarios.

- [Chrome Explainer](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#animating-with-javascript)
- [Angular Example on StackBlitz](https://stackblitz.com/edit/stackblitz-starters-cklnkm)
