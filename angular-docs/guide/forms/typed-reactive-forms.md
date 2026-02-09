# Strictly Typed Reactive Forms
> Source: https://angular.dev/guide/forms/typed-forms

As of Angular 14, reactive forms are strictly typed by default. This enhancement provides type safety when working with form models, preventing common errors that occurred in previous Angular versions where APIs included `any` types.

## Key Benefits

The strict typing system enables:

- **Type Safety**: Code like `login.value.email.domain` won't compile if the property doesn't exist
- **IDE Autocomplete**: Better developer experience with intelligent suggestions
- **Explicit Form Structure**: Clear specification of form hierarchies and their types

## FormControl: Getting Started

### Basic Usage

The simplest form consists of a single control:

```typescript
const email = new FormControl('angularrox@gmail.com');
```

This automatically infers the type as `FormControl<string|null>`. TypeScript enforces this type across all APIs: `email.value`, `email.valueChanges`, `email.setValue(...)`, etc.

### Nullability

Controls can become `null` by calling `reset()`:

```typescript
const email = new FormControl('angularrox@gmail.com');
email.reset();
console.log(email.value); // null
```

To prevent null values, use the `nonNullable` option, which resets to the initial value instead:

```typescript
const email = new FormControl('angularrox@gmail.com', {nonNullable: true});
email.reset();
console.log(email.value); // angularrox@gmail.com
```

### Explicit Type Specification

When a control initializes to `null`, explicitly specify the type:

```typescript
const email = new FormControl<string | null>(null);
email.setValue('angularrox@gmail.com');
```

## FormArray: Dynamic, Homogenous Collections

A `FormArray` manages an open-ended list of controls with uniform typing:

```typescript
const names = new FormArray([new FormControl('Alex')]);
names.push(new FormControl('Jess'));
```

Add multiple controls at once:

```typescript
const aliases = new FormArray([new FormControl('ng')]);
aliases.push([new FormControl('ngDev'), new FormControl('ngAwesome')]);
```

The `clear()` method removes all controls:

```typescript
aliases.clear();
console.log(aliases.length); // 0
```

**Note**: For heterogeneous arrays with different element types, use `UntypedFormArray`.

## FormGroup and FormRecord

### Partial Values

With a typical form:

```typescript
const login = new FormGroup({
  email: new FormControl('', {nonNullable: true}),
  password: new FormControl('', {nonNullable: true}),
});
```

Disabled controls don't appear in `login.value`, making fields potentially `undefined`. Access all values including disabled controls with `login.getRawValue()`.

### Optional Controls

Some forms have controls that may not always be present:

```typescript
interface LoginForm {
  email: FormControl<string>;
  password?: FormControl<string>;
}
const login = new FormGroup<LoginForm>({
  email: new FormControl('', {nonNullable: true}),
  password: new FormControl('', {nonNullable: true}),
});
login.removeControl('password');
```

TypeScript enforces that only optional controls can be added or removed.

### FormRecord for Dynamic Groups

For forms where keys aren't known beforehand:

```typescript
const addresses = new FormRecord<FormControl<string | null>>({});
addresses.addControl('Andrew', new FormControl('2340 Folsom St'));
```

Build with FormBuilder:

```typescript
const addresses = fb.record({'Andrew': '2340 Folsom St'});
```

For dynamic, heterogeneous groups, use `UntypedFormGroup`.

## FormBuilder and NonNullableFormBuilder

### FormBuilder

The `FormBuilder` class supports the new type system:

```typescript
const fb = new FormBuilder();
const login = fb.group({
  email: '',
  password: '',
});
```

### NonNullableFormBuilder

Shorthand for applying `{nonNullable: true}` to every control:

```typescript
const login = fb.nonNullable.group({
  email: '',
  password: '',
});
```

Inject it directly:

```typescript
constructor(private fb: NonNullableFormBuilder) {}
```

## Untyped Forms (Legacy Support)

Non-typed forms remain supported for backward compatibility. Import `Untyped` symbols:

```typescript
const login = new UntypedFormGroup({
  email: new UntypedFormControl(''),
  password: new UntypedFormControl(''),
});
```

These maintain previous Angular version semantics. Incrementally migrate by removing `Untyped` prefixes.

## Scope and Limitations

These improvements apply exclusively to **reactive forms**. Template-driven forms do not yet support strict typing.
