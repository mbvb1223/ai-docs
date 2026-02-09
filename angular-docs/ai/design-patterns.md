# Design Patterns for AI SDKs and Signal APIs
> Source: https://angular.dev/ai/design-patterns

## Overview

Angular's signals and resource API provide tools for managing asynchronous operations, handling streaming data, and creating responsive user experiences when working with AI and Large Language Model (LLM) APIs.

## Key Design Patterns

### Triggering Requests with Signals

Separate user input from submission to control when API calls execute:

1. Store raw user input in one signal as they type
2. Create a second signal updated only on user submission
3. Use the second signal in the `params` field of a `resource`

This ensures the loader function executes only on explicit user action, not on every keystroke.

**Example Implementation:**

```typescript
storyResource = resource({
  defaultValue: DEFAULT_STORY,
  params: () => this.storyInput(),
  loader: ({params}): Promise<StoryData> => {
    const url = this.endpoint();
    return runFlow({
      url,
      input: {
        userInput: params,
        sessionId: this.storyService.sessionId(),
      },
    });
  },
});
```

Additional signal parameters like `sessionId` or `userId` can be read from other signals without re-triggering the loader.

### Preparing LLM Data for Templates

Configure LLMs to return structured data with strong typing for better type safety and editor autocompletion.

Use `computed` or `linkedSignal` to manage state derived from resources. `linkedSignal` is particularly useful because it provides access to previous values, enabling:

- Building chat history
- Preserving data during LLM generation

**Example with linkedSignal:**

```typescript
storyParts = linkedSignal<string[], string[]>({
  source: () => this.storyResource.value().storyParts,
  computation: (newStoryParts, previous) => {
    const existingStoryParts = previous?.value || [];
    return [...existingStoryParts, ...newStoryParts];
  },
});
```

## Performance and User Experience

### Optimization Strategies

**Scoped Loading:** Place resources in components that directly use the data to limit change detection cycles and prevent blocking other application sections.

**SSR and Hydration:** Use Server-Side Rendering with incremental hydration to render initial content quickly, deferring AI-generated content until client hydration.

**Loading State:** Leverage the `LOADING` status to display spinners or indicators during requests.

**Error Handling and Retries:** Use the `reload()` method to allow users to retry failed requests.

**Template Example:**

```html
@if (imgResource.isLoading()) {
  <div class="img-placeholder">
    <mat-spinner [diameter]="50" />
  </div>
} @else if (imgResource.hasValue()) {
  <img [src]="imgResource.value()" />
} @else {
  <div class="img-placeholder" (click)="imgResource.reload()">
    <mat-icon fontIcon="refresh" />
    <p>Failed to load image. Click to retry.</p>
  </div>
}
```

## Streaming Chat Responses

The `stream` property accepts an asynchronous function that updates a signal value over time, supporting incremental display of LLM responses:

```typescript
characters = resource({
  stream: async () => {
    const data = signal<ResourceStreamItem<string>>({value: ''});
    const response = streamFlow({
      url: '/streamCharacters',
      input: 10,
    });
    (async () => {
      for await (const chunk of response.stream) {
        data.update((prev) => {
          if ('value' in prev) {
            return {value: `${prev.value} ${chunk}`};
          } else {
            return {error: chunk as unknown as Error};
          }
        });
      }
    })();
    return data;
  },
});
```

**Template Display:**

```html
@if (characters.isLoading()) {
  <p>Loading...</p>
} @else if (characters.hasValue()) {
  <p>{{ characters.value() }}</p>
} @else {
  <p>{{ characters.error() }}</p>
}
```

### Server-Side Implementation

Use frameworks like Genkit with streaming support:

```typescript
import {startFlowServer} from '@genkit-ai/express';
import {genkit} from 'genkit/beta';
import {googleAI, gemini20Flash} from '@genkit-ai/googleai';

const ai = genkit({plugins: [googleAI()]});

export const streamCharacters = ai.defineFlow(
  {
    name: 'streamCharacters',
    inputSchema: z.number(),
    outputSchema: z.string(),
    streamSchema: z.string(),
  },
  async (count, {sendChunk}) => {
    const {response, stream} = ai.generateStream({
      model: gemini20Flash,
      config: {temperature: 1},
      prompt: `Generate ${count} different RPG game characters.`,
    });
    (async () => {
      for await (const chunk of stream) {
        sendChunk(chunk.content[0].text!);
      }
    })();
    return (await response).text;
  },
);

startFlowServer({flows: [streamCharacters]});
```

## Summary

These patterns leverage Angular's signal-based reactive system to create performant, user-friendly AI-powered applications that gracefully handle asynchronous operations and streaming data.
