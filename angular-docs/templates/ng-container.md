# Grouping Elements with ng-container
> Source: https://angular.dev/guide/templates/ng-container

## Overview

The `<ng-container>` element serves as a special grouping mechanism in Angular templates. It allows developers to organize multiple elements or mark specific template locations without introducing actual DOM elements.

### Key Characteristic

When rendered, `<ng-container>` does not appear in the final DOM structure:

```html
<!-- Template -->
<section>
  <ng-container>
    <h3>User bio</h3>
    <p>Here's some info about the user</p>
  </ng-container>
</section>

<!-- Rendered Result -->
<section>
  <h3>User bio</h3>
  <p>Here's some info about the user</p>
</section>
```

**Important Limitation:** Angular ignores all attribute bindings and event listeners applied directly to `<ng-container>`, including those applied through directives.

## Using ng-container for Dynamic Content

### Rendering Components Dynamically

The `NgComponentOutlet` directive enables dynamic component rendering at the container's location:

```typescript
@Component({
  template: `
    <h2>Your profile</h2>
    <ng-container [ngComponentOutlet]="profileComponent()" />
  `,
})
export class UserProfile {
  isAdmin = input(false);
  profileComponent = computed(() => (
    this.isAdmin() ? AdminProfile : BasicUserProfile
  ));
}
```

This approach selects between `AdminProfile` or `BasicUserProfile` based on the `isAdmin` signal value.

### Rendering Template Fragments

Use `NgTemplateOutlet` to dynamically render template fragments:

```typescript
@Component({
  template: `
    <h2>Your profile</h2>
    <ng-container [ngTemplateOutlet]="profileTemplate()" />
    <ng-template #admin>This is the admin profile</ng-template>
    <ng-template #basic>This is the basic profile</ng-template>
  `,
})
export class UserProfile {
  isAdmin = input(false);
  adminTemplate = viewChild('admin', {read: TemplateRef});
  basicTemplate = viewChild('basic', {read: TemplateRef});
  profileTemplate = computed(() => (
    this.isAdmin() ? this.adminTemplate() : this.basicTemplate()
  ));
}
```

## Using ng-container with Structural Directives

Structural directives like `ngIf` and `ngFor` work effectively with `<ng-container>`:

```html
<ng-container *ngIf="permissions == 'admin'">
  <h1>Admin Dashboard</h1>
  <admin-infographic />
</ng-container>

<ng-container *ngFor="let item of items; index as i; trackBy: trackByFn">
  <h2>{{ item.title }}</h2>
  <p>{{ item.description }}</p>
</ng-container>
```

This prevents unnecessary wrapper elements while maintaining conditional or iterative rendering.

## Using ng-container for Dependency Injection

Apply directives to `<ng-container>` to provide values to descendant components. Descendant elements can inject the directive or anything it provides:

```typescript
@Directive({
  selector: '[theme]',
})
export class Theme {
  // Input accepts 'light' or 'dark', defaulting to 'light'
  mode = input<'light' | 'dark'>('light');
}
```

```html
<ng-container theme="dark">
  <profile-pic />
  <user-bio />
</ng-container>
```

Both `ProfilePic` and `UserBio` components can inject the `Theme` directive and apply styling based on its `mode` value. This provides a declarative way to supply configuration to a template section without introducing wrapper elements.
