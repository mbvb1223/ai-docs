# DI in Action
> Source: https://angular.dev/guide/di/di-in-action

This guide explores additional features of dependency injection in Angular, building on the foundational concepts of DI.

**Note:** For comprehensive coverage of InjectionToken and custom providers, consult the [defining dependency providers guide](providers.md).

## Inject the Component's DOM Element

Although developers strive to avoid it, certain visual effects and third-party tools require direct DOM access. Angular exposes the underlying element of a `@Component` or `@Directive` via the `ElementRef` injection token.

### Example: HighlightDirective

```typescript
import {Directive, ElementRef, inject} from '@angular/core';

@Directive({
  selector: '[appHighlight]',
})
export class HighlightDirective {
  private element = inject(ElementRef);

  update() {
    this.element.nativeElement.style.color = 'red';
  }
}
```

The `ElementRef` provides access to `nativeElement`, enabling direct manipulation of the host DOM element's properties and styles.

## Inject the Host Element's Tag Name

When you need the tag name of a host element, use the `HOST_TAG_NAME` token for injection.

### Example: RoleButtonDirective

```typescript
import {Directive, HOST_TAG_NAME, inject} from '@angular/core';

@Directive({
  selector: '[roleButton]',
})
export class RoleButtonDirective {
  private tagName = inject(HOST_TAG_NAME);

  onAction() {
    switch (this.tagName) {
      case 'button':
        // Handle button action
        break;
      case 'a':
        // Handle anchor action
        break;
      default:
        // Handle other elements
        break;
    }
  }
}
```

**Important:** If the host element might lack a tag name (such as `ng-container` or `ng-template`), make the injection optional to prevent errors.

## Resolve Circular Dependencies with a Forward Reference

TypeScript class declaration order matters -- you cannot reference a class until it is defined. While adhering to the "one class per file" convention minimizes this issue, circular references sometimes become unavoidable.

The Angular `forwardRef()` function creates an indirect reference that Angular resolves later, breaking circular dependency chains.

### Example: Using forwardRef()

```typescript
providers: [
  {
    provide: PARENT_MENU_ITEM,
    useExisting: forwardRef(() => MenuItem),
  },
]
```

This pattern is especially useful when a class references itself in its `providers` array, since the `@Component()` decorator must appear before the class definition.

## Key Takeaways

- **ElementRef** enables direct DOM access when necessary for visual effects or third-party integrations
- **HOST_TAG_NAME** allows directives to determine and react to their host element's tag type
- **forwardRef()** resolves circular dependencies by deferring class reference resolution
