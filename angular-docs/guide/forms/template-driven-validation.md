# Template-Driven Form Validation
> Source: https://angular.dev/guide/forms/template-driven-forms
> Source: https://angular.dev/guide/forms/form-validation

This guide covers validation techniques specific to Angular's template-driven forms. Template-driven validation uses HTML validation attributes combined with Angular directives to validate user input directly in the template.

## Overview

In template-driven forms, validation is configured through HTML attributes and custom directives rather than programmatic validator functions. Angular maps standard HTML validation attributes to its built-in validator functions automatically.

## Adding Validation to Template-Driven Forms

### Built-in Validation Attributes

Add standard HTML validation attributes to form controls. Angular automatically activates the corresponding validator directives:

```html
<input
  type="text"
  id="name"
  name="name"
  class="form-control"
  required
  minlength="4"
  [(ngModel)]="actor.name"
  #name="ngModel"
/>
```

Supported built-in validation attributes:

- `required` -- Field must have a value
- `minlength` -- Minimum character length
- `maxlength` -- Maximum character length
- `pattern` -- Must match a regular expression
- `email` -- Must be a valid email format

### Accessing Control State

Export `NgModel` to a local template variable using `#name="ngModel"` to inspect the control's validation state. `NgModel` mirrors many properties of the underlying `FormControl` instance:

- `valid` / `invalid` -- Whether the control passes all validators
- `dirty` / `pristine` -- Whether the user has changed the value
- `touched` / `untouched` -- Whether the user has focused and blurred the control
- `errors` -- Object containing active validation errors

### Displaying Validation Errors

Show error messages conditionally based on control state. Check for `dirty` or `touched` to avoid showing errors before the user interacts with the field:

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

## Control Status CSS Classes

Angular automatically applies CSS classes that reflect the validation and interaction state of each control:

| CSS Class | Condition |
|-----------|-----------|
| `.ng-valid` | Control passes all validators |
| `.ng-invalid` | Control fails one or more validators |
| `.ng-pending` | Async validation is in progress |
| `.ng-pristine` | User has not changed the value |
| `.ng-dirty` | User has changed the value |
| `.ng-untouched` | User has not focused and blurred the control |
| `.ng-touched` | User has focused and blurred the control |
| `.ng-submitted` | Form has been submitted (form elements only) |

### Styling Example

```css
.ng-valid[required],
.ng-valid.required {
  border-left: 5px solid #42A948; /* green */
}

.ng-invalid:not(form) {
  border-left: 5px solid #a94442; /* red */
}
```

## Custom Validators for Template-Driven Forms

To use a custom validator in a template-driven form, you must wrap it in a directive and register it with the `NG_VALIDATORS` provider token.

### Creating the Validator Function

```typescript
export function forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = nameRe.test(control.value);
    return forbidden ? {forbiddenName: {value: control.value}} : null;
  };
}
```

### Creating the Validator Directive

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

### Using the Custom Validator

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

## Cross-Field Validation in Template-Driven Forms

Cross-field validators compare values across multiple controls. In template-driven forms, implement them as directives applied to the `<form>` element.

### Creating the Cross-Field Validator

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
```

### Creating the Directive

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

### Using Cross-Field Validation

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

## Async Validators in Template-Driven Forms

Async validators are used when validation requires a server call (e.g., checking for uniqueness). They are registered with `NG_ASYNC_VALIDATORS`.

### Creating the Async Validator Service

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

### Creating the Async Validator Directive

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

### Using the Async Validator

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

### Showing Pending State

Display a spinner or loading indicator while async validation is in progress:

```html
<input [(ngModel)]="name" #model="ngModel" appSomeAsyncValidator />
@if (model.pending) {
  <app-spinner />
}
```

## Performance: Controlling Update Timing

By default, validation runs on every input change. For async validators making HTTP requests, this can be expensive. Use `ngModelOptions` to defer validation:

### Validate on Blur

```html
<input [(ngModel)]="name" [ngModelOptions]="{updateOn: 'blur'}" />
```

### Validate on Submit

```html
<input [(ngModel)]="name" [ngModelOptions]="{updateOn: 'submit'}" />
```

## Disabling the Submit Button

Bind the submit button's `disabled` property to the form's validity:

```html
<form #actorForm="ngForm">
  <!-- form controls -->
  <button type="submit" [disabled]="!actorForm.form.valid">
    Submit
  </button>
</form>
```

## Native HTML Validation

By default, Angular disables native HTML form validation by adding the `novalidate` attribute. To re-enable it alongside Angular validation, use the `ngNativeValidate` directive:

```html
<form ngNativeValidate>
  <!-- form controls -->
</form>
```
