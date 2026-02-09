# Programmatically Rendering Components
> Source: https://angular.dev/guide/components/programmatic-rendering

## Overview

Beyond template-based component usage, Angular allows dynamic component rendering through code. This approach is essential when component selection depends on runtime conditions or isn't known in advance.

**Two primary methods exist:**
1. Template-based: `NgComponentOutlet` directive
2. Code-based: `ViewContainerRef` API

> For lazy-loading use-cases (for example if you want to delay loading of a heavy component), consider using the built-in `@defer` feature instead.

## Using NgComponentOutlet

`NgComponentOutlet` is a structural directive enabling dynamic component rendering within templates.

### Basic Implementation

```typescript
@Component({/*...*/})
export class AdminBio { /* ... */ }

@Component({/*...*/})
export class StandardBio { /* ... */ }

@Component({
  ...,
  template: `
    <p>Profile for {{user.name}}</p>
    <ng-container *ngComponentOutlet="getBioComponent()" />
  `
})
export class CustomDialog {
  user = input.required<User>();
  getBioComponent() {
    return this.user().isAdmin ? AdminBio : StandardBio;
  }
}
```

### Passing Inputs

Input values transfer to dynamically rendered components via `ngComponentOutletInputs`:

```typescript
@Component({
  selector: 'user-greeting',
  template: `
    <div>
      <p>User: {{ username() }}</p>
      <p>Role: {{ role() }}</p>
    </div>
  `,
})
export class UserGreeting {
  username = input.required<string>();
  role = input('guest');
}

@Component({
  selector: 'profile-view',
  imports: [NgComponentOutlet],
  template: `
    <ng-container
      *ngComponentOutlet="greetingComponent; inputs: greetingInputs()"
    />
  `,
})
export class ProfileView {
  greetingComponent = UserGreeting;
  greetingInputs = signal({username: 'ngAwesome', role: 'admin'});
}
```

Inputs update reactively when the signal changes, maintaining synchronization.

### Content Projection

Use `ngComponentOutletContent` to pass projected content to dynamic components:

```typescript
@Component({
  selector: 'card-wrapper',
  template: `
    <div class="card">
      <ng-content />
    </div>
  `,
})
export class CardWrapper {}

@Component({
  imports: [NgComponentOutlet],
  template: `
    <ng-container
      *ngComponentOutlet="cardComponent; content: cardContent()"
    />
    <ng-template #contentTemplate>
      <h3>Dynamic Content</h3>
      <p>This content is projected into the card.</p>
    </ng-template>
  `,
})
export class DynamicCard {
  private vcr = inject(ViewContainerRef);
  cardComponent = CardWrapper;
  private contentTemplate = viewChild<TemplateRef<unknown>>('contentTemplate');

  cardContent = computed(() => {
    const template = this.contentTemplate();
    if (!template) return [];
    return [this.vcr.createEmbeddedView(template).rootNodes];
  });
}
```

**Important:** Native DOM APIs bypass hydration compatibility. Use Angular APIs or add `ngSkipHydration` to resolve NG0503 errors.

### Custom Injectors

Provide component-specific services through custom injectors:

```typescript
export const THEME_DATA = new InjectionToken<string>('THEME_DATA', {
  factory: () => 'light',
});

@Component({
  selector: 'themed-panel',
  template: `<div [class]="theme">...</div>`,
})
export class ThemedPanel {
  theme = inject(THEME_DATA);
}

@Component({
  selector: 'dynamic-panel',
  imports: [NgComponentOutlet],
  template: `
    <ng-container
      *ngComponentOutlet="panelComponent; injector: customInjector"
    />
  `,
})
export class DynamicPanel {
  panelComponent = ThemedPanel;
  customInjector = Injector.create({
    providers: [{provide: THEME_DATA, useValue: 'dark'}],
  });
}
```

### Accessing Component Instances

Reference dynamically created components using `exportAs`:

```typescript
@Component({
  selector: 'counter',
  template: `<p>Count: {{ count() }}</p>`,
})
export class Counter {
  count = signal(0);
  increment() {
    this.count.update((c) => c + 1);
  }
}

@Component({
  imports: [NgComponentOutlet],
  template: `
    <ng-container
      [ngComponentOutlet]="counterComponent"
      #outlet="ngComponentOutlet"
    />
    <button (click)="outlet.componentInstance?.increment()">
      Increment
    </button>
  `,
})
export class CounterHost {
  counterComponent = Counter;
}
```

**Note:** `componentInstance` is null before rendering completes.

## Using ViewContainerRef

A **view container** represents a location in the component tree capable of holding content. Components inject `ViewContainerRef` to access the container at their location.

The `createComponent()` method dynamically instantiates and renders components, appending them as siblings to the requesting component or directive.

### Basic Example

```typescript
@Component({
  selector: 'leaf-content',
  template: `This is the leaf content`,
})
export class LeafContent {}

@Component({
  selector: 'outer-container',
  template: `
    <p>This is the start of the outer container</p>
    <inner-item />
    <p>This is the end of the outer container</p>
  `,
})
export class OuterContainer {}

@Component({
  selector: 'inner-item',
  template: `<button (click)="loadContent()">Load content</button>`,
})
export class InnerItem {
  private viewContainer = inject(ViewContainerRef);

  loadContent() {
    this.viewContainer.createComponent(LeafContent);
  }
}
```

**Resulting DOM structure:**
```html
<outer-container>
  <p>This is the start of the outer container</p>
  <inner-item>
    <button>Load content</button>
  </inner-item>
  <leaf-content>This is the leaf content</leaf-content>
  <p>This is the end of the outer container</p>
</outer-container>
```

## Lazy-Loading Components

For deferred loading scenarios not covered by `@defer`, use dynamic imports:

```typescript
@Component({
  ...,
  template: `
    <section>
      <h2>Basic settings</h2>
      <basic-settings />
    </section>
    <section>
      <h2>Advanced settings</h2>
      @if(!advancedSettings) {
        <button (click)="loadAdvanced()">
          Load advanced settings
        </button>
      }
      <ng-container *ngComponentOutlet="advancedSettings" />
    </section>
  `
})
export class AdminSettings {
  advancedSettings: {new(): AdvancedSettings} | undefined;

  async loadAdvanced() {
    const { AdvancedSettings } = await import('path/to/advanced_settings.js');
    this.advancedSettings = AdvancedSettings;
  }
}
```

## Advanced Binding and Configuration

Both `createComponent()` and `ViewContainerRef.createComponent()` accept a `bindings` array for declarative input/output configuration using helper functions.

### With ViewContainerRef

```typescript
@Component({
  selector: 'app-warning',
  template: `
    @if (isExpanded()) {
      <section>
        <p>Warning: Action needed!</p>
        <button (click)="close.emit(true)">Close</button>
      </section>
    }
  `,
})
export class AppWarning {
  readonly canClose = input.required<boolean>();
  readonly isExpanded = model<boolean>();
  readonly close = output<boolean>();
}

@Component({
  template: `<ng-container #container />`,
})
export class Host {
  private vcr = inject(ViewContainerRef);
  readonly canClose = signal(true);
  readonly isExpanded = signal(true);

  showWarning() {
    const compRef = this.vcr.createComponent(AppWarning, {
      bindings: [
        inputBinding('canClose', this.canClose),
        twoWayBinding('isExpanded', this.isExpanded),
        outputBinding<boolean>('close', (confirmed) => {
          console.log('Closed with result:', confirmed);
        }),
      ],
      directives: [
        FocusTrap,
        {type: ThemeDirective, bindings: [inputBinding('theme', () => 'warning')]},
      ],
    });
  }
}
```

### Popup Outside View Hierarchy

For overlays and modals, use standalone `createComponent()` with `hostElement`:

```typescript
@Injectable({providedIn: 'root'})
export class PopupService {
  private readonly injector = inject(EnvironmentInjector);
  private readonly appRef = inject(ApplicationRef);

  show(message: string) {
    const host = document.createElement('popup-host');

    const ref = createComponent(Popup, {
      environmentInjector: this.injector,
      hostElement: host,
      bindings: [
        inputBinding('message', () => message),
        outputBinding('closed', () => {
          document.body.removeChild(host);
          this.appRef.detachView(ref.hostView);
          ref.destroy();
        }),
      ],
    });

    this.appRef.attachView(ref.hostView);
    document.body.appendChild(host);
  }
}
```

**Key differences:**
- `ViewContainerRef.createComponent`: Inserts into existing view hierarchy
- Standalone `createComponent`: Provides explicit placement control, suitable for portals and overlays
