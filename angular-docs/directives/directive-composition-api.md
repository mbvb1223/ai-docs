# Directive Composition API
> Source: https://angular.dev/guide/directives/directive-composition-api

## Overview

Angular's directive composition API enables developers to apply directives to a component's host element directly from the component's TypeScript class, rather than in templates. This approach encapsulates reusable behaviors including attributes, CSS classes, and event listeners.

## Adding Directives to a Component

Directives are applied through the `hostDirectives` property in a component's decorator:

```typescript
@Component({
  selector: 'admin-menu',
  template: 'admin-menu.html',
  hostDirectives: [MenuBehavior],
})
export class AdminMenu {}
```

When Angular renders the component, it instantiates each host directive and applies their bindings to the component's host element. By default, host directive inputs and outputs remain hidden from the component's public API.

**Key Constraints:**
- Angular applies host directives statically at compile time. You cannot dynamically add directives at runtime.
- Directives in `hostDirectives` cannot specify `standalone: false`.
- Angular ignores directive selectors when applied via `hostDirectives`.

## Including Inputs and Outputs

To expose host directive inputs and outputs as part of your component's API, expand the `hostDirectives` entry:

```typescript
@Component({
  selector: 'admin-menu',
  template: 'admin-menu.html',
  hostDirectives: [
    {
      directive: MenuBehavior,
      inputs: ['menuId'],
      outputs: ['menuClosed'],
    },
  ],
})
export class AdminMenu {}
```

This allows consumers to bind these properties:

```html
<admin-menu menuId="top-menu" (menuClosed)="logMenuClosed()"></admin-menu>
```

### Aliasing Inputs and Outputs

Customize your component's API by aliasing directive inputs and outputs:

```typescript
@Component({
  selector: 'admin-menu',
  template: 'admin-menu.html',
  hostDirectives: [
    {
      directive: MenuBehavior,
      inputs: ['menuId: id'],
      outputs: ['menuClosed: closed'],
    },
  ],
})
export class AdminMenu {}
```

```html
<admin-menu id="top-menu" (closed)="logMenuClosed()"></admin-menu>
```

## Adding Directives to Another Directive

`hostDirectives` can be applied to directives as well, enabling transitive composition:

```typescript
@Directive({...})
export class Menu { }

@Directive({...})
export class Tooltip { }

@Directive({
  hostDirectives: [Tooltip, Menu],
})
export class MenuWithTooltip { }

@Directive({
  hostDirectives: [MenuWithTooltip],
})
export class SpecializedMenuWithTooltip { }
```

When `SpecializedMenuWithTooltip` is used, instances of all three directives are created with their host bindings applied.

## Host Directive Semantics

### Directive Execution Order

Host directives execute before the component or directive they're applied to. For this example:

```typescript
@Component({
  selector: 'admin-menu',
  template: 'admin-menu.html',
  hostDirectives: [MenuBehavior],
})
export class AdminMenu {}
```

Execution sequence:
1. `MenuBehavior` instantiated
2. `AdminMenu` instantiated
3. `MenuBehavior` receives inputs (`ngOnInit`)
4. `AdminMenu` receives inputs (`ngOnInit`)
5. `MenuBehavior` applies host bindings
6. `AdminMenu` applies host bindings

This order of operations means that components with `hostDirectives` can override any host bindings specified by a host directive.

With nested directives, the pattern extends depth-first:

```typescript
@Directive({...})
export class Tooltip { }

@Directive({
  hostDirectives: [Tooltip],
})
export class CustomTooltip { }

@Directive({
  hostDirectives: [CustomTooltip],
})
export class EvenMoreCustomTooltip { }
```

Execution order:
1. `Tooltip` instantiated
2. `CustomTooltip` instantiated
3. `EvenMoreCustomTooltip` instantiated
4. `Tooltip` receives inputs
5. `CustomTooltip` receives inputs
6. `EvenMoreCustomTooltip` receives inputs
7. `Tooltip` applies host bindings
8. `CustomTooltip` applies host bindings
9. `EvenMoreCustomTooltip` applies host bindings

### Dependency Injection

Components and directives using `hostDirectives` can inject instances of those directives and vice versa. When both a component with `hostDirectives` and its host directives provide the same injection token, the providers defined by class with `hostDirectives` take precedence over providers defined by the host directives.
