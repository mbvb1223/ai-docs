# Angular Elements Overview
> Source: https://angular.dev/guide/elements

## What Are Angular Elements?

Angular elements are Angular components packaged as custom elements (also called Web Components), providing a framework-agnostic way to define new HTML elements. They extend HTML functionality by allowing you to create tags whose content is managed by JavaScript code.

## Key Concepts

### Custom Elements Registry

The browser maintains a `CustomElementRegistry` that maps instantiable JavaScript classes to HTML tags. This enables standard DOM APIs to work with your custom elements seamlessly.

### The Bridge

The `@angular/elements` package exports a `createCustomElement()` API that bridges Angular's component interface and change detection to the built-in DOM API. This makes Angular infrastructure automatically available to the browser.

## Installation

To add the package to your workspace, use:

```bash
npm install @angular/elements
# or
yarn add @angular/elements
# or
pnpm add @angular/elements
# or
bun add @angular/elements
```

## How Angular Elements Work

### Basic Usage

The `createCustomElement()` function converts a component into a class that registers with the browser as a custom element. Once registered, use it like any HTML element:

```html
<my-popup message="Use Angular!"></my-popup>
```

When placed on a page, the browser instantiates the registered class and adds it to the DOM. The component's template renders using Angular's template syntax, and input properties map to element attributes.

## Transforming Components to Custom Elements

### The Process

1. Angular's `createCustomElement()` converts your component and dependencies to a custom element
2. The conversion implements the `NgElementConstructor` interface
3. Use the browser's native `customElements.define()` to register with `CustomElementRegistry`
4. When the browser encounters the registered tag, it creates a custom element instance

### Critical Warning

Avoid using the component's selector as the custom element tag name. This can lead to unexpected behavior, due to Angular creating two component instances for a single DOM element: One regular Angular component and a second one using the custom element.

## Property and Event Mapping

### Input Properties

The creation API scans your component for input properties and creates corresponding attributes. Property names transform to be case-insensitive using dash-separated lowercase:

- Example: `inputProp = input({alias: 'myInputProp'})` becomes the attribute `my-input-prop`

### Component Outputs

Outputs dispatch as HTML Custom Events matching the output name:

- Example: `valueChanged = output()` dispatches events named "valueChanged"
- The emitted data appears on the event's `detail` property
- With alias: `clicks = output<string>({alias: 'myClick'})` dispatches "myClick" events

## Example: Popup Service

### Files Overview

| File | Purpose |
|------|---------|
| `popup.ts` | Defines a popup component with animation and styling |
| `popup.service.ts` | Provides two methods: dynamic component loading and custom element usage |
| `app.ts` | Root component converting Popup to a custom element |

### popup.ts

```typescript
import {Component, computed, input, output} from '@angular/core';
import {animate, state, style, transition, trigger} from '@angular/animations';

@Component({
  selector: 'my-popup',
  template: `
    <span>Popup: {{ message() }}</span>
    <button type="button" (click)="closed.emit()">&#x2716;</button>
  `,
  animations: [
    trigger('state', [
      state('opened', style({transform: 'translateY(0%)'})),
      state('void, closed', style({transform: 'translateY(100%)', opacity: 0})),
      transition('* => *', animate('100ms ease-in')),
    ]),
  ],
  styles: [
    `
      :host {
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        background: #009cff;
        height: 48px;
        padding: 16px;
        display: flex;
        justify-content: space-between;
        align-items: center;
        border-top: 1px solid black;
        font-size: 24px;
      }
      button {
        border-radius: 50%;
      }
    `,
  ],
  host: {
    '[@state]': 'state()',
  },
})
export class Popup {
  readonly message = input('');
  readonly closed = output<void>();
  readonly state = computed(() => (this.message() ? 'opened' : 'closed'));
}
```

### popup.service.ts

```typescript
import {
  ApplicationRef,
  createComponent,
  EnvironmentInjector,
  inject,
  Injectable,
} from '@angular/core';
import {NgElement, WithProperties} from '@angular/elements';
import {Popup} from './popup';

@Injectable()
export class PopupService {
  private readonly injector = inject(EnvironmentInjector);
  private readonly applicationRef = inject(ApplicationRef);

  // Dynamic-loading method
  showAsComponent(message: string) {
    const popup = document.createElement('popup-component');
    const popupComponentRef = createComponent(Popup, {
      environmentInjector: this.injector,
      hostElement: popup,
    });
    this.applicationRef.attachView(popupComponentRef.hostView);
    popupComponentRef.instance.closed.subscribe(() => {
      document.body.removeChild(popup);
      this.applicationRef.detachView(popupComponentRef.hostView);
    });
    popupComponentRef.setInput('message', message);
    document.body.appendChild(popup);
  }

  // Custom-element method
  showAsElement(message: string) {
    const popupEl: NgElement & WithProperties<Popup> = document.createElement(
      'popup-element',
    ) as any;
    popupEl.addEventListener('closed', () => document.body.removeChild(popupEl));
    popupEl.setAttribute('message', message);
    document.body.appendChild(popupEl);
  }
}
```

### app.ts

```typescript
import {Component, Injector} from '@angular/core';
import {createCustomElement} from '@angular/elements';
import {Popup} from './popup';
import {PopupService} from './popup.service';

@Component({
  selector: 'app-root',
  template: `
    <input #input value="Message" />
    <button type="button" (click)="popup.showAsComponent(input.value)">
      Show as component
    </button>
    <button type="button" (click)="popup.showAsElement(input.value)">
      Show as element
    </button>
  `,
  providers: [PopupService],
  imports: [Popup],
})
export class App {
  constructor(
    injector: Injector,
    public popup: PopupService,
  ) {
    const PopupElement = createCustomElement(Popup, {injector});
    customElements.define('popup-element', PopupElement);
  }
}
```

## Type Safety for Custom Elements

### The Challenge

Generic DOM APIs like `document.createElement()` return appropriate element types, but with unknown custom elements, they return generic `HTMLElement` without type information.

### Solution 1: Casting

Use `NgElement` and `WithProperties` types for casting:

```typescript
const aDialog = document.createElement('my-dialog') as NgElement &
  WithProperties<{content: string}>;
aDialog.content = 'Hello, world!';
```

### Solution 2: HTMLElementTagNameMap

Define custom element types once using global augmentation:

```typescript
declare global {
  interface HTMLElementTagNameMap {
    'my-dialog': NgElement & WithProperties<{content: string}>;
    'my-other-element': NgElement & WithProperties<{foo: 'bar'}>;
  }
}

// Now TypeScript infers correct types automatically:
document.createElement('my-dialog'); // NgElement & WithProperties<{content: string}>
document.querySelector('my-other-element'); // NgElement & WithProperties<{foo: 'bar'}>
```

## Limitations

Care must be taken when destroying and re-attaching custom elements due to issues with the `disconnect()` callback. This can occur when:

- Rendering a component in `ng-if` or `ng-repeat` in AngularJS
- Manually detaching and re-attaching an element to the DOM

## Key Advantages

- Custom elements bootstrap automatically when added to the DOM
- No special Angular knowledge required to use them
- Automatic change detection and data binding
- Framework-agnostic approach compatible with other technologies
- Simpler than dynamic component loading with fewer setup requirements
