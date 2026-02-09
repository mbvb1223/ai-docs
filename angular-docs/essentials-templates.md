# Angular Templates - Dynamic Interfaces
> Source: https://angular.dev/essentials/templates

## Overview

Angular templates enable developers to create dynamic user interfaces by establishing connections between component data and rendered HTML. Templates automatically update when underlying data changes, maintaining synchronization between the component class and the view.

## Showing Dynamic Text

### Interpolation with Bindings

Angular uses double curly-braces to create bindings that display dynamic content:

```typescript
@Component({
  selector: 'user-profile',
  template: `<h1>Profile for {{ userName() }}</h1>`,
})
export class UserProfile {
  userName = signal('pro_programmer_123');
}
```

This renders as:

```html
<h1>Profile for pro_programmer_123</h1>
```

### Reactive Updates

When signal values change, Angular automatically refreshes the rendered output:

```typescript
this.userName.set('cool_coder_789');
```

The template immediately reflects: `<h1>Profile for cool_coder_789</h1>`

## Setting Dynamic Properties and Attributes

### Property Binding

Square bracket syntax binds values to DOM properties:

```typescript
@Component({
  template: `<button [disabled]="!isValidUserId()">Save changes</button>`,
})
export class UserProfile {
  isValidUserId = signal(false);
}
```

### Attribute Binding

Prefix attribute names with `attr.` to bind to HTML attributes:

```html
<ul [attr.role]="listRole()"></ul>
```

Angular maintains synchronization as bound values change.

## Handling User Interaction

### Event Listeners

Parentheses attach event handlers to template elements:

```typescript
@Component({
  template: `<button (click)="cancelSubscription()">Cancel subscription</button>`,
})
export class UserProfile {
  cancelSubscription() {
    /* Event handling logic */
  }
}
```

### Accessing Event Objects

Use Angular's built-in `$event` variable to pass event data:

```typescript
@Component({
  template: `<button (click)="cancelSubscription($event)">Cancel subscription</button>`,
})
export class UserProfile {
  cancelSubscription(event: Event) {
    /* Event handling with access to Event object */
  }
}
```

## Control Flow

### Conditional Rendering with @if

The `@if` block conditionally displays sections:

```html
<h1>User profile</h1>
@if (isAdmin()) {
  <h2>Admin settings</h2>
  <!-- admin content -->
}
```

### Optional @else Block

```html
@if (isAdmin()) {
  <h2>Admin settings</h2>
} @else {
  <h2>User settings</h2>
}
```

### Looping with @for

The `@for` block repeats template sections with the `track` keyword:

```html
<ul class="user-badge-list">
  @for (badge of badges(); track badge.id) {
    <li class="user-badge">{{ badge.name }}</li>
  }
</ul>
```

The `track` keyword associates data elements with DOM nodes for optimal rendering performance.

## Next Steps

After mastering template basics, explore the comprehensive in-depth templates guide for advanced features including control flow patterns, advanced bindings, and template optimization techniques.
