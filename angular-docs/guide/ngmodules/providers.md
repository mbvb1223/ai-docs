# NgModule Providers
> Source: https://angular.dev/guide/ngmodules/providers
>
> Note: This page redirects to the main NgModules guide. The providers content is consolidated within the NgModules documentation.

## NgModule Providers

An NgModule can specify `providers` for injected dependencies. These are available to:

- Any standalone component, directive, or pipe that imports the NgModule
- The declarations and providers of other NgModules that import this NgModule

### Basic Provider Configuration

```typescript
@NgModule({
  imports: [PopupTrigger, SelectionIndicator],
  declarations: [CustomMenu, CustomMenuItem],
  providers: [OverlayManager],
})
export class CustomMenuModule {}

@NgModule({
  imports: [CustomMenuModule],
  declarations: [UserProfile],
  providers: [UserDataClient],
})
export class UserProfileModule {}
```

### Injection Availability

- `CustomMenu` and `CustomMenuItem` can inject `OverlayManager` (declared in the providing module)
- `UserProfile` can inject `OverlayManager` (its module imports the providing module)
- `UserDataClient` can inject `OverlayManager` (its module imports the providing module)

### The forRoot and forChild Pattern

Some NgModules define static `forRoot()` methods accepting configuration and returning provider arrays. This convention indicates providers should be added exclusively at the application root during bootstrap.

```typescript
bootstrapApplication(MyApplicationRoot, {
  providers: [CustomMenuModule.forRoot(/* config */)],
});
```

Similarly, `forChild()` indicates providers are intended for components within the application hierarchy:

```typescript
@Component({
  providers: [CustomMenuModule.forChild(/* config */)],
})
export class UserProfile { }
```

### Key Points

- Providers specified in an NgModule are available to all declarations within that module and to any other NgModule that imports it
- The `forRoot` convention helps avoid duplicate provider instances in lazy-loaded modules
- The `forChild` convention is used for feature modules that need their own provider configuration
- The Angular team recommends using standalone components with `providedIn: 'root'` for new code instead of NgModule providers

For the complete NgModules documentation, see the [NgModules guide](../ngmodules/index.md).
