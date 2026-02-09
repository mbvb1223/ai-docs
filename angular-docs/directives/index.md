# Built-in Directives
> Source: https://angular.dev/guide/directives

## Overview

Directives are classes that enhance element behavior in Angular applications. The framework provides three primary directive categories:

| Type | Purpose |
|------|---------|
| Components | Template-based directives (most common) |
| Attribute Directives | Modify element appearance/behavior |
| Structural Directives | Alter DOM layout by adding/removing elements |

## Built-in Attribute Directives

Attribute directives listen to and modify HTML elements, attributes, properties, and components. Common examples include:

| Directive | Function |
|-----------|----------|
| `NgClass` | Add/remove multiple CSS classes |
| `NgStyle` | Add/remove multiple inline styles |
| `NgModel` | Enable two-way form data binding |

**Note:** Built-in directives utilize only public APIs without special private access.

## NgClass: Managing CSS Classes

### Implementation Steps

**1. Import NgClass**

Add `NgClass` to your component's imports array:

```typescript
import {NgClass} from '@angular/common';

@Component({
  imports: [NgClass],
})
export class AppComponent implements OnInit {
  // component logic
}
```

### Using with Expressions

Bind `[ngClass]` to a conditional expression:

```html
<div [ngClass]="isSpecial ? 'special' : ''">
  This div is special
</div>
```

### Using with Methods

Create a method that returns an object mapping class names to boolean conditions:

```typescript
currentClasses: Record<string, boolean> = {};

setCurrentClasses() {
  this.currentClasses = {
    saveable: this.canSave,
    modified: !this.isUnchanged,
    special: this.isSpecial,
  };
}
```

Apply in template:

```html
<div [ngClass]="currentClasses">
  This div is initially saveable, unchanged, and special.
</div>
```

**Guidance:** For single class toggling, use class binding instead of `NgClass`.

## NgStyle: Managing Inline Styles

### Implementation Steps

**1. Import NgStyle**

```typescript
import {NgStyle} from '@angular/common';

@Component({
  imports: [NgStyle],
})
export class AppComponent implements OnInit {
  // component logic
}
```

### Setting Styles

Create a method returning an object with style properties:

```typescript
currentStyles: Record<string, string> = {};

setCurrentStyles() {
  this.currentStyles = {
    'font-style': this.canSave ? 'italic' : 'normal',
    'font-weight': !this.isUnchanged ? 'bold' : 'normal',
    'font-size': this.isSpecial ? '24px' : '12px',
  };
}
```

Apply in template:

```html
<div [ngStyle]="currentStyles">
  This div is initially italic, normal weight, and extra large (24px).
</div>
```

**Guidance:** Use style binding for individual styles rather than `NgStyle`.

## ng-container: Directive Hosting Without DOM Elements

The `<ng-container>` element groups directives without interfering with DOM structure or styling, as Angular excludes it from the rendered output.

### Conditional Content Example

```html
<p>
  I turned the corner
  <ng-container *ngIf="hero">
    and saw {{ hero.name }}. I waved
  </ng-container>
  and continued on my way.
</p>
```

### Form Options Example

```html
<div>
  Pick your favorite hero
  (<label for="showSad">
    <input
      id="showSad"
      type="checkbox"
      checked
      (change)="showSad = !showSad"
    />
    show sad
  </label>)
</div>

<select [(ngModel)]="hero">
  <ng-container *ngFor="let h of heroes">
    <ng-container *ngIf="showSad || h.emotion !== 'sad'">
      <option [ngValue]="h">
        {{ h.name }} ({{ h.emotion }})
      </option>
    </ng-container>
  </ng-container>
</select>
```

## Next Steps

- Explore [Attribute Directives](./attribute-directives.md)
- Learn [Structural Directives](./structural-directives.md)
- Implement [Directive Composition API](./directive-composition-api.md)
