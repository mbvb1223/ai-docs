# Anatomy of a Component
> Source: https://angular.dev/guide/components

## Overview

Angular components consist of three essential elements:

1. **TypeScript class** - Contains behaviors like handling user input and fetching server data
2. **HTML template** - Controls what renders into the DOM
3. **CSS selector** - Defines how the component is used in HTML

## Basic Component Structure

Components are created using the `@Component` decorator:

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
})
export class ProfilePhoto {}
```

The object passed to `@Component` is called the component's **metadata**, which includes selector, template, and other properties.

## Styling Components

Optional CSS styles can be added directly to the component:

```typescript
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
  styles: `
    img {
      border-radius: 50%;
    }
  `,
})
export class ProfilePhoto {}
```

By default, a component's styles only affect elements in that component's template.

## Separate Template and Style Files

You can organize presentation and behavior separately:

```typescript
@Component({
  selector: 'profile-photo',
  templateUrl: 'profile-photo.html',
  styleUrl: 'profile-photo.css',
})
export class ProfilePhoto {}
```

Both `templateUrl` and `styleUrl` are relative to the component's directory.

## Using Components

### Imports in the Decorator

To use a component, directive, or pipe, add it to the `imports` array:

```typescript
import {ProfilePhoto} from './profile-photo';

@Component({
  imports: [ProfilePhoto],
  /* ... */
})
export class UserProfile {}
```

**Note:** By default, Angular components are standalone. Earlier versions may specify `standalone: false`. In versions before 19.0.0, `standalone` defaults to `false`.

### Showing Components in Templates

Every component defines a CSS selector:

```typescript
@Component({
  selector: 'profile-photo',
  ...
})
export class ProfilePhoto { }
```

Display a component by creating a matching HTML element:

```typescript
@Component({
  selector: 'profile-photo',
})
export class ProfilePhoto {}

@Component({
  imports: [ProfilePhoto],
  template: `<profile-photo />`,
})
export class UserProfile {}
```

## Key Concepts

**Host Element:** The DOM element that matches a component's selector.

**View:** The DOM rendered by a component, corresponding to that component's template.

**Component Tree:** Applications compose components in a hierarchical tree structure, with each component instance created for every matching HTML element encountered. This structure is important for understanding dependency injection and child queries.

---

**Related Topics:** For detailed template information including data binding and event handling, see the Templates guide. For styling details, see the Styling Components guide. For component selectors guidance, see Component Selectors.
