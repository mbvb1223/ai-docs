# Angular Coding Style Guide
> Source: https://angular.dev/style-guide

## Overview

The Angular Style Guide provides standardized coding conventions for Angular applications. These practices promote consistency across the Angular ecosystem and facilitate code sharing between projects, though they are not strictly required for Angular functionality.

## Key Principle

Whenever you encounter a situation in which these rules contradict the style of a particular file, prioritize maintaining consistency within a file.

---

## Naming Conventions

### File Naming
- **Separate words with hyphens**: Use kebab-case for file names (e.g., `user-profile.ts`)
- **Test files**: End unit test files with `.spec.ts` (e.g., `user-profile.spec.ts`)
- **Match content to filename**: File names should reflect their contents; avoid generic names like `helpers.ts`, `utils.ts`, or `common.ts`

### Component File Structure
Components should use consistent naming across related files:
- TypeScript: `user-profile.ts`
- Template: `user-profile.html`
- Styles: `user-profile.css`

For multiple style files, append descriptive terms (e.g., `user-profile-settings.css`, `user-profile-subscription.css`)

---

## Project Structure

### Directory Organization
- **UI code location**: All Angular code lives in a `src` directory
- **Bootstrap entry point**: Application initialization occurs in `src/main.ts`
- **File grouping**: Place related files (component TypeScript, template, styles, and tests) in the same directory
- **Feature-based organization**: Structure projects by feature areas rather than code type

**Example structure:**
```
src/
├─ movie-reel/
│ ├─ show-times/
│ │ ├─ film-calendar/
│ │ ├─ film-details/
│ ├─ reserve-tickets/
│ │ ├─ payment-info/
│ │ ├─ purchase-confirmation/
```

### Anti-patterns to Avoid
- Do not create directories based on code type (`components/`, `directives/`, `services/`)
- Avoid overcrowding directories; split into subdirectories when they become hard to navigate
- Do not collect unrelated tests in a single `tests` directory

### Single Concept per File
Focus each file on one primary concept. For Angular specifically, this typically means one component, directive, or service per file, though small related classes can coexist if part of a unified concept.

---

## Dependency Injection

### Prefer the `inject` Function
Use the `inject()` function instead of constructor parameter injection for several advantages:

- **Readability**: More readable when classes have multiple dependencies
- **Comments**: Easier to add comments to injected dependencies
- **Type inference**: Offers better type safety
- **ES2022+ compatibility**: Works better with `useDefineForClassFields`

Migration tools exist to refactor existing code to use `inject()`.

---

## Components and Directives

### Selector Conventions
- Components and directives should use consistent application-specific prefixes
- Directive attribute selectors use camelCase (e.g., `[mrTooltip]` for a MovieReel tooltip directive)

### Code Organization
Group Angular-specific properties at the top of class declarations:
- Injected dependencies
- Input properties
- Output properties
- Query decorators
- Methods follow property declarations

This improves discoverability of template APIs and component dependencies.

### Presentation Focus
Components and directives should relate primarily to UI rendering. Move unrelated logic (validation rules, data transformations) into separate functions or services.

### Template Complexity
- Use template expressions for straightforward logic
- Refactor complex logic into TypeScript code, typically as computed signals
- No hard rule defines "complex" -- use professional judgment

### Access Modifiers

**`protected` for template-only members:**
```typescript
@Component({
  template: `<p>{{ fullName() }}</p>`,
})
export class UserProfile {
  firstName = input();
  lastName = input();

  // Used only in template, not part of public API
  protected fullName = computed(() =>
    `${this.firstName()} ${this.lastName()}`
  );
}
```

**`readonly` for Angular-initialized properties:**
- Mark inputs, outputs, model, and query properties as `readonly`
- Prevents accidental overwrites of values set by Angular
- Applies to `input()`, `model()`, `output()` functions and decorator-based queries

### Styling Preferences

**Prefer native bindings over directives:**

Preferred approach:
```typescript
[class.admin]="isAdmin"
[class.dense]="density === 'high'"
[style.color]="textColor"
[style.background-color]="backgroundColor"
```

Avoid:
```typescript
[ngClass]="{admin: isAdmin, dense: density === 'high'}"
[ngStyle]="{'color': textColor, 'background-color': backgroundColor}"
```

Native bindings use clearer syntax aligned with standard HTML and offer better performance than `NgClass` and `NgStyle` directives.

### Event Handler Naming

**Name handlers for their action, not the triggering event:**

Preferred:
```html
<button (click)="saveUserData()">Save</button>
```

Avoid:
```html
<button (click)="handleClick()">Save</button>
```

For keyboard events, use key modifiers with specific names:
```html
<textarea
  (keydown.control.enter)="commitNotes()"
  (keydown.control.space)="showSuggestions()">
</textarea>
```

### Lifecycle Management

**Keep lifecycle hooks simple:**
- Avoid long or complex logic inside hooks like `ngOnInit`
- Call well-named methods from lifecycle hooks instead

Preferred:
```typescript
ngOnInit() {
  this.startLogging();
  this.runBackgroundTask();
}
```

Avoid:
```typescript
ngOnInit() {
  this.logger.setMode('info');
  this.logger.monitorErrors();
  // ... all unrolled logic
}
```

**Implement lifecycle interfaces:**
```typescript
import {Component, OnInit} from '@angular/core';

@Component({/*...*/})
export class UserProfile implements OnInit {
  ngOnInit() {
    /* ... */
  }
}
```

Implementing lifecycle interfaces ensures method names remain correct and improves code clarity.

---

## Related Resources

- [Google's TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
- Components guide (selectors, inputs, outputs, queries)
- Expression syntax guide
- Testing guide
- Signal documentation
