# Testing Pipes
> Source: https://angular.dev/guide/testing/pipes

## Overview

Pipes can be tested without Angular's testing utilities since they're pure, stateless functions with a single `transform` method that rarely interacts with the DOM.

## Testing the TitleCasePipe

### Implementation Example

A `TitleCasePipe` capitalizes the first letter of each word using a regular expression:

```typescript
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({name: 'titlecase', pure: true})
/** Transform to Title Case: uppercase the first letter of the words in a string. */
export class TitleCasePipe implements PipeTransform {
  transform(input: string): string {
    return input.length === 0
      ? ''
      : input.replace(/\w\S*/g, (txt) => txt[0].toUpperCase() + txt.slice(1).toLowerCase());
  }
}
```

### Unit Testing Approach

Since regex patterns warrant thorough testing, use standard unit testing techniques to verify expected cases and edge cases:

```typescript
describe('TitleCasePipe', () => {
  // This pipe is a pure, stateless function so no need for BeforeEach
  const pipe = new TitleCasePipe();

  it('transforms "abc" to "Abc"', () => {
    expect(pipe.transform('abc')).toBe('Abc');
  });

  it('transforms "abc def" to "Abc Def"', () => {
    expect(pipe.transform('abc def')).toBe('Abc Def');
  });

  // ... more tests ...
});
```

## Writing DOM Tests for Pipe Integration

Isolated pipe tests cannot verify whether the pipe functions correctly within application components. Consider adding component-level tests:

```typescript
it('should convert hero name to Title Case', async () => {
  // get the name's input and display elements from the DOM
  const hostElement: HTMLElement = harness.routeNativeElement!;
  const nameInput: HTMLInputElement = hostElement.querySelector('input')!;
  const nameDisplay: HTMLElement = hostElement.querySelector('span')!;

  // simulate user entering a new name into the input box
  nameInput.value = 'quick BROWN  fOx';

  // Dispatch a DOM event so that Angular learns of input value change.
  nameInput.dispatchEvent(new Event('input'));

  // Wait for Angular to update the display binding through the title pipe
  await harness.fixture.whenStable();

  expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
});
```

## Key Testing Principles

- **Isolation Testing**: Test pipe logic independently without Angular utilities
- **Edge Cases**: Thoroughly test regex implementations with various inputs
- **Integration Testing**: Verify pipe behavior within actual component contexts
- **Event Handling**: Use `dispatchEvent()` to trigger Angular change detection
- **Async Stability**: Use `whenStable()` to wait for binding updates
