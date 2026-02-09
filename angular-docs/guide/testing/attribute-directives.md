# Testing Attribute Directives
> Source: https://angular.dev/guide/testing/attribute-directives

## Overview

An attribute directive modifies the behavior of an element, component, or another directive, applied as an attribute on a host element.

## The Highlight Directive Example

The sample application includes a `Highlight` directive that sets background color based on data binding or default values:

```typescript
import {Directive, inject, input} from '@angular/core';

/**
 * Set backgroundColor for the attached element to highlight color
 * and set the element's customProperty attribute to true
 */
@Directive({
  selector: '[highlight]',
  host: {
    '[style.backgroundColor]': 'bgColor() || defaultColor',
  },
})
export class Highlight {
  readonly defaultColor = 'rgb(211, 211, 211)'; // lightgray
  readonly bgColor = input('', {alias: 'highlight'});
}
```

### Usage Example

The directive appears in components like this:

```typescript
@Component({
  imports: [Twain, Highlight],
  template: `
    <h2 highlight="skyblue">About</h2>
    <h3>Quote of the day:</h3>
    <twain-quote />
  `,
})
export class About {}
```

## Testing Approaches

### Limitations of Component-Specific Tests

Testing a directive only through individual component usage has drawbacks. Finding all components using the directive is tedious, brittle, and unlikely to provide comprehensive coverage. Isolated class-only tests don't interact with the DOM, limiting confidence in directive efficacy.

### Artificial Test Component Solution

Create a dedicated test component demonstrating all directive usage patterns:

```typescript
@Component({
  imports: [Highlight],
  template: `
    <h2 highlight="yellow">Something Yellow</h2>
    <h2 highlight>The Default (Gray)</h2>
    <h2>No Highlight</h2>
    <input #box [highlight]="box.value" value="cyan" />
  `,
})
class Test {}
```

**Note:** The input element binds the directive to its color value, initially "cyan".

## Test Implementation

```typescript
let fixture: ComponentFixture<Test>;
let des: DebugElement[]; // the three elements w/ the directive

beforeEach(async () => {
  fixture = TestBed.createComponent(Test);
  await fixture.whenStable();

  // all elements with an attached Highlight
  des = fixture.debugElement.queryAll(By.directive(Highlight));
});

// color tests
it('should have three highlighted elements', () => {
  expect(des.length).toBe(3);
});

it('should color 1st <h2> background "yellow"', () => {
  const bgColor = des[0].nativeElement.style.backgroundColor;
  expect(bgColor).toBe('yellow');
});

it('should color 2nd <h2> background w/ default color', () => {
  const dir = des[1].injector.get(Highlight);
  const bgColor = des[1].nativeElement.style.backgroundColor;
  expect(bgColor).toBe(dir.defaultColor);
});

it('should bind <input> background to value color', async () => {
  // easier to work with nativeElement
  const input = des[2].nativeElement as HTMLInputElement;
  expect(input.style.backgroundColor, 'initial backgroundColor').toBe('cyan');
  input.value = 'green';
  // Dispatch a DOM event so that Angular responds to the input value change.
  input.dispatchEvent(new Event('input'));
  await fixture.whenStable();
  expect(input.style.backgroundColor, 'changed backgroundColor').toBe('green');
});

it('bare <h2> should not have a backgroundColor', () => {
  // the h2 without the Highlight directive
  const bareH2 = fixture.debugElement.query(By.css('h2:not([highlight])'));
  expect(bareH2.styles.backgroundColor).toBeUndefined();
});
```

## Key Testing Techniques

### 1. By.directive() Predicate

The `By.directive()` method finds elements with a specific directive when element types are unknown.

### 2. CSS :not() Pseudo-class

Use selectors like `By.css('h2:not([highlight])')` to locate elements without the directive, or `By.css('*:not([highlight])')` for any element lacking it.

### 3. DebugElement.styles

Access element styles through the abstraction layer without requiring a real browser.

### 4. Directive Injector Access

Retrieve directive instances via `DebugElement.injector.get(DirectiveClass)` to access directive properties.

### 5. DebugElement.properties

Access artificial custom properties set by directives through this property.

## Best Practices

- Create artificial test components that demonstrate all directive usage scenarios
- Use `By.directive()` for element discovery without knowing types
- Combine DOM manipulation tests with directive instance verification
- Dispatch DOM events to test reactive behavior with data binding
- Leverage `DebugElement` abstractions for cross-environment testing
