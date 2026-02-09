# Referencing Component Children with Queries
> Source: https://angular.dev/guide/components/queries

## Overview

Angular components can define **queries** to locate child elements and access values from their injectors. This capability enables developers to retrieve references to child components, directives, DOM elements, and other injectable dependencies.

Query functions return signals reflecting current results. These signals work in reactive contexts like `computed` and `effect` by calling the signal as a function.

Two primary query categories exist: **view queries** and **content queries**.

## View Queries

View queries retrieve results from elements defined in a component's own template. Use `viewChild` for single results:

```typescript
@Component({
  selector: 'custom-card-header',
  /*...*/
})
export class CustomCardHeader {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: '<custom-card-header>Visit sunny California!</custom-card-header>',
})
export class CustomCard {
  header = viewChild(CustomCardHeader);
  headerText = computed(() => this.header()?.text);
}
```

When no result exists, the query returns `undefined`. This occurs if the target element is conditionally hidden via `@if`. Angular maintains up-to-date query results as state changes.

For multiple results, use `viewChildren`:

```typescript
@Component({
  selector: 'custom-card-action',
  /*...*/
})
export class CustomCardAction {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: `
    <custom-card-action>Save</custom-card-action>
    <custom-card-action>Cancel</custom-card-action>
  `,
})
export class CustomCard {
  actions = viewChildren(CustomCardAction);
  actionsTexts = computed(() => this.actions().map((action) => action.text));
}
```

**Important:** Queries never pierce component boundaries. View queries only retrieve results from the component's own template.

## Content Queries

Content queries retrieve results from elements nested inside a component (its content). Use `contentChild` for single results:

```typescript
@Component({
  selector: 'custom-toggle',
  /*...*/
})
export class CustomToggle {
  text: string;
}

@Component({
  selector: 'custom-expando',
  /* ... */
})
export class CustomExpando {
  toggle = contentChild(CustomToggle);
  toggleText = computed(() => this.toggle()?.text);
}

@Component({
  /* ... */
  template: `
    <custom-expando>
      <custom-toggle>Show</custom-toggle>
    </custom-expando>
  `,
})
export class UserProfile {}
```

By default, content queries find only direct children, not descendants. When absent or hidden conditionally, the query returns `undefined`.

For multiple results, use `contentChildren`:

```typescript
@Component({
  selector: 'custom-menu-item',
  /*...*/
})
export class CustomMenuItem {
  text: string;
}

@Component({
  selector: 'custom-menu',
  /*...*/
})
export class CustomMenu {
  items = contentChildren(CustomMenuItem);
  itemTexts = computed(() => this.items().map((item) => item.text));
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-menu>
      <custom-menu-item>Cheese</custom-menu-item>
      <custom-menu-item>Tomato</custom-menu-item>
    </custom-menu>
  `,
})
export class UserProfile {}
```

**Important:** Content queries never cross component boundaries.

## Required Queries

When child queries find no result, they return `undefined`. Use required queries when certain a child always exists:

```typescript
@Component({
  /*...*/
})
export class CustomCard {
  header = viewChild.required(CustomCardHeader);
  body = contentChild.required(CustomCardBody);
}
```

Required queries report an error if no match exists. They exclude `undefined` from the signal's type.

## Query Locators

The first parameter specifies the **locator** -- typically a component or directive:

```typescript
@Component({
  /*...*/
  template: `
    <button #save>Save</button>
    <button #cancel>Cancel</button>
  `,
})
export class ActionBar {
  saveButton = viewChild<ElementRef<HTMLButtonElement>>('save');
}
```

Template reference variables work as string locators. CSS selectors are unsupported.

### Injector-Based Locators

Use any `ProviderToken` as a locator:

```typescript
const SUB_ITEM = new InjectionToken<string>('sub-item');

@Component({
  /*...*/
  providers: [{provide: SUB_ITEM, useValue: 'special-item'}],
})
export class SpecialItem {}

@Component({
  /*...*/
})
export class CustomList {
  subItemType = contentChild(SUB_ITEM);
}
```

## Query Options

All query functions accept an options object controlling result retrieval.

### Reading Specific Injector Values

The `read` option retrieves different values from matched elements:

```typescript
@Component({
  /*...*/
})
export class CustomExpando {
  toggle = contentChild(ExpandoContent, {read: TemplateRef});
}
```

Common uses include retrieving `ElementRef` and `TemplateRef`.

### Content Descendants

By default, `contentChildren` finds only direct children. Set `descendants: true` to traverse all descendants:

```typescript
@Component({
  selector: 'custom-expando',
  /*...*/
})
export class CustomExpando {
  toggle = contentChildren(CustomToggle); // none found
  // toggle = contentChild(CustomToggle); // found
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-expando>
      <some-other-component>
        <custom-toggle>Show</custom-toggle>
      </some-other-component>
    </custom-expando>
  `,
})
export class UserProfile {}
```

View queries always traverse descendants.

## Decorator-Based Queries

Signal-based queries are recommended for new projects, but decorators remain fully supported.

### View Query Decorators

Use `@ViewChild` for single results:

```typescript
@Component({
  selector: 'custom-card-header',
  /*...*/
})
export class CustomCardHeader {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: '<custom-card-header>Visit sunny California!</custom-card-header>',
})
export class CustomCard implements AfterViewInit {
  @ViewChild(CustomCardHeader) header: CustomCardHeader;
  ngAfterViewInit() {
    console.log(this.header.text);
  }
}
```

Results become available in `ngAfterViewInit`. Before this point, values are `undefined`.

Use `@ViewChildren` for multiple results:

```typescript
@Component({
  selector: 'custom-card-action',
  /*...*/
})
export class CustomCardAction {
  text: string;
}

@Component({
  selector: 'custom-card',
  template: `
    <custom-card-action>Save</custom-card-action>
    <custom-card-action>Cancel</custom-card-action>
  `,
})
export class CustomCard implements AfterViewInit {
  @ViewChildren(CustomCardAction) actions: QueryList<CustomCardAction>;
  ngAfterViewInit() {
    this.actions.forEach((action) => {
      console.log(action.text);
    });
  }
}
```

`@ViewChildren` returns a `QueryList` with array-like convenience APIs and a `changes` property for subscribing to updates.

### Content Query Decorators

Use `@ContentChild` for single results:

```typescript
@Component({
  selector: 'custom-toggle',
  /*...*/
})
export class CustomToggle {
  text: string;
}

@Component({
  selector: 'custom-expando',
  /*...*/
})
export class CustomExpando implements AfterContentInit {
  @ContentChild(CustomToggle) toggle: CustomToggle;
  ngAfterContentInit() {
    console.log(this.toggle.text);
  }
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-expando>
      <custom-toggle>Show</custom-toggle>
    </custom-expando>
  `,
})
export class UserProfile {}
```

Results become available in `ngAfterContentInit`. Before this point, values are `undefined`.

Use `@ContentChildren` for multiple results:

```typescript
@Component({
  selector: 'custom-menu-item',
  /*...*/
})
export class CustomMenuItem {
  text: string;
}

@Component({
  selector: 'custom-menu',
  /*...*/
})
export class CustomMenu implements AfterContentInit {
  @ContentChildren(CustomMenuItem) items: QueryList<CustomMenuItem>;
  ngAfterContentInit() {
    this.items.forEach((item) => {
      console.log(item.text);
    });
  }
}

@Component({
  selector: 'user-profile',
  template: `
    <custom-menu>
      <custom-menu-item>Cheese</custom-menu-item>
      <custom-menu-item>Tomato</custom-menu-item>
    </custom-menu>
  `,
})
export class UserProfile {}
```

### Static Queries

The `static` option for `@ViewChild` and `@ContentChild` guarantees the target is always present and unconditionally rendered:

```typescript
@Component({
  selector: 'custom-card',
  template: '<custom-card-header>Visit sunny California!</custom-card-header>',
})
export class CustomCard implements OnInit {
  @ViewChild(CustomCardHeader, {static: true}) header: CustomCardHeader;
  ngOnInit() {
    console.log(this.header.text);
  }
}
```

Static queries become available in `ngOnInit` but do not update after initialization.

## Common Pitfalls

- **Shared State:** Maintain a single source of truth for state shared between components to prevent synchronization issues.

- **Direct Child Writes:** Avoid directly modifying child component state, which creates brittle, error-prone code susceptible to `ExpressionChangedAfterItHasBeenChecked` errors.

- **Direct Parent Writes:** Never directly modify parent or ancestor component state, leading to fragile code and similar error conditions.
