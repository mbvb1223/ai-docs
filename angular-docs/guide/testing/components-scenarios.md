# Component Testing Scenarios
> Source: https://angular.dev/guide/testing/components-scenarios

## Overview

This guide covers practical component testing approaches in Angular, addressing common real-world scenarios including data binding, service dependencies, async operations, and routing.

## Component Binding

### Initial Setup Challenge

The `Banner` component displays dynamic content via signal-based interpolation:

```typescript
export class Banner {
  title = signal('Test Tour of Heroes');
}
```

A critical testing principle: **`TestBed.createComponent()` does not automatically trigger change detection**. Initial binding requires explicit synchronization using `fixture.whenStable()`.

### Testing Data Flow

```typescript
it('should display original title', async () => {
  await fixture.whenStable();
  expect(h1.textContent).toContain(component.title());
});
```

This delayed change detection pattern is intentional -- it allows test code to modify component state before Angular initiates binding and lifecycle hooks.

### Dynamic Updates

```typescript
it('should display a different test title', async () => {
  component.title.set('Test Title');
  await fixture.whenStable();
  expect(h1.textContent).toContain('Test Title');
});
```

### Signal Input Binding

Dynamic bindings to inputs and outputs can use helper functions:

```typescript
const fixture = TestBed.createComponent(ValueDisplay, {
  bindings: [
    inputBinding('value', value),
    outputBinding('valueChange', () => (/* ... */)),
  ],
});
```

### User Input Simulation

When testing form inputs, dispatching events is essential -- Angular doesn't monitor property assignments:

```typescript
it('should convert hero name to Title Case', async () => {
  const nameInput: HTMLInputElement =
    hostElement.querySelector('input')!;
  nameInput.value = 'quick BROWN  fOx';
  nameInput.dispatchEvent(new Event('input'));
  await fixture.whenStable();
  expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
});
```

## Component Dependencies

### Service Injection Pattern

For components with injected services, use `TestBed.inject()` to access the test instance:

```typescript
let userAuth: UserAuthentication;

beforeEach(() => {
  fixture = TestBed.createComponent(Welcome);
  userAuth = TestBed.inject(UserAuthentication);
  el = fixture.nativeElement.querySelector('.welcome');
});
```

### Service Interaction Tests

```typescript
it('should welcome the user', async () => {
  await fixture.whenStable();
  expect(el.textContent).toContain('Welcome');
  expect(el.textContent).toContain('Test User');
});

it('should welcome "Bubba"', async () => {
  userAuth.user.set({name: 'Bubba'});
  await fixture.whenStable();
  expect(el.textContent).toContain('Bubba');
});
```

### Alternative: Component-Level Injection

When `TestBed.inject()` doesn't work (e.g., multiple providers at different levels):

```typescript
userAuth = fixture.debugElement.injector.get(UserAuthentication);
```

## Async Service Testing

### HTTP Mocking with HttpTestingController

For services using `HttpClient`, mock HTTP responses at the service level:

```typescript
TestBed.configureTestingModule({
  providers: [provideHttpClientTesting()],
});

const httpMock = TestBed.inject(HttpTestingController);
httpMock.expectOne('api/endpoint').flush(mockData);
```

### Stubbed Service Implementation

Create stub classes for async operations:

```typescript
class TwainQuotesStub implements TwainQuotes {
  private testQuote = 'Test Quote';
  getQuote() {
    return of(this.testQuote);
  }
}

beforeEach(async () => {
  TestBed.configureTestingModule({
    providers: [{provide: TwainQuotes, useClass: TwainQuotesStub}],
  });
});
```

### Vitest Fake Timers

For controlled async behavior:

```typescript
it('should display error when service fails', async () => {
  vi.useFakeTimers();
  const fixture = TestBed.createComponent(TwainComponent);
  await vi.runAllTimersAsync();
  expect(fixture.nativeElement.querySelector('.error')!
    .textContent).toMatch(/test failure/);
  vi.useRealTimers();
});
```

### Observable Testing with Subjects

```typescript
class MockTwainQuotes implements TwainQuotes {
  private subject = new Subject<string>();
  getQuote() {
    return this.subject.asObservable();
  }
  emit(val: string) {
    this.subject.next(val);
  }
}
```

## Input/Output Components

### Standalone Testing with `setInput()`

```typescript
beforeEach(async () => {
  fixture = TestBed.createComponent(DashboardHero);
  comp = fixture.componentInstance;
  expectedHero = {id: 42, name: 'Test Name'};
  fixture.componentRef.setInput('hero', expectedHero);
  await fixture.whenStable();
});
```

### Output Event Verification

```typescript
it('should raise selected event when clicked', () => {
  let selectedHero: Hero | undefined;
  comp.selected.subscribe((hero: Hero) => (selectedHero = hero));
  heroDe.triggerEventHandler('click');
  expect(selectedHero).toBe(expectedHero);
});
```

### Click Helper Utility

A reusable helper abstracts click behavior:

```typescript
export function click(
  el: DebugElement | HTMLElement,
  eventObj: any = {button: 0},
): void {
  if (el instanceof HTMLElement) {
    el.click();
  } else {
    el.triggerEventHandler('click', eventObj);
  }
}
```

## Test Host Pattern

For testing components within their intended parent context:

```typescript
@Component({
  template: `<dashboard-hero
    [hero]="hero"
    (selected)="onSelected($event)" />`,
})
class TestHost {
  hero: Hero = {id: 42, name: 'Test Name'};
  selectedHero: Hero | undefined;
  onSelected(hero: Hero) {
    this.selectedHero = hero;
  }
}

beforeEach(async () => {
  fixture = TestBed.createComponent(TestHost);
  testHost = fixture.componentInstance;
  await fixture.whenStable();
});
```

## Routing Components

### Router Testing Setup

```typescript
beforeEach(async () => {
  TestBed.configureTestingModule({
    providers: [
      provideRouter([{path: '**', component: Dashboard}]),
      provideHttpClientTesting(),
    ],
  });
  harness = await RouterTestingHarness.create();
  comp = await harness.navigateByUrl('/', Dashboard);
});
```

### Navigation Verification

```typescript
it('should navigate to hero detail', async () => {
  const heroDe = harness.routeDebugElement!.query(By.css('hero'));
  heroDe.triggerEventHandler('selected', comp.heroes[0]);
  const id = comp.heroes[0].id;
  expect(TestBed.inject(Router).url).toEqual(`/heroes/${id}`);
});
```

## Routed Components

Components receiving route parameters through `ActivatedRoute`:

```typescript
export class HeroDetail {
  private route = inject(ActivatedRoute);

  constructor() {
    this.route.paramMap
      .pipe(takeUntilDestroyed())
      .subscribe((pmap) => this.getHero(pmap.get('id')));
  }
}
```

## Shallow Component Testing

### Stub Approach

Replace nested components with minimal stubs:

```typescript
@Component({selector: 'app-banner', template: ''})
class BannerStub {}

TestBed.configureTestingModule({...}).overrideComponent(App, {
  set: {
    imports: [RouterLink, BannerStub, RouterOutletStub, WelcomeStub],
  },
});
```

### NO_ERRORS_SCHEMA Approach

Suppress unrecognized element errors:

```typescript
TestBed.configureTestingModule({...}).overrideComponent(App, {
  set: {
    imports: [],
    schemas: [NO_ERRORS_SCHEMA],
  },
});
```

**Trade-off**: Schema suppression provides simpler setup but sacrifices compiler error detection for misspelled selectors or missing dependencies.
