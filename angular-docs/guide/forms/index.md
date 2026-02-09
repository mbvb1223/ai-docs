# Angular Forms Overview
> Source: https://angular.dev/guide/forms

Angular provides two primary approaches for handling user input through forms: **reactive forms** and **template-driven forms**. Both capture user input events, validate data, create form models, and track changes, but they differ in implementation philosophy and data flow patterns.

## Choosing Your Approach

### Reactive Forms

Reactive forms offer direct access to the underlying form object model. They excel in scenarios where forms are central to your application:

- More robust and scalable
- Better for reusability and testing
- Ideal for complex form requirements
- Use synchronous data flow

### Template-Driven Forms

Template-driven forms rely on directives within templates to manage the underlying object model:

- Simpler to implement for basic scenarios
- Useful for simple additions like email signup forms
- Less scalable than reactive forms
- Use asynchronous data flow

## Key Differences Summary

| Aspect | Reactive | Template-driven |
|--------|----------|-----------------|
| Form model setup | Explicit, in component class | Implicit, created by directives |
| Data model | Structured and immutable | Unstructured and mutable |
| Data flow | Synchronous | Asynchronous |
| Form validation | Functions | Directives |

## Foundation Classes

Both approaches build upon these base classes:

- **FormControl**: Tracks value and validation status of individual form controls
- **FormGroup**: Tracks values and status for a collection of form controls
- **FormArray**: Tracks values and status for an array of form controls
- **ControlValueAccessor**: Creates a bridge between FormControl instances and built-in DOM elements

## Reactive Forms Setup

In reactive forms, you explicitly define the form model in the component class and link it using the `[formControl]` directive:

```typescript
import {Component} from '@angular/core';
import {FormControl, ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'app-reactive-favorite-color',
  template: ` Favorite Color: <input type="text" [formControl]="favoriteColorControl" /> `,
  imports: [ReactiveFormsModule],
})
export class FavoriteColorReactive {
  favoriteColorControl = new FormControl('');
}
```

In reactive forms, the form model is the source of truth; it provides the value and status of the form element at any given point in time.

## Template-Driven Forms Setup

In template-driven forms, the `NgModel` directive manages the FormControl instance implicitly:

```typescript
import {Component, signal} from '@angular/core';
import {FormsModule} from '@angular/forms';

@Component({
  selector: 'app-template-favorite-color',
  template: ` Favorite Color: <input type="text" [(ngModel)]="favoriteColor" /> `,
  imports: [FormsModule],
})
export class FavoriteColorTemplate {
  favoriteColor = signal('');
}
```

In a template-driven form the source of truth is the template. The NgModel directive automatically manages the FormControl instance for you.

## Data Flow in Reactive Forms

### View to Model Flow

1. User types a value into the input element
2. Form input element emits an "input" event with the latest value
3. ControlValueAccessor immediately relays the new value to the FormControl instance
4. FormControl emits the new value through the `valueChanges` observable
5. Subscribers to `valueChanges` receive the new value

### Model to View Flow

1. User calls `setValue()` method on the FormControl
2. FormControl instance emits the new value through the `valueChanges` observable
3. Subscribers receive the new value
4. Control value accessor updates the form input element with the new value

## Data Flow in Template-Driven Forms

### View to Model Flow

1. User types a value into the input element
2. Input element emits an "input" event with the value
3. Control value accessor triggers `setValue()` on the FormControl instance
4. FormControl emits the new value through the `valueChanges` observable
5. Subscribers receive the new value
6. Control value accessor calls `NgModel.viewToModelUpdate()`, emitting `ngModelChange`
7. Two-way data binding updates the component property with the emitted value

### Model to View Flow

1. Component property value changes
2. Change detection begins
3. `ngOnChanges` lifecycle hook is called on the NgModel directive instance
4. `ngOnChanges()` queues an async task to set the value for the internal FormControl
5. Change detection completes
6. On the next tick, the task to set FormControl value executes
7. FormControl emits the latest value through the `valueChanges` observable
8. Subscribers receive the new value
9. Control value accessor updates the form input element in the view

## Scalability Considerations

Reactive forms are more scalable than template-driven forms. They provide direct access to the underlying form API, and use synchronous data flow.

Reactive forms advantages:

- Direct API access enables reusability across components
- Synchronous data flow simplifies large-scale form management
- Less setup required for testing
- Testing does not require deep understanding of change detection

Template-driven forms considerations:

- Focus on simple scenarios with limited reusability
- Abstract away the underlying form API
- Asynchronous data flow affects testing complexity
- Tests are deeply reliant on manual change detection execution

## Summary

Choose **reactive forms** when forms are central to your application, require scalability, or need complex validation logic. Select **template-driven forms** for simple form additions with basic requirements that can be managed entirely within the template.
