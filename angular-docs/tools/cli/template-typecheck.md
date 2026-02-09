# Template Type Checking in Angular
> Source: https://angular.dev/tools/cli/template-typecheck

## Overview

Angular provides type checking for template expressions and bindings, similar to how TypeScript catches code errors. The system operates in three modes controlled by `fullTemplateTypeCheck` and `strictTemplates` compiler flags.

## Three Modes of Type Checking

### Basic Mode

With `fullTemplateTypeCheck: false`, Angular validates only top-level expressions:

- Verifies properties exist on component classes
- Checks object property chains (e.g., `user.address.city`)
- Does NOT verify value assignability to component inputs

**Limitations:**
- Skips embedded views (`*ngIf`, `*ngFor`, `<ng-template>`)
- Template references (`#refs`), pipe results, and `$event` all become `any` type
- Subsequent expressions remain unchecked

### Full Mode

With `fullTemplateTypeCheck: true`, Angular performs more aggressive checking:

- Validates embedded views
- Verifies pipe return types
- Types directive and pipe references correctly (except generic parameters)

Items still typed as `any`:
- DOM element references
- `$event` objects
- Safe navigation expressions

**Note:** This flag has been deprecated since Angular 13 in favor of `strictTemplates`.

### Strict Mode

Setting `strictTemplates: true` combines full mode behavior with additional checks:

- Confirms component/directive binding assignability to `input()` declarations
- Respects TypeScript's `strictNullChecks` flag
- Infers correct types for components/directives including generics
- Determines template context types (e.g., for `NgFor`)
- Properly types `$event` in component, directive, DOM, and animation bindings
- Infers DOM element reference types based on HTML tag names

## Embedded View Type Checking Example

Consider this template structure:

```typescript
interface User {
  name: string;
  address: {
    city: string;
    state: string;
  };
}
```

```html
<div *ngFor="let user of users">
  <h2>{{config.title}}</h2>
  <span>City: {{user.address.city}}</span>
</div>
```

**Behavior across modes:**
- **Basic:** Neither element checked
- **Full:** Both checked with `any` type assumptions
- **Strict:** `user` properly typed as `User`, `address` recognized as object with `city: string`

## Troubleshooting Type Errors

Strict mode may reveal genuine type mismatches or false positives from incomplete library typings. Solutions include:

### Using `$any()` Cast

Suppress type-checking for specific expressions:

```html
{{$any(person).address.street}}
```

### Strictness Flags

| Flag | Purpose |
|------|---------|
| `strictInputTypes` | Validates binding expression assignability to `@Input()` fields |
| `strictInputAccessModifiers` | Honors `private`/`protected`/`readonly` on inputs |
| `strictNullInputTypes` | Applies `strictNullChecks` to input bindings |
| `strictAttributeTypes` | Checks attribute-based input bindings |
| `strictSafeNavigationTypes` | Infers safe navigation operator return types |
| `strictDomLocalRefTypes` | Types DOM element references correctly |
| `strictOutputEventTypes` | Types `$event` for component/directive outputs |
| `strictDomEventTypes` | Types `$event` for DOM events |
| `strictContextGenerics` | Infers generic component type parameters |
| `strictLiteralTypes` | Infers object/array literal types |

### Configuration Options

- Disable all strict checking: `strictTemplates: false` in `tsconfig.json`
- Skip null checks on inputs: `strictNullInputTypes: false`
- Disable specific checks individually while maintaining others

## Input Type Handling

### Strict Null Checking

Two common scenarios cause false positives:

1. **Nullable values bound to library components** not compiled with `strictNullChecks`
2. **`async` pipe with synchronous Observables** (always returns `null | value`)

**Workarounds:**

```html
<!-- Non-null assertion -->
<user-detail [user]="user!"></user-detail>

<!-- For async pipe -->
<user-detail [user]="(user$ | async)!"></user-detail>
```

Or disable strict null checking entirely via `strictNullInputTypes: false`.

### Input Setter Coercion

When an input setter accepts multiple types through transformation (e.g., `boolean | ''` for a disabled attribute), use the static property pattern:

```typescript
@Component({
  selector: 'submit-button',
  template: '<button [disabled]="disabled">Submit</button>'
})
class SubmitButton {
  @Input()
  set disabled(value: boolean) {
    this._disabled = value === '' || value;
  }

  static ngAcceptInputType_disabled: boolean | '';
}
```

This field communicates acceptable binding types to the type checker without requiring an actual value. Note: TypeScript 4.3+ supports setter type overloading, making this pattern less necessary.

## Fallback Strategy

If troubleshooting doesn't resolve issues:
1. Try disabling `strictTemplates`
2. If needed, disable `fullTemplateTypeCheck`
3. File an issue if errors prevent compilation -- this may indicate a type-checker bug

## Library Author Guidance

- Enable `strictNullChecks` in your build
- Include `null` in input types where applicable
- Provide type hints for structural directives using template guards
- Document input coercion behavior clearly
