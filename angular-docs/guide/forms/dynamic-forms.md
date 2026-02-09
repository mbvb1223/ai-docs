# Building Dynamic Forms
> Source: https://angular.dev/guide/forms/dynamic-forms

Dynamic forms enable you to create form templates based on metadata that describes a business object model. This approach allows you to generate different form versions automatically as data models change, which is particularly useful for questionnaires and scenarios requiring frequent content updates.

## Key Concepts

### When to Use Dynamic Forms

Dynamic forms are ideal when:

- Form content must change frequently to meet evolving business requirements
- You need to collect input from users in varying contexts
- The form format should remain consistent while questions vary
- You want to avoid hardcoding form structures in application code

### Tutorial Overview

This guide walks through creating a dynamic form for a hero job application by:

1. Enabling reactive forms in your project
2. Establishing a data model for form controls
3. Populating the model with sample data
4. Developing components that create form controls dynamically

## Implementation Steps

### 1. Enable Reactive Forms

Import `ReactiveFormsModule` from `@angular/forms` into your components:

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-dynamic-form',
  templateUrl: './dynamic-form.component.html',
  imports: [DynamicFormQuestionComponent, ReactiveFormsModule],
})
export class DynamicFormComponent { }
```

### 2. Create the Form Object Model

#### QuestionBase Class

Establish a base class representing form questions:

```typescript
export class QuestionBase<T> {
  value: T | undefined;
  key: string;
  label: string;
  required: boolean;
  order: number;
  controlType: string;
  type: string;
  options: {key: string; value: string}[];

  constructor(
    options: {
      value?: T;
      key?: string;
      label?: string;
      required?: boolean;
      order?: number;
      controlType?: string;
      type?: string;
      options?: {key: string; value: string}[];
    } = {},
  ) {
    this.value = options.value;
    this.key = options.key || '';
    this.label = options.label || '';
    this.required = !!options.required;
    this.order = options.order === undefined ? 1 : options.order;
    this.controlType = options.controlType || '';
    this.type = options.type || '';
    this.options = options.options || [];
  }
}
```

#### Define Control Classes

Create specialized question types:

**TextboxQuestion:**

```typescript
import { QuestionBase } from './question-base';

export class TextboxQuestion extends QuestionBase<string> {
  override controlType = 'textbox';
}
```

**DropdownQuestion:**

```typescript
import { QuestionBase } from './question-base';

export class DropdownQuestion extends QuestionBase<string> {
  override controlType = 'dropdown';
}
```

### 3. Compose Form Groups

Create a service to build form groups from question metadata:

```typescript
import { Injectable } from '@angular/core';
import { FormControl, FormGroup, Validators } from '@angular/forms';
import { QuestionBase } from './question-base';

@Injectable()
export class QuestionControlService {
  toFormGroup(questions: QuestionBase<string>[]) {
    const group: any = {};
    questions.forEach((question) => {
      group[question.key] = question.required
        ? new FormControl(question.value || '', Validators.required)
        : new FormControl(question.value || '');
    });
    return new FormGroup(group);
  }
}
```

### 4. Build Dynamic Form Components

#### DynamicFormQuestionComponent Template

```html
<div [formGroup]="form()">
  <label [attr.for]="question().key">{{ question().label }}</label>
  <div>
    @switch (question().controlType) {
      @case ('textbox') {
        <input
          [formControlName]="question().key"
          [id]="question().key"
          [type]="question().type"
        />
      }
      @case ('dropdown') {
        <select [id]="question().key" [formControlName]="question().key">
          @for (opt of question().options; track opt) {
            <option [value]="opt.key">{{ opt.value }}</option>
          }
        </select>
      }
    }
  </div>
  @if (!isValid) {
    <div class="errorMessage">{{ question().label }} is required</div>
  }
</div>
```

#### DynamicFormQuestionComponent Class

```typescript
import { Component, input } from '@angular/core';
import { FormGroup, ReactiveFormsModule } from '@angular/forms';
import { QuestionBase } from './question-base';

@Component({
  selector: 'app-question',
  templateUrl: './dynamic-form-question.component.html',
  imports: [ReactiveFormsModule],
})
export class DynamicFormQuestionComponent {
  readonly question = input.required<QuestionBase<string>>();
  readonly form = input.required<FormGroup>();

  get isValid() {
    return this.form().controls[this.question().key].valid;
  }
}
```

### 5. Supply Question Data

Create a service to provide question metadata:

```typescript
import { Injectable } from '@angular/core';
import { DropdownQuestion } from './question-dropdown';
import { QuestionBase } from './question-base';
import { TextboxQuestion } from './question-textbox';
import { of } from 'rxjs';

@Injectable()
export class QuestionService {
  getQuestions() {
    const questions: QuestionBase<string>[] = [
      new DropdownQuestion({
        key: 'favoriteAnimal',
        label: 'Favorite Animal',
        options: [
          {key: 'cat', value: 'Cat'},
          {key: 'dog', value: 'Dog'},
          {key: 'horse', value: 'Horse'},
          {key: 'capybara', value: 'Capybara'},
        ],
        order: 3,
      }),
      new TextboxQuestion({
        key: 'firstName',
        label: 'First name',
        value: 'Alex',
        required: true,
        order: 1,
      }),
      new TextboxQuestion({
        key: 'emailAddress',
        label: 'Email',
        type: 'email',
        order: 2,
      }),
    ];
    return of(questions.sort((a, b) => a.order - b.order));
  }
}
```

### 6. Create the Dynamic Form Template

#### DynamicFormComponent Template

```html
<div>
  <form (ngSubmit)="onSubmit()" [formGroup]="form()">
    @for (question of questions(); track question) {
      <div class="form-row">
        <app-question [question]="question" [form]="form()" />
      </div>
    }
    <div class="form-row">
      <button type="submit" [disabled]="!form().valid">Save</button>
    </div>
  </form>
  @if (payLoad) {
    <div class="form-row">
      <strong>Saved the following values</strong><br />{{ payLoad }}
    </div>
  }
</div>
```

#### DynamicFormComponent Class

```typescript
import { Component, computed, inject, input } from '@angular/core';
import { FormGroup, ReactiveFormsModule } from '@angular/forms';
import { DynamicFormQuestionComponent } from './dynamic-form-question.component';
import { QuestionBase } from './question-base';
import { QuestionControlService } from './question-control.service';

@Component({
  selector: 'app-dynamic-form',
  templateUrl: './dynamic-form.component.html',
  providers: [QuestionControlService],
  imports: [DynamicFormQuestionComponent, ReactiveFormsModule],
})
export class DynamicFormComponent {
  private readonly qcs = inject(QuestionControlService);
  readonly questions = input<QuestionBase<string>[] | null>([]);
  readonly form = computed<FormGroup>(() =>
    this.qcs.toFormGroup(this.questions() as QuestionBase<string>[]),
  );
  payLoad = '';

  onSubmit() {
    this.payLoad = JSON.stringify(this.form().getRawValue());
  }
}
```

### 7. Display the Form

Create a root component that provides questions to the form:

```typescript
import { Component, inject } from '@angular/core';
import { AsyncPipe } from '@angular/common';
import { DynamicFormComponent } from './dynamic-form.component';
import { QuestionService } from './question.service';
import { QuestionBase } from './question-base';
import { Observable } from 'rxjs';

@Component({
  selector: 'app-root',
  template: `
    <div>
      <h2>Job Application for Heroes</h2>
      <app-dynamic-form [questions]="questions$ | async" />
    </div>
  `,
  providers: [QuestionService],
  imports: [AsyncPipe, DynamicFormComponent],
})
export class AppComponent {
  questions$: Observable<QuestionBase<string>[]> =
    inject(QuestionService).getQuestions();
}
```

## Key Features

### Input Validation

The form automatically validates based on question requirements and control types, displaying error messages when inputs are invalid.

### Conditional Submit Button

The Submit button remains disabled until all required fields are valid, ensuring data quality.

### Flexible Architecture

The separation between model and rendering allows you to reuse components for any survey compatible with the question object model.

### Dynamic Rendering

Use `@switch` statements to render different control types based on question metadata, allowing unlimited question types without modifying component code.

## Benefits

This approach enables:

- Rapid form creation without code changes
- Easy updates to questionnaire content
- Consistent user experience across varying forms
- Reusable components across projects
- Clear separation of concerns between data and presentation
