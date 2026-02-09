# Custom Events with Outputs
> Source: https://angular.dev/guide/components/outputs

## Overview

Angular components can define custom events using the `output` function. This guide covers implementing component outputs for raising custom events, similar to native browser events like `click`.

## Basic Output Definition

```typescript
@Component({/*...*/})
export class ExpandablePanel {
  panelClosed = output<void>();
}
```

```html
<expandable-panel (panelClosed)="savePanelState()" />
```

The `output` function returns an `OutputEmitterRef`, which has an `emit` method for triggering events:

```typescript
this.panelClosed.emit();
```

## Key Characteristics

- **No DOM Bubbling**: Angular custom events do not bubble up the DOM
- **Case Sensitivity**: Output names are case-sensitive
- **Inheritance**: Outputs are inherited by child component classes
- **Compiler Requirement**: The `output` function can only be called in component and directive property initializers

## Emitting Event Data

Pass data when emitting events:

```typescript
this.valueChanged.emit(7);

this.thumbDropped.emit({
  pointerX: 123,
  pointerY: 456,
});
```

Access event data in templates using the `$event` variable:

```html
<custom-slider (valueChanged)="logValue($event)" />
```

Parent component receives the data:

```typescript
@Component({/*...*/})
export class App {
  logValue(value: number) { ... }
}
```

## Customizing Output Names

Use the `alias` parameter to specify a different event name in templates:

```typescript
@Component({/*...*/})
export class CustomSlider {
  changed = output({alias: 'valueChanged'});
}
```

```html
<custom-slider (valueChanged)="saveVolume()" />
```

The alias doesn't affect TypeScript property usage.

## Programmatic Subscription

Subscribe to outputs from dynamically created components:

```typescript
const someComponentRef: ComponentRef<SomeComponent> =
  viewContainerRef.createComponent(/*...*/);

someComponentRef.instance.someEventProperty.subscribe((eventData) => {
  console.log(eventData);
});
```

Manual unsubscription:

```typescript
const eventSubscription = someComponent.someEventProperty.subscribe((eventData) => {
  console.log(eventData);
});

// ...
eventSubscription.unsubscribe();
```

Angular automatically cleans up subscriptions when destroying components with subscribers.

## Naming Best Practices

- Avoid collision with DOM element events (HTMLElement)
- Don't prefix outputs like component selectors
- Use camelCase consistently
- Don't prefix with "on"

## Legacy @Output Decorator Approach

The decorator-based API remains fully supported:

```typescript
@Component({/*...*/})
export class ExpandablePanel {
  @Output() panelClosed = new EventEmitter<void>();
}
```

Emit events by calling the `emit` method on `EventEmitter`.

### Decorator Aliases

```typescript
@Component({/*...*/})
export class CustomSlider {
  @Output('valueChanged') changed = new EventEmitter<number>();
}
```

## Declaring Outputs via @Component Decorator

Specify outputs in the `@Component` decorator, useful when inheriting properties:

```typescript
@Component({
  /*...*/
  outputs: ['valueChanged'],
})
export class CustomSlider extends BaseSlider {}
```

Specify aliases with a colon:

```typescript
@Component({
  /*...*/
  outputs: ['valueChanged: volumeChanged'],
})
export class CustomSlider extends BaseSlider {}
```

## RxJS Interoperability

For details on using outputs with RxJS, refer to the documentation on "RxJS interop with component and directive outputs."
