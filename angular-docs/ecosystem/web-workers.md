# Background Processing Using Web Workers
> Source: https://angular.dev/ecosystem/web-workers

## Overview

Web workers enable CPU-intensive computations to run in background threads, keeping the main thread available for UI updates. Applications performing heavy calculations -- such as CAD drawings or geometric computations -- can leverage web workers to enhance performance.

**Note:** The Angular CLI does not support running itself within a web worker context.

## Adding a Web Worker

### Generation Command

To integrate a web worker into an existing Angular project, use the Angular CLI:

```bash
ng generate web-worker <location>
```

### Example Implementation

To add a web worker to the root component (`src/app/app.component.ts`):

```bash
ng generate web-worker app
```

### What the Command Does

The generation command performs three key actions:

1. **Configures the project** to support web workers (if not already configured)

2. **Creates worker scaffold** in `src/app/app.worker.ts`:

```typescript
addEventListener('message', ({data}) => {
  const response = `worker response to ${data}`;
  postMessage(response);
});
```

3. **Creates component scaffold** in `src/app/app.component.ts`:

```typescript
if (typeof Worker !== 'undefined') {
  const worker = new Worker(new URL('./app.worker', import.meta.url));
  worker.onmessage = ({data}) => {
    console.log(`page got message: ${data}`);
  };
  worker.postMessage('hello');
} else {
  // Web workers are not supported in this environment.
  // You should add a fallback so that your program still executes correctly.
}
```

## Implementation Requirements

After scaffolding, refactor your code to establish bidirectional messaging between the main thread and worker thread.

## Environmental Considerations

**Critical:** Some platforms like `@angular/platform-server` (used for server-side rendering) lack web worker support.

To ensure cross-environment compatibility, implement fallback mechanisms that perform worker computations directly in unsupported environments.
