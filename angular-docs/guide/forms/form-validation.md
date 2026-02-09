# Validating Form Input
> Source: https://angular.dev/guide/forms/form-validation

Angular provides comprehensive form validation capabilities for improving data quality by validating user input for accuracy and completeness. The framework supports validation in both template-driven and reactive forms.

## Template-Driven Forms Validation

### Basic Approach

Template-driven forms use HTML validation attributes combined with Angular directives. Angular automatically runs validation whenever form control values change, generating either validation errors (resulting in `INVALID` status) or `null` (resulting in `VALID` status).

### Implementation Example

Export `NgModel` to a local template variable to inspect control state:

```html
<input
  type="text"
  id="name"
  name="name"
  class="form-control"
  required
  minlength="4"
  appForbiddenName="bob"
  [(ngModel)]="actor.name"
  #name="ngModel"
/>

@if (name.invalid && (name.dirty || name.touched)) {
  <div class="alert">
    @if (name.hasError('required')) {
      <div>Name is required.</div>
    }
    @if (name.hasError('minlength')) {
      <div>Name must be at least 4 characters long.</div>
    }
    @if (name.hasError('forbiddenName')) {
      <div>Name cannot be Bob.</div>
    }
  </div>
}
```

### Key Features

- HTML validation attributes (`required`, `minlength`) activate built-in validators
- Custom directives (like `forbiddenName`) wrap custom validator logic
- `NgModel` mirrors properties of the underlying `FormControl` instance
- Error display should check for `dirty` or `touched` states to avoid premature feedback

## Reactive Forms Validation

### Structure

In reactive forms, validators are functions added directly to form control models in the component class rather than as template attributes.

```typescript
actorForm = new FormGroup({
  name: new FormControl(this.actor.name, [
    Validators.required,
    Validators.minLength(4),
    forbiddenNameValidator(/bob/i),
  ]),
  role: new FormControl(this.actor.role),
  skill: new FormControl(this.actor.skill, Validators.required),
});

get name() {
  return this.actorForm.get('name');
}

get skill() {
  return this.actorForm.get('skill');
}
```

### Template Implementation

```html
<input
  type="text"
  id="name"
  class="form-control"
  formControlName="name"
  required
/>

@if (name.invalid && (name.dirty || name.touched)) {
  <div class="alert alert-danger">
    @if (name.hasError('required')) {
      <div>Name is required.</div>
    }
    @if (name.hasError('minlength')) {
      <div>Name must be at least 4 characters long.</div>
    }
    @if (name.hasError('forbiddenName')) {
      <div>Name cannot be Bob.</div>
    }
  </div>
}
```

## Validator Functions

### Synchronous Validators

Synchronous functions return validation errors or `null` immediately. Pass as the second argument when instantiating `FormControl`.

### Asynchronous Validators

Asynchronous functions return a Promise or Observable that eventually emits validation errors or `null`. Pass as the third argument to `FormControl`. Angular only runs async validators after all sync validators pass.

### Built-in Validators

Angular provides validators available as both attributes (template-driven) and functions (reactive):

- `Validators.required`
- `Validators.minLength()`
- `Validators.maxLength()`
- `Validators.pattern()`
- `Validators.email`

See the [Validators API reference](https://angular.dev/api/forms/Validators) for the complete list.

## Custom Validators

### Synchronous Custom Validator

```typescript
export function forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = nameRe.test(control.value);
    return forbidden ? {forbiddenName: {value: control.value}} : null;
  };
}
```

The validator factory returns a function that checks control values and returns `null` for valid input or a validation error object for invalid input.

### Adding to Reactive Forms

Pass the custom validator function directly to `FormControl`:

```typescript
actorForm = new FormGroup({
  name: new FormControl(this.actor.name, [
    Validators.required,
    Validators.minLength(4),
    forbiddenNameValidator(/bob/i),
  ]),
  // ...
});
```

### Adding to Template-Driven Forms

Create a directive wrapping the validator and register it with `NG_VALIDATORS`:

```typescript
@Directive({
  selector: '[appForbiddenName]',
  providers: [
    {
      provide: NG_VALIDATORS,
      useExisting: forwardRef(() => ForbiddenValidatorDirective),
      multi: true,
    },
  ],
})
export class ForbiddenValidatorDirective implements Validator {
  readonly forbiddenName = input<string>('', {alias: 'appForbiddenName'});

  validate(control: AbstractControl): ValidationErrors | null {
    return this.forbiddenName
      ? forbiddenNameValidator(new RegExp(this.forbiddenName(), 'i'))(control)
      : null;
  }
}
```

Then use in templates:

```html
<input
  type="text"
  id="name"
  name="name"
  class="form-control"
  required
  minlength="4"
  appForbiddenName="bob"
  [(ngModel)]="actor.name"
  #name="ngModel"
/>
```

## Control Status CSS Classes

Angular automatically applies CSS classes reflecting control state:

- `.ng-valid` / `.ng-invalid`
- `.ng-pending`
- `.ng-pristine` / `.ng-dirty`
- `.ng-untouched` / `.ng-touched`
- `.ng-submitted` (on form elements only)

### Example Styling

```css
.ng-valid[required], .ng-valid.required {
  border-left: 5px solid #42A948; /* green */
}

.ng-invalid:not(form) {
  border-left: 5px solid #a94442; /* red */
}

.alert div {
  background-color: #fed3d3;
  color: #820000;
  padding: 1rem;
  margin-bottom: 1rem;
}
```

## Cross-Field Validation

Cross-field validators compare values across multiple form fields. They're implemented as custom validators applied at the `FormGroup` level rather than individual controls.

### Reactive Forms Example

```typescript
export const unambiguousRoleValidator: ValidatorFn = (
  control: AbstractControl,
): ValidationErrors | null => {
  const name = control.get('name');
  const role = control.get('role');
  return name && role && name.value === role.value
    ? {unambiguousRole: true}
    : null;
};

const actorForm = new FormGroup(
  {
    'name': new FormControl(),
    'role': new FormControl(),
    'skill': new FormControl(),
  },
  {validators: unambiguousRoleValidator},
);
```

Display error messages when validation fails:

```html
@if (actorForm.hasError('unambiguousRole') &&
     (actorForm.touched || actorForm.dirty)) {
  <div class="cross-validation-error-message alert alert-danger">
    Name cannot match role or audiences will be confused.
  </div>
}
```

### Template-Driven Forms Example

Create a directive implementing `Validator`:

```typescript
@Directive({
  selector: '[appUnambiguousRole]',
  providers: [
    {
      provide: NG_VALIDATORS,
      useExisting: forwardRef(() => UnambiguousRoleValidatorDirective),
      multi: true,
    },
  ],
})
export class UnambiguousRoleValidatorDirective implements Validator {
  validate(control: AbstractControl): ValidationErrors | null {
    return unambiguousRoleValidator(control);
  }
}
```

Apply to the form tag:

```html
<form #actorForm="ngForm" appUnambiguousRole>
  <!-- form controls -->
</form>

@if (actorForm.hasError('unambiguousRole') &&
     (actorForm.touched || actorForm.dirty)) {
  <div class="cross-validation-error-message alert">
    Name cannot match role.
  </div>
}
```

## Asynchronous Validators

### Implementation Pattern

Asynchronous validators implement `AsyncValidatorFn` or `AsyncValidator` interfaces and must return a Promise or Observable that completes.

```typescript
@Injectable({providedIn: 'root'})
export class UniqueRoleValidator implements AsyncValidator {
  private readonly actorsService = inject(ActorsService);

  validate(control: AbstractControl): Observable<ValidationErrors | null> {
    return this.actorsService.isRoleTaken(control.value).pipe(
      map((isTaken) => (isTaken ? {uniqueRole: true} : null)),
      catchError(() => of(null)),
    );
  }
}
```

The form control enters a `pending` state during async validation and remains there until the observable completes.

### Adding to Reactive Forms

```typescript
roleValidator = inject(UniqueRoleValidator);

const roleControl = new FormControl('', {
  asyncValidators: [this.roleValidator.validate.bind(this.roleValidator)],
  updateOn: 'blur',
});
```

### Adding to Template-Driven Forms

Create a directive and register with `NG_ASYNC_VALIDATORS`:

```typescript
@Directive({
  selector: '[appUniqueRole]',
  providers: [
    {
      provide: NG_ASYNC_VALIDATORS,
      useExisting: forwardRef(() => UniqueRoleValidatorDirective),
      multi: true,
    },
  ],
})
export class UniqueRoleValidatorDirective implements AsyncValidator {
  private readonly validator = inject(UniqueRoleValidator);

  validate(control: AbstractControl): Observable<ValidationErrors | null> {
    return this.validator.validate(control);
  }
}
```

Use in templates:

```html
<input
  type="text"
  id="role"
  name="role"
  #role="ngModel"
  [(ngModel)]="actor.role"
  [ngModelOptions]="{updateOn: 'blur'}"
  appUniqueRole
/>
```

### Handling Pending State

Display visual feedback while async validation executes:

```html
<input [(ngModel)]="name" #model="ngModel" appSomeAsyncValidator />
@if (model.pending) {
  <app-spinner />
}
```

### Performance Optimization

To avoid excessive HTTP requests, defer validation to blur or submit events rather than validating on every change:

**Template-Driven:**

```html
<input [(ngModel)]="name" [ngModelOptions]="{updateOn: 'blur'}" />
```

**Reactive:**

```typescript
new FormControl('', {updateOn: 'blur'});
```

## Native HTML Validation Integration

By default, Angular disables native HTML form validation by adding the `novalidate` attribute to forms. To use native validation alongside Angular validation, enable it with the `ngNativeValidate` directive.
