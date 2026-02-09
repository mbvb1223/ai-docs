# Adding Event Listeners
> Source: https://angular.dev/guide/templates/event-listeners

## Overview

Angular enables developers to attach event listeners to template elements using parentheses notation `()`. This syntax allows you to specify event names and execute statements whenever those events occur.

## Listening to Native Events

To add event listeners to HTML elements, wrap the event name in parentheses and provide a handler statement:

```typescript
@Component({
  template: `
    <input type="text" (keyup)="updateField()" />
  `,
  ...
})
export class App {
  updateField(): void {
    console.log('Field is updated!');
  }
}
```

Angular automatically calls the `updateField` method each time the `<input>` element emits a `keyup` event. You can listen for any native event, including `click`, `keydown`, `mouseover`, and others available on HTML elements.

## Accessing Event Arguments

Angular provides a special variable named `$event` in template event listeners that contains a reference to the event object:

```typescript
@Component({
  template: `
    <input type="text" (keyup)="updateField($event)" />
  `,
  ...
})
export class App {
  updateField(event: KeyboardEvent): void {
    console.log(`The user pressed: ${event.key}`);
  }
}
```

## Using Key Modifiers

Angular simplifies keyboard event handling by allowing you to filter events by specific keys using period (`.`) notation:

### Basic Key Modifier

Instead of manually checking `event.key === 'Enter'`, you can use:

```typescript
@Component({
  template: `
    <input type="text" (keyup.enter)="updateField($event)" />
  `,
  ...
})
export class App {
  updateField(event: KeyboardEvent): void {
    console.log('The user pressed enter in the text field.');
  }
}
```

### Multiple Modifiers

Combine multiple modifiers using additional period notation:

```html
<!-- Matches shift and enter -->
<input type="text" (keyup.shift.enter)="updateField($event)" />
```

### Supported Modifiers

Angular supports four modifier keys: `alt`, `control`, `meta`, and `shift`.

### Key and Code Values

By default, Angular uses the [Key values for keyboard events](https://developer.mozilla.org/docs/Web/API/UI_Events/Keyboard_event_key_values). You can also specify [Code values](https://developer.mozilla.org/docs/Web/API/UI_Events/Keyboard_event_code_values) using the `code` suffix:

```html
<!-- Matches alt and left shift -->
<input type="text" (keydown.code.alt.shiftleft)="updateField($event)" />
```

This approach ensures consistent keyboard event handling across different operating systems, particularly useful for platforms like MacOS where Alt key modifiers alter character output.

## Listening on Global Targets

Use global target prefixes to listen for events on `window`, `document`, or `body`:

```typescript
@Component({
  /* ... */
  host: {
    'window:click': 'onWindowClick()',
    'document:click': 'onDocumentClick()',
    'body:click': 'onBodyClick()',
  },
})
export class MyView {}
```

## Preventing Default Browser Behavior

When you need to replace native browser behavior, use the event object's `preventDefault()` method:

```typescript
@Component({
  template: `
    <a href="#overlay" (click)="showOverlay($event)">
  `,
  ...
})
export class App {
  showOverlay(event: PointerEvent): void {
    event.preventDefault();
    console.log('Show overlay without updating the URL!');
  }
}
```

Alternatively, if your handler statement evaluates to `false`, Angular automatically calls `preventDefault()`. However, explicit invocation is recommended for code clarity.

## Extending Event Handling

Angular's event system is extensible through custom event plugins registered with the `EVENT_MANAGER_PLUGINS` injection token.

### Creating a Custom Event Plugin

Extend the `EventManagerPlugin` class:

```typescript
import {Injectable} from '@angular/core';
import {EventManagerPlugin} from '@angular/platform-browser';

@Injectable()
export class DebounceEventPlugin extends EventManagerPlugin {
  constructor() {
    super(document);
  }

  override supports(eventName: string) {
    return /debounce/.test(eventName);
  }

  override addEventListener(element: HTMLElement, eventName: string, handler: Function) {
    const [event, method, delay = 300] = eventName.split('.');
    let timeoutId: number;

    const listener = (event: Event) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
        handler(event);
      }, delay);
    };

    element.addEventListener(event, listener);
    return () => {
      clearTimeout(timeoutId);
      element.removeEventListener(event, listener);
    };
  }
}
```

### Registering the Plugin

Use the `EVENT_MANAGER_PLUGINS` token in your application providers:

```typescript
import {bootstrapApplication} from '@angular/platform-browser';
import {EVENT_MANAGER_PLUGINS} from '@angular/platform-browser';
import {App} from './app';
import {DebounceEventPlugin} from './debounce-event-plugin';

bootstrapApplication(App, {
  providers: [
    {
      provide: EVENT_MANAGER_PLUGINS,
      useClass: DebounceEventPlugin,
      multi: true,
    },
  ],
});
```

### Using Custom Event Syntax

Once registered, use the custom syntax in templates:

```typescript
@Component({
  template: `
    <input
      type="text"
      (input.debounce.500)="onSearch($event.target.value)"
      placeholder="Search..."
    />
  `,
  ...
})
export class Search {
  onSearch(query: string): void {
    console.log('Searching for:', query);
  }
}
```

Or with the `host` property:

```typescript
@Component({
  ...,
  host: {
    '(click.debounce.500)': 'handleDebouncedClick()',
  },
})
export class AwesomeCard {
  handleDebouncedClick(): void {
    console.log('Debounced click!');
  }
}
```
