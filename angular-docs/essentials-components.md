# Angular Components Guide
> Source: https://angular.dev/essentials/components

## Overview

Components serve as the fundamental building blocks for Angular applications. They represent distinct sections of web pages and help organize applications into maintainable, clearly-separated code sections.

## Core Component Structure

Every Angular component contains four main parts:

1. **@Component decorator** - Configuration used by Angular
2. **HTML template** - Controls what renders to the DOM
3. **CSS selector** - Defines how the component is used in HTML
4. **TypeScript class** - Contains behaviors like handling user input or server requests

## Basic Component Example

```typescript
// user-profile.ts
@Component({
  selector: 'user-profile',
  template: `
    <h1>User profile</h1>
    <p>This is the user profile page</p>
  `,
})
export class UserProfile {
  /* Your component code goes here */
}
```

## Adding Styles

Styles can be included inline using the `styles` property:

```typescript
@Component({
  selector: 'user-profile',
  template: `
    <h1>User profile</h1>
    <p>This is the user profile page</p>
  `,
  styles: `
    h1 {
      font-size: 3em;
    }
  `,
})
export class UserProfile {
  /* Your component code goes here */
}
```

## Separating Files

For larger components, use `templateUrl` and `styleUrl` to reference external files:

```typescript
// user-profile.ts
@Component({
  selector: 'user-profile',
  templateUrl: 'user-profile.html',
  styleUrl: 'user-profile.css',
})
export class UserProfile {
  // Component behavior is defined in here
}
```

```html
<!-- user-profile.html -->
<h1>User profile</h1>
<p>This is the user profile page</p>
```

```css
/* user-profile.css */
h1 {
  font-size: 3em;
}
```

## Using Components

Applications are built by composing multiple components together. For example, a user profile page might include:

- **UserProfile** (parent)
  - UserBiography
  - ProfilePhoto
  - UserAddress

## Importing and Using Components

To use a component:

1. Add an `import` statement in your TypeScript file
2. Include the component in the `imports` array within `@Component`
3. Add an element matching the component's selector in your template

```typescript
// user-profile.ts
import { ProfilePhoto } from 'profile-photo.ts';

@Component({
  selector: 'user-profile',
  imports: [ProfilePhoto],
  template: `
    <h1>User profile</h1>
    <profile-photo />
    <p>This is the user profile page</p>
  `,
})
export class UserProfile {
  // Component behavior is defined in here
}
```

## Next Steps

After understanding components, learn about signals for managing dynamic data in applications. The in-depth components guide provides additional details on advanced component features.
