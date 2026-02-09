# Basics of Testing Components
> Source: https://angular.dev/guide/testing/components-basics

## Overview

A component uniquely combines an HTML template with a TypeScript class. The component truly is the template and the class working together. Effective testing requires validating that these parts function correctly as an integrated unit.

Testing components demands creating the component's host element in the browser DOM and examining how the component class interacts with the DOM structure defined by its template.

## Component DOM Testing

Components involve more than just class logic. They interact with DOM elements and other components. Class-only testing cannot verify whether:

- Methods are properly bound to user interactions
- Template values display correctly
- Users can interact with component elements
- Data transforms as expected (e.g., uppercase conversion)
- Template content renders properly

Complex components have intricate DOM interactions where HTML appears and disappears based on state changes. Comprehensive testing requires creating associated DOM elements, examining state-dependent rendering at appropriate moments, and simulating user interactions to confirm expected behavior.

### CLI-Generated Tests

The Angular CLI automatically generates initial test files. For example:

```bash
ng generate component banner --inline-template --inline-style
```

This generates `banner.spec.ts`:

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {Banner} from './banner';

describe('Banner', () => {
  let component: Banner;
  let fixture: ComponentFixture<Banner>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [Banner],
    }).compileComponents();
    fixture = TestBed.createComponent(Banner);
    component = fixture.componentInstance;
    await fixture.whenStable();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

### Reducing Setup Boilerplate

Most of this generated code is setup anticipating future needs. You can simplify it significantly:

```typescript
describe('Banner (minimal)', () => {
  it('should create', () => {
    const fixture = TestBed.createComponent(Banner);
    const component = fixture.componentInstance;
    expect(component).toBeDefined();
  });
});
```

**Note:** `TestBed.compileComponents` is only required when `@defer` blocks appear in tested components.

## Key Testing Concepts

### `createComponent()`

After configuring `TestBed`, call its `createComponent()` method:

```typescript
const fixture = TestBed.createComponent(Banner);
```

This method creates a component instance, adds it to the test-runner DOM, and returns a `ComponentFixture`.

**Important:** Do not reconfigure `TestBed` after calling `createComponent()`. This call freezes the current configuration, preventing further modifications. Attempting additional configuration throws an error.

### `ComponentFixture`

The `ComponentFixture` serves as a test harness for component interaction. Access the component instance through it:

```typescript
const component = fixture.componentInstance;
expect(component).toBeDefined();
```

### `beforeEach()`

Rather than duplicating setup code across tests, refactor shared configuration into `beforeEach()`:

```typescript
describe('Banner (with beforeEach)', () => {
  let component: Banner;
  let fixture: ComponentFixture<Banner>;

  beforeEach(async () => {
    fixture = TestBed.createComponent(Banner);
    component = fixture.componentInstance;
    await fixture.whenStable(); // necessary to wait for initial rendering
  });

  it('should create', () => {
    expect(component).toBeDefined();
  });
});
```

Awaiting `fixture.whenStable()` ensures the initial rendering completes, making individual tests synchronous.

### Alternative: Setup Functions

Instead of `beforeEach`, create a customizable setup function:

```typescript
function setup(providers?: StaticProviders[]): ComponentFixture<Banner> {
  TestBed.configureTestingModule({providers});
  return TestBed.createComponent(Banner);
}
```

### `nativeElement`

Access the component's DOM element via `fixture.nativeElement`:

```typescript
it('should contain "banner works!"', () => {
  const bannerElement: HTMLElement = fixture.nativeElement;
  expect(bannerElement.textContent).toContain('banner works!');
});
```

The `nativeElement` property has an `any` type because Angular runs on multiple platforms (browser, server, node). In browser environments, it's always an `HTMLElement`.

Use standard HTML methods like `querySelector`:

```typescript
it('should have <p> with "banner works!"', () => {
  const bannerElement: HTMLElement = fixture.nativeElement;
  const p = bannerElement.querySelector('p')!;
  expect(p.textContent).toEqual('banner works!');
});
```

### `DebugElement`

The `DebugElement` abstraction provides a platform-agnostic way to interact with components:

```typescript
const bannerDe: DebugElement = fixture.debugElement;
const bannerEl: HTMLElement = bannerDe.nativeElement;
const p = bannerEl.querySelector('p')!;
expect(p.textContent).toEqual('banner works!');
```

`DebugElement` wraps platform-specific element objects, allowing tests to work across different environments (browser, server, etc.).

### `By.css()` for Platform-Independent Queries

For maximum compatibility across platforms, use `DebugElement.query()` with `By.css()`:

```typescript
import {By} from '@angular/platform-browser';

it('should find the <p> with fixture.debugElement.query(By.css)', () => {
  const bannerDe: DebugElement = fixture.debugElement;
  const paragraphDe = bannerDe.query(By.css('p'));
  const p: HTMLElement = paragraphDe.nativeElement;
  expect(p.textContent).toEqual('banner works!');
});
```

Key points:

- `By.css()` uses standard CSS selectors
- `query()` returns a `DebugElement` for the matched element
- Unwrap results to access the native element

For browser-only testing with straightforward assertions, standard HTML methods like `querySelector()` are often clearer and more direct than CSS selector-based approaches.
