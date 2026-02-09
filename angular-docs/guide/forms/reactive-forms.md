# Reactive Forms
> Source: https://angular.dev/guide/forms/reactive-forms

Reactive forms offer a model-driven approach for managing form inputs that change over time. This paradigm leverages explicit, immutable state management where each change returns a new state, maintaining model integrity. Reactive forms use an explicit and immutable approach to managing the state of a form at a given point in time.

Key distinctions from template-driven forms include:

- Synchronous access to data models
- Immutability through observable operators
- Change tracking via observable streams

## Core Concepts

### FormControl Basics

A `FormControl` represents a single form field. Implementation requires three steps:

1. Generate a component and import `ReactiveFormsModule`
2. Create a `FormControl` instance
3. Register it in the template using `[formControl]` binding

**Basic Example:**

```typescript
import {Component} from '@angular/core';
import {FormControl, ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'app-name-editor',
  templateUrl: './name-editor.component.html',
  styleUrls: ['./name-editor.component.css'],
  imports: [ReactiveFormsModule],
})
export class NameEditorComponent {
  name = new FormControl('');
}
```

**Template binding:**

```html
<label for="name">Name: </label>
<input id="name" type="text" [formControl]="name" />
<p>Value: {{ name.value }}</p>
```

### Displaying and Updating Values

Access current values through:

- The `value` property (snapshot)
- The `valueChanges` observable (reactive updates)

Update values programmatically:

```typescript
updateName() {
  this.name.setValue('Nancy');
}
```

## FormGroup: Multiple Controls

`FormGroup` manages related controls as a single unit. Create one by providing an object of named controls:

```typescript
profileForm = new FormGroup({
  firstName: new FormControl(''),
  lastName: new FormControl(''),
});
```

**Template integration:**

```html
<form [formGroup]="profileForm" (ngSubmit)="onSubmit()">
  <label for="first-name">First Name: </label>
  <input id="first-name" type="text" formControlName="firstName" />

  <label for="last-name">Last Name: </label>
  <input id="last-name" type="text" formControlName="lastName" />

  <button type="submit" [disabled]="!profileForm.valid">Submit</button>
</form>
```

### Nested Form Groups

Complex forms benefit from hierarchical organization:

```typescript
profileForm = new FormGroup({
  firstName: new FormControl(''),
  lastName: new FormControl(''),
  address: new FormGroup({
    street: new FormControl(''),
    city: new FormControl(''),
    state: new FormControl(''),
    zip: new FormControl(''),
  }),
});
```

**Nested template structure:**

```html
<div formGroupName="address">
  <h2>Address</h2>
  <label for="street">Street: </label>
  <input id="street" type="text" formControlName="street" />
  <label for="city">City: </label>
  <input id="city" type="text" formControlName="city" />
</div>
```

### Updating Partial Data

Two methods handle model updates:

| Method | Behavior |
|--------|----------|
| `setValue()` | Strictly adheres to form structure; replaces entire value |
| `patchValue()` | Updates only specified properties; silently ignores mismatches |

**Example:**

```typescript
updateProfile() {
  this.profileForm.patchValue({
    firstName: 'Nancy',
    address: {
      street: '123 Drew Street',
    },
  });
}
```

## FormBuilder Service

`FormBuilder` reduces repetitive control instantiation:

```typescript
import {FormBuilder, ReactiveFormsModule} from '@angular/forms';

private formBuilder = inject(FormBuilder);

profileForm = this.formBuilder.group({
  firstName: [''],
  lastName: [''],
  address: this.formBuilder.group({
    street: [''],
    city: [''],
    state: [''],
    zip: [''],
  }),
});
```

The array syntax allows initial values as the first element, with validators as subsequent arguments.

## Form Validation

Reactive forms include built-in validators. Basic implementation:

```typescript
import {Validators} from '@angular/forms';

profileForm = this.formBuilder.group({
  firstName: ['', Validators.required],
  lastName: [''],
  address: this.formBuilder.group({
    street: [''],
    city: [''],
    state: [''],
    zip: [''],
  }),
});
```

Display validation status:

```html
<p>Form Status: {{ profileForm.status }}</p>
```

## FormArray: Dynamic Forms

`FormArray` manages an unbounded number of unnamed controls. This proves valuable when the control count is unknown:

```typescript
import {FormArray} from '@angular/forms';

profileForm = this.formBuilder.group({
  firstName: ['', Validators.required],
  lastName: [''],
  address: this.formBuilder.group({
    street: [''],
    city: [''],
    state: [''],
    zip: [''],
  }),
  aliases: this.formBuilder.array([this.formBuilder.control('')]),
});

get aliases() {
  return this.profileForm.get('aliases') as FormArray;
}

addAlias() {
  this.aliases.push(this.formBuilder.control(''));
}
```

**Template rendering:**

```html
<div formArrayName="aliases">
  <h2>Aliases</h2>
  <button type="button" (click)="addAlias()">+ Add another alias</button>

  @for (alias of aliases.controls; track $index; let i = $index) {
    <div>
      <label for="alias-{{ i }}">Alias:</label>
      <input id="alias-{{ i }}" type="text" [formControlName]="i" />
    </div>
  }
</div>
```

## Top-Level FormArray

Bind a `FormArray` directly to a form element when it represents the entire form model:

```typescript
import {Component} from '@angular/core';
import {FormArray, FormControl} from '@angular/forms';

@Component({
  selector: 'form-array-example',
  template: `
    <form [formArray]="form">
      @for (control of form.controls; track $index) {
        <input [formControlName]="$index" />
      }
    </form>
  `,
})
export class FormArrayExampleComponent {
  controls = [
    new FormControl('fish'),
    new FormControl('cat'),
    new FormControl('dog'),
  ];
  form = new FormArray(this.controls);
}
```

## Unified Control State Change Events

All form controls expose state changes through the `events` observable on `AbstractControl`, providing a single unified stream for tracking modifications across `FormControl`, `FormGroup`, `FormArray`, and `FormRecord` instances.

## Key Advantages

Reactive forms excel in scenarios requiring:

- Complex, multi-step validation
- Dynamic control management
- Programmatic value synchronization
- Testable, type-safe implementations
- Observable-based reactive patterns

The structured approach maintains clear separation between model and view, enabling sophisticated form orchestration while maintaining code maintainability.
