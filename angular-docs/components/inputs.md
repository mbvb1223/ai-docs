# Accepting Data with Input Properties
> Source: https://angular.dev/guide/components/inputs

## Overview

Input properties enable components to receive data from parent components. Angular provides two approaches: the modern signal-based `input` function and the decorator-based `@Input` API.

## Signal-Based Inputs (Recommended)

### Basic Declaration

Components declare inputs using the `input()` function:

```typescript
import {Component, input} from '@angular/core';

@Component({/*...*/})
export class CustomSlider {
  value = input(0);
}
```

Usage in templates:
```html
<custom-slider [value]="50" />
```

### Type Inference

TypeScript automatically infers types from default values. Without a default, the type becomes `Type | undefined`:

```typescript
value = input(0);              // InputSignal<number>
value = input<number>();      // InputSignal<number | undefined>
```

### Key Characteristics

- **Compile-time static recording**: Inputs cannot be dynamically added or removed at runtime
- **Read-only signals**: The `input()` function returns read-only `InputSignal` objects
- **Case-sensitive names**: Input names distinguish between uppercase and lowercase
- **Inheritance**: Child classes inherit parent inputs automatically
- **Exclusive usage**: `input()` calls are only valid in component/directive property initializers

## Reading Input Values

Access input values by calling the signal as a function:

```typescript
import {Component, input, computed} from '@angular/core';

@Component({/*...*/})
export class CustomSlider {
  value = input(0);
  label = computed(() => `The slider's value is ${this.value()}`);
}
```

## Required Inputs

Mark inputs as mandatory using `input.required()`:

```typescript
@Component({/*...*/})
export class CustomSlider {
  value = input.required<number>();
}
```

Angular enforces at build-time that all required inputs receive values when components are instantiated.

## Configuring Inputs

### Input Transforms

Transform functions modify input values when set. Common use cases include accepting broader value types:

```typescript
@Component({selector: 'custom-slider', /*...*/})
export class CustomSlider {
  label = input('', {transform: trimString});
}

function trimString(value: string | undefined): string {
  return value?.trim() ?? '';
}
```

**Constraints**:
- Must be statically analyzable at build-time
- Should be pure functions without external state dependencies
- The transform parameter type determines accepted template values

**Built-in transforms**:
- `booleanAttribute`: Treats attribute presence as true; handles "false" string as boolean false
- `numberAttribute`: Parses values to numbers, producing `NaN` on failure

```typescript
import {Component, input, booleanAttribute, numberAttribute} from '@angular/core';

@Component({/*...*/})
export class CustomSlider {
  disabled = input(false, {transform: booleanAttribute});
  value = input(0, {transform: numberAttribute});
}
```

### Input Aliases

Change the template property name while keeping the TypeScript property unchanged:

```typescript
@Component({/*...*/})
export class CustomSlider {
  value = input(0, {alias: 'sliderValue'});
}
```

```html
<custom-slider [sliderValue]="50" />
```

**Use cases**: Avoiding DOM property name collisions or preserving backward compatibility.

## Model Inputs

Model inputs enable two-way data binding, allowing components to propagate values back to parent components:

```typescript
import {Component, model} from '@angular/core';

@Component({/*...*/})
export class CustomSlider {
  value = model(0);

  increment() {
    this.value.update((oldValue) => oldValue + 10);
  }
}
```

Parent usage with two-way binding:
```typescript
@Component({
  template: `<custom-slider [(value)]="volume" />`,
})
export class MediaControls {
  volume = signal(0);
}
```

### Implicit Change Outputs

Declaring a model input automatically creates a corresponding output named `{inputName}Change`:

```typescript
@Directive({/*...*/})
export class CustomCheckbox {
  checked = model(false);  // Automatically creates "checkedChange" output
}
```

### Two-Way Binding with Plain Properties

Model inputs work with plain JavaScript properties:

```typescript
@Component({
  template: '<custom-slider [(value)]="volume" />',
})
export class MediaControls {
  protected volume = 0;
}
```

### Model Input Configuration

Model inputs support `required` and `alias` options but not transforms.

### When to Use Model Inputs

Use model inputs for user-interaction-driven value modifications, typically in custom form controls like date pickers or comboboxes.

## Decorator-Based Inputs (@Input)

The original `@Input` decorator remains fully supported:

```typescript
@Component({/*...*/})
export class CustomSlider {
  @Input() value = 0;
}
```

Template binding is identical to signal-based inputs.

### @Input Configuration

**Required inputs**:
```typescript
@Input({required: true}) value = 0;
```

**Transforms**:
```typescript
@Input({transform: trimString}) label = '';
```

**Aliases**:
```typescript
@Input({alias: 'sliderValue'}) value = 0;
// Or: @Input('sliderValue') value = 0;
```

### Getters and Setters

Properties with getters/setters can be inputs:

```typescript
export class CustomSlider {
  @Input()
  get value(): number {
    return this.internalValue;
  }
  set value(newValue: number) {
    this.internalValue = newValue;
  }
  private internalValue = 0;
}
```

**Note**: Input transforms are preferable to getters/setters. Avoid costly operations in setters due to multiple Angular invocations.

### Specifying Inputs in @Component

Declare inputs via the decorator's `inputs` property, useful for inherited properties:

```typescript
@Component({
  ...,
  inputs: ['disabled'],
})
export class CustomSlider extends BaseSlider { }
```

Include aliases using colon syntax:
```typescript
inputs: ['disabled: sliderDisabled']
```

## Input Naming Best Practices

- Avoid collisions with standard DOM element properties (e.g., HTMLElement properties)
- Skip component-specific prefixes; the component selector already identifies the element as custom
