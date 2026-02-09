# Navigate to Routes
> Source: https://angular.dev/guide/routing/navigate-to-routes

## Overview

The RouterLink directive enables declarative navigation in Angular applications using standard anchor elements (`<a>`) that integrate seamlessly with Angular's routing system.

## RouterLink Usage

### Basic Implementation

Import RouterLink and apply it to anchor elements:

```typescript
import {RouterLink} from '@angular/router';

@Component({
  template: `
    <nav>
      <a routerLink="/user-profile">User profile</a>
      <a routerLink="/settings">Settings</a>
    </nav>
  `,
  imports: [RouterLink],
  ...
})
export class App {}
```

### Absolute vs. Relative Links

**Absolute URLs** include the full protocol and domain (e.g., `https://www.angular.dev/essentials`). **Relative URLs** assume you're already on the correct domain and specify only the path (e.g., `/essentials`). Relative URLs are generally preferred for maintainability.

### Relative URL Syntax

Angular routing supports two syntaxes for relative paths:

**String syntax** (most common):

```html
<!-- Navigates to /dashboard -->
<a routerLink="dashboard">Dashboard</a>
```

**Array syntax** (for dynamic parameters):

```html
<a [routerLink]="['user', currentUserId]">Current User</a>
```

### Path Resolution Rules

When on `example.com/settings`, routing behaves as follows:

```html
<!-- Navigates to /settings/notifications -->
<a routerLink="notifications">Notifications</a>

<!-- Navigates to /settings/notifications (absolute) -->
<a routerLink="/settings/notifications">Notifications</a>

<!-- Navigates to /team/123/user/456 -->
<a [routerLink]="['/team', teamId, 'user', userId]">Current User</a>
```

Forward slashes prefix absolute paths from the root domain.

## Programmatic Navigation

For logic-based navigation, inject the `Router` service to navigate dynamically through TypeScript code.

### router.navigate() Method

Navigate between routes using a URL path array:

```typescript
import {Router} from '@angular/router';

@Component({
  selector: 'app-dashboard',
  template: `<button (click)="navigateToProfile()">View Profile</button>`,
})
export class AppDashboard {
  private router = inject(Router);

  navigateToProfile() {
    // Standard navigation
    this.router.navigate(['/profile']);

    // With route parameters
    this.router.navigate(['/users', userId]);

    // With query parameters
    this.router.navigate(['/search'], {
      queryParams: {category: 'books', sort: 'price'},
    });

    // With matrix parameters
    this.router.navigate(['/products', {featured: true, onSale: true}]);
  }
}
```

### Relative Navigation with relativeTo

Build dynamic paths relative to the current component's location:

```typescript
import {Router, ActivatedRoute} from '@angular/router';

@Component({
  selector: 'app-user-detail',
  template: `
    <button (click)="navigateToEdit()">Edit User</button>
    <button (click)="navigateToParent()">Back to List</button>
  `,
})
export class UserDetail {
  private route = inject(ActivatedRoute);
  private router = inject(Router);

  // Navigate to sibling route: /users/123 -> /users/123/edit
  navigateToEdit() {
    this.router.navigate(['edit'], {relativeTo: this.route});
  }

  // Navigate to parent: /users/123 -> /users
  navigateToParent() {
    this.router.navigate(['..'], {relativeTo: this.route});
  }
}
```

### router.navigateByUrl() Method

Navigate using complete URL path strings for absolute navigation or deep linking:

```typescript
// Standard route navigation
router.navigateByUrl('/products');

// Navigate to nested route
router.navigateByUrl('/products/featured');

// Complete URL with parameters and fragment
router.navigateByUrl('/products/123?view=details#reviews');

// With query parameters
router.navigateByUrl('/search?category=books&sortBy=price');

// With matrix parameters
router.navigateByUrl('/sales-awesome;isOffer=true;showModal=false');

// Replace current URL in history
router.navigateByUrl('/checkout', {replaceUrl: true});
```

## Next Steps

Learn to read route state to build responsive, context-aware components.
