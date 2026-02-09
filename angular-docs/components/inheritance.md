# Component Inheritance
> Source: https://angular.dev/guide/components/inheritance

## Overview

Angular components are TypeScript classes that participate in standard JavaScript inheritance semantics. Components can extend any base class to inherit functionality and metadata.

## Basic Component Extension

A component can extend another class:

```typescript
export class ListboxBase {
  value: string;
}

@Component({
  /*...*/
})
export class CustomListbox extends ListboxBase {
  // CustomListbox inherits the `value` property.
}
```

## Extending Other Components and Directives

When a component extends another component or directive, it inherits metadata from the base class decorator and decorated members, including:

- Host bindings
- Inputs
- Outputs
- Lifecycle methods

### Example

```typescript
@Component({
  selector: 'base-listbox',
  template: ` ... `,
  host: {
    '(keydown)': 'handleKey($event)',
  },
})
export class ListboxBase {
  value = input.required<string>();
  handleKey(event: KeyboardEvent) {
    /* ... */
  }
}

@Component({
  selector: 'custom-listbox',
  template: ` ... `,
  host: {
    '(click)': 'focusActiveOption()',
  },
})
export class CustomListbox extends ListboxBase {
  disabled = input(false);
  focusActiveOption() {
    /* ... */
  }
}
```

In this example, `CustomListbox` inherits all information from `ListboxBase` while overriding the selector and template. The child class has two inputs (`value` and `disabled`) and two event listeners (`keydown` and `click`).

**Key point:** Child classes end up with the union of all of their ancestors' inputs, outputs, and host bindings and their own.

## Forwarding Injected Dependencies

If a base class injects dependencies as constructor parameters, the child class must explicitly pass these dependencies to `super`:

```typescript
@Component({
  /*...*/
})
export class ListboxBase {
  constructor(private element: ElementRef) {}
}

@Component({
  /*...*/
})
export class CustomListbox extends ListboxBase {
  constructor(element: ElementRef) {
    super(element);
  }
}
```

## Overriding Lifecycle Methods

When a child class implements a lifecycle method like `ngOnInit` that exists in the base class, it overrides the base implementation. To preserve the base class's lifecycle method, explicitly call it with `super`:

```typescript
@Component({
  /*...*/
})
export class ListboxBase {
  protected isInitialized = false;
  ngOnInit() {
    this.isInitialized = true;
  }
}

@Component({
  /*...*/
})
export class CustomListbox extends ListboxBase {
  override ngOnInit() {
    super.ngOnInit();
    /* ... */
  }
}
```

This approach ensures both the base and child class initialization logic executes properly.
