# Angular Service Workers
> Source: https://angular.dev/ecosystem/service-workers

## What Are Service Workers?

Service workers are browser scripts that manage caching for applications, functioning as network proxies. They intercept HTTP requests and can serve cached responses, enabling offline functionality and improved performance.

## Key Capabilities

Service workers provide several important functions:

- **Network Interception**: They intercept all outgoing HTTP requests from applications, including resources referenced in HTML and the initial `index.html` request
- **Persistent Operation**: Unlike other application scripts, service workers remain active after users close browser tabs
- **Complete Offline Support**: When properly configured, service workers can completely satisfy application loading without network access
- **Latency Reduction**: They minimize dependency on network round-trips, significantly improving user experience

## Angular's Service Worker Implementation

Angular includes a built-in service worker designed to optimize user experience on slow or unreliable connections while minimizing risks of serving outdated content.

### Core Design Principles

Angular's service worker follows these guidelines:

1. **Atomic Caching**: Applications cache as unified units; all files update together
2. **Version Consistency**: Running applications continue with the same file versions, avoiding incompatible cached content from newer versions
3. **Refresh Behavior**: User refreshes display the latest fully cached version; new tabs load current cached code
4. **Background Updates**: Updates occur in the background after deployment, with previous versions served until new versions are ready
5. **Bandwidth Conservation**: Resources only download when content has changed

## Manifest System

Angular service workers use a manifest file called `ngsw.json` that:

- Describes resources to cache
- Includes hashes of every file's contents
- Gets regenerated when applications are deployed
- Is generated from a configuration file named `ngsw-config.json`

## Installation

Installing the Angular service worker is straightforward via running an Angular CLI command. The process registers the service worker with the browser and makes several injectable services available for controlling the service worker and checking for updates.

## Prerequisites

### Browser Requirements

- **HTTPS Required**: Service workers only register on secure connections (HTTPS), except for `localhost` development
- **Browser Support**: Service workers are supported in latest versions of Chrome, Firefox, Edge, Safari, Opera, UC Browser (Android), and Samsung Internet
- **Unsupported Browsers**: IE and Opera Mini do not support service workers

### Graceful Degradation

For unsupported browsers:

- Service worker scripts and `ngsw.json` manifest files are not downloaded
- API calls like `SwUpdate.checkForUpdate()` return rejected promises
- Observable events from related services are not triggered

Developers should verify applications function without service worker support and check `SwUpdate.isEnabled` before interacting with the service worker.

## Important Notice

The Angular Service Worker is a basic caching utility for simple offline support with a limited featureset. Angular will only accept security fixes for this tool. For advanced caching and offline capabilities, exploring native browser APIs directly is recommended.

## Related Resources

- Configuration file documentation
- Service worker communication guides
- Push notification implementation
- Service worker DevOps practices
- App shell pattern documentation

## Next Steps

Developers should proceed to the "Getting Started with service workers" guide to begin implementation.
