# Building a Template-Driven Form
> Source: https://angular.dev/guide/forms/template-driven-forms

This guide teaches how to create template-driven forms in Angular, where form controls bind to data properties using two-way data binding. These forms are ideal for small to moderately complex applications.

## Key Concepts

**Template-driven vs. Reactive Forms:**

- Template-driven forms work well for simple forms with fewer controls
- Reactive forms suit complex, scalable applications
- Template-driven forms leverage special directives in Angular templates

## Essential Directives

| Directive | Purpose |
|-----------|---------|
| `NgModel` | Creates two-way bindings for form control values |
| `NgForm` | Generates a top-level FormGroup and tracks form status |
| `NgModelGroup` | Creates FormGroup instances for form sections |

## Step-by-Step Implementation

### 1. Define the Data Model

Create a basic class representing your form data:

```typescript
export class Actor {
  constructor(
    public id: number,
    public name: string,
    public skill: string,
    public studio?: string,
  ) {}
}
```

### 2. Set Up the Component

```typescript
import {Component} from '@angular/core';
import {Actor} from '../actor';
import {FormsModule} from '@angular/forms';
import {JsonPipe} from '@angular/common';

@Component({
  selector: 'app-actor-form',
  templateUrl: './actor-form.component.html',
  imports: [FormsModule, JsonPipe],
})
export class ActorFormComponent {
  skills = ['Method Acting', 'Singing', 'Dancing', 'Swordfighting'];
  model = new Actor(18, 'Tom Cruise', this.skills[3], 'CW Productions');
  submitted = false;

  onSubmit() {
    this.submitted = true;
  }

  newActor() {
    this.model = new Actor(42, '', '');
  }
}
```

### 3. Bind Controls with `ngModel`

Use two-way binding syntax `[(ngModel)]` to connect form inputs to model properties:

```html
<form (ngSubmit)="onSubmit()" #actorForm="ngForm">
  <div class="form-group">
    <label for="name">Name</label>
    <input
      type="text"
      class="form-control"
      id="name"
      required
      [(ngModel)]="model.name"
      name="name"
      #name="ngModel"
    />
  </div>

  <div class="form-group">
    <label for="studio">Studio Affiliation</label>
    <input
      type="text"
      class="form-control"
      id="studio"
      [(ngModel)]="model.studio"
      name="studio"
    />
  </div>

  <div class="form-group">
    <label for="skill">Skill</label>
    <select
      class="form-control"
      id="skill"
      required
      [(ngModel)]="model.skill"
      name="skill"
      #skill="ngModel"
    >
      @for (skill of skills; track $index) {
        <option [value]="skill">{{ skill }}</option>
      }
    </select>
  </div>
</form>
```

## Control State Tracking

Angular automatically applies CSS classes reflecting control state:

| State | True Class | False Class |
|-------|-----------|------------|
| Visited | `ng-touched` | `ng-untouched` |
| Modified | `ng-dirty` | `ng-pristine` |
| Valid | `ng-valid` | `ng-invalid` |

Forms receive `ng-submitted` after submission.

## Visual Feedback with CSS

Add styling for invalid/required controls:

```css
.ng-valid[required],
.ng-valid.required {
  border-left: 5px solid #42A948; /* green */
}

.ng-invalid:not(form) {
  border-left: 5px solid #a94442; /* red */
}
```

## Validation Error Messages

Display conditional error messages using template reference variables:

```html
<input
  type="text"
  id="name"
  required
  [(ngModel)]="model.name"
  name="name"
  #name="ngModel"
/>
<div [hidden]="name.valid || name.pristine" class="alert alert-danger">
  Name is required
</div>
```

The `pristine` check prevents showing errors on initial form load.

## Form Submission

### Handle Submit Events

Bind the form's `ngSubmit` event to your submission handler:

```html
<form (ngSubmit)="onSubmit()" #actorForm="ngForm">
  <!-- form controls -->
  <button type="submit" [disabled]="!actorForm.form.valid">
    Submit
  </button>
</form>
```

### Control Button State

Disable the submit button while the form is invalid by binding to the form's validity status.

## Complete Example

**Component:**

```typescript
export class ActorFormComponent {
  skills = ['Method Acting', 'Singing', 'Dancing', 'Swordfighting'];
  model = new Actor(18, 'Tom Cruise', this.skills[3], 'CW Productions');
  submitted = false;

  onSubmit() {
    this.submitted = true;
  }

  newActor() {
    this.model = new Actor(42, '', '');
  }
}
```

**Template (key sections):**

```html
<div [hidden]="submitted">
  <h1>Actor Form</h1>
  <form (ngSubmit)="onSubmit()" #actorForm="ngForm">
    <!-- All form controls with ngModel bindings -->
    <button type="submit" [disabled]="!actorForm.form.valid">
      Submit
    </button>
    <button type="button" (click)="newActor(); actorForm.reset()">
      New Actor
    </button>
  </form>
</div>

<div [hidden]="!submitted">
  <h2>You submitted the following:</h2>
  <div class="row">
    <div class="col-xs-3">Name</div>
    <div class="col-xs-9">{{ model.name }}</div>
  </div>
  <button type="button" (click)="submitted = false">Edit</button>
</div>
```

## Key Takeaways

- Template-driven forms require importing `FormsModule`
- Use `[(ngModel)]` with `name` attributes for two-way binding
- Template reference variables (`#varName="ngModel"`) access control state
- CSS classes automatically track validity, pristine state, and touched status
- Bind form validity to button `disabled` properties for better UX
- Use conditional rendering to show/hide error messages and form states
