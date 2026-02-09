# Testing Services
> Source: https://angular.dev/guide/testing/services

## Overview

Services are among the simplest files to unit test. You can write tests that verify services work as intended through both synchronous and asynchronous approaches.

## Basic Service Testing (Without Angular Utilities)

```typescript
describe('ValueService', () => {
  let service: ValueService;

  beforeEach(() => {
    // Only works if the service doesn't rely on Angular inject()
    service = new ValueService();
  });

  it('getValue should return real value', () => {
    expect(service.getValue()).toBe('real value');
  });

  it('getObservableValue should return value from observable', async () => {
    const value = await new Promise<string>((resolve) => {
      service.getObservableValue().subscribe(resolve);
    });
    expect(value).toBe('observable value');
  });

  it('getPromiseValue should return value from a promise', async () => {
    const value = await service.getPromiseValue();
    expect(value).toBe('promise value');
  });
});
```

## Testing Services with TestBed

### Overview of Dependency Injection in Testing

Angular applications use dependency injection to create services. Services may depend on other services, and Angular's DI system handles this hierarchy. As a service consumer, you don't manage these dependencies, but as a tester, you should understand at least the first level of dependencies.

The `TestBed` utility allows you to leverage Angular's DI system during testing rather than manually managing service creation and constructor arguments.

### Angular TestBed Basics

The `TestBed` represents the most crucial Angular testing utility. It constructs a dynamically-generated Angular test module that emulates an `@NgModule`.

**Basic Setup:**

```typescript
let service: ValueService;

beforeEach(() => {
  TestBed.configureTestingModule({providers: [ValueService]});
});
```

**Injecting Services:**

Inject services within tests using `TestBed.inject()`:

```typescript
it('should use ValueService', () => {
  service = TestBed.inject(ValueService);
  expect(service.getValue()).toBe('real value');
});
```

**In Setup:**

```typescript
beforeEach(() => {
  TestBed.configureTestingModule({providers: [ValueService]});
  service = TestBed.inject(ValueService);
});
```

## Testing Services with Dependencies

When a service depends on other services, provide mocks in the `providers` array:

```typescript
let masterService: MainService;
let valueServiceSpy: Mocked<ValueService>;

beforeEach(() => {
  const spy: Mocked<ValueService> = {getValue: vi.fn()};

  TestBed.configureTestingModule({
    providers: [MainService, {provide: ValueService, useValue: spy}],
  });

  masterService = TestBed.inject(MainService);
  valueServiceSpy = TestBed.inject(ValueService) as Mocked<ValueService>;
});
```

**Using Spy Objects in Tests:**

```typescript
it('getValue should return stubbed value from a spy', () => {
  const stubValue = 'stub value';
  valueServiceSpy.getValue.mockReturnValue(stubValue);

  expect(masterService.getValue(), 'service returned stub value').toBe(stubValue);
  expect(valueServiceSpy.getValue, 'spy method was called once').toHaveBeenCalledTimes(1);
  expect(valueServiceSpy.getValue.mock.results.at(-1)?.value).toBe(stubValue);
});
```

## Testing HTTP Services

Services that depend on `HttpClient` have dedicated testing guidance available in the HTTP testing guide. Refer to the separate documentation for handling HTTP-based service tests.
