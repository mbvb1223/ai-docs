# Angular DevTools
> Source: https://angular.dev/tools/devtools

## What is Angular DevTools?

Angular DevTools is a browser extension providing debugging and profiling capabilities for Angular applications. It's available for both Chrome and Firefox browsers.

## Installation

Install Angular DevTools from:
- **Chrome**: [Chrome Web Store](https://chrome.google.com/webstore/detail/angular-developer-tools/ienfalfjdbdpebioblfackkekamfmbnh)
- **Firefox**: [Firefox Addons](https://addons.mozilla.org/firefox/addon/angular-devtools/)

## Opening DevTools

Access browser DevTools using:
- **Windows/Linux**: F12 or Ctrl+Shift+I
- **Mac**: Fn+F12 or Cmd+Option+I

Once open, locate the "Angular" tab when the extension is installed.

**Note**: Chrome's new tab page does not run installed extensions, so the Angular tab will not appear in DevTools.

## Main Features

Angular DevTools provides three primary tabs:

| Tab | Functionality |
|-----|---------------|
| **Components** | Explore components and directives; preview or edit their state |
| **Profiler** | Profile applications and identify performance bottlenecks during change detection |
| **Injector Tree** | Visualize Environment and Element Injector hierarchy |

Additional experimental tabs (Router Tree, Transfer State) can be enabled via settings.

## Common Issues

### Angular Application Not Detected

This error indicates the extension cannot communicate with an Angular app on the page. Common causes include inspecting a non-Angular webpage or an application that isn't running.

### Production Build Detected

The error message states: "We detected an application built with production configuration. Angular DevTools only supports development builds."

Production builds remove debug features for performance optimization. To use DevTools, compile with optimizations disabled using `ng serve` (default) or set the `optimization` configuration option to `false`.

## Additional Resources

For Chromium-based browser users, Angular also offers Performance panel integration for additional profiling capabilities.
