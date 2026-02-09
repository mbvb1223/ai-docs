# Complex Animation Sequences
> Source: https://angular.dev/guide/animations/complex-sequences
> (Redirects to legacy animations: https://angular.dev/guide/legacy-animations/complex-sequences)

> **DEPRECATION NOTICE:** The `@angular/animations` package is deprecated. Angular recommends using native CSS with `animate.enter` and `animate.leave` for new code. See the migration guide for transitioning to pure CSS animations.

## Overview

This guide covers coordinated animation sequences in Angular, allowing developers to animate multiple elements entering and leaving a page, either in parallel or sequentially.

## Core Animation Functions

| Function | Purpose |
|----------|---------|
| `query()` | Locates one or more inner HTML elements |
| `stagger()` | Applies cascading delays to multi-element animations |
| `group()` | Runs multiple animation steps simultaneously |
| `sequence()` | Executes animation steps sequentially |

## The query() Function

The `query()` function finds child elements and applies animations to them. It accepts CSS selectors plus Angular-specific tokens:

- `:enter` / `:leave` -- Elements entering or leaving the DOM
- `:animating` -- Currently animating elements
- `@*` / `@triggerName` -- Elements with any or specific animation triggers
- `:self` -- The animating element itself

### Entering and Leaving Elements

Not all child elements qualify as entering/leaving; behavior can be counterintuitive. Consult the API documentation for detailed clarification and examine provided examples under the Querying tab.

## Animate Multiple Elements with query() and stagger()

The `stagger()` function creates timing gaps between animated items. This example demonstrates animating a hero list with sequential entry, slight delay, and top-to-bottom progression:

```typescript
animations: [
  trigger('pageAnimations', [
    transition(':enter', [
      query('.hero', [
        style({opacity: 0, transform: 'translateY(-100px)'}),
        stagger(30, [
          animate('500ms cubic-bezier(0.35, 0, 0.25, 1)',
            style({opacity: 1, transform: 'none'})),
        ]),
      ]),
    ]),
  ]),
]
```

**Process:**

1. Use `query()` to target entering elements matching specific criteria
2. Apply `style()` to set initial styling -- make transparent and offset via transform
3. Use `stagger()` to delay each animation by 30 milliseconds
4. Animate each element for 0.5 seconds with custom easing, fading and transforming simultaneously

## Parallel Animation with group()

The `group()` function runs multiple animation steps in parallel rather than sequentially. This enables animating different CSS properties of the same element with distinct easing functions.

> **Key Point:** `group()` organizes animation steps, not animated elements.

Example applying `group()` to both `:enter` and `:leave` transitions with separate timing:

```typescript
animations: [
  trigger('flyInOut', [
    state(
      'in',
      style({
        width: '*',
        transform: 'translateX(0)',
        opacity: 1,
      }),
    ),
    transition(':enter', [
      style({width: 10, transform: 'translateX(50px)', opacity: 0}),
      group([
        animate(
          '0.3s 0.1s ease',
          style({
            transform: 'translateX(0)',
            width: '*',
          }),
        ),
        animate(
          '0.3s ease',
          style({
            opacity: 1,
          }),
        ),
      ]),
    ]),
    transition(':leave', [
      group([
        animate(
          '0.3s ease',
          style({
            transform: 'translateX(50px)',
            width: 10,
          }),
        ),
        animate(
          '0.3s 0.2s ease',
          style({
            opacity: 0,
          }),
        ),
      ]),
    ]),
  ]),
],
```

## Sequential vs. Parallel Animations

Complex animations may combine multiple simultaneous or sequential effects. Use `group()` for parallel execution and `sequence()` for sequential execution. Within sequences, use `style()` for immediate application or `animate()` for time-based application.

## Filter Animation Example

A practical implementation filters a hero list in real-time. As users type search text, elements exit the page; as they delete characters, elements re-enter progressively.

**HTML template with `filterAnimation` trigger:**

```html
<label for="search">Search heroes: </label>
<input
  type="text"
  id="search"
  #criteria
  (input)="updateCriteria(criteria.value)"
  placeholder="Search heroes"/>
<ul class="heroes" [@filterAnimation]="heroesTotal">
  @for (hero of heroes; track hero) {
    <li class="hero">
      <div class="inner">
        <span class="badge">{{ hero.id }}</span>
        <span class="name">{{ hero.name }}</span>
      </div>
    </li>
  }
</ul>
```

**Component implementation with three transitions:**

```typescript
@Component({
  animations: [
    trigger('filterAnimation', [
      transition(':enter, * => 0, * => -1', []),
      transition(':increment', [
        query(
          ':enter',
          [
            style({opacity: 0, width: 0}),
            stagger(50, [animate('300ms ease-out', style({opacity: 1, width: '*'}))]),
          ],
          {optional: true},
        ),
      ]),
      transition(':decrement', [
        query(':leave', [stagger(50, [animate('300ms ease-out', style({opacity: 0, width: 0}))])]),
      ]),
    ]),
  ],
})
export class HeroListPage implements OnInit {
  heroesTotal = -1;

  get heroes() {
    return this._heroes;
  }

  private _heroes: Hero[] = [];

  ngOnInit() {
    this._heroes = HEROES;
  }

  updateCriteria(criteria: string) {
    criteria = criteria ? criteria.trim() : '';
    this._heroes = HEROES.filter((hero) =>
      hero.name.toLowerCase().includes(criteria.toLowerCase()),
    );
    const newTotal = this.heroes.length;
    if (this.heroesTotal !== newTotal) {
      this.heroesTotal = newTotal;
    } else if (!criteria) {
      this.heroesTotal = -1;
    }
  }
}
```

**Behavior:**

- Skips animations on initial page load or navigation (animation only works on existing DOM elements)
- Filters heroes based on search input
- Sets exiting elements' opacity and width to zero
- Animates entering elements over 300 milliseconds with default width and opacity
- Staggers multiple entering/leaving elements with 50-millisecond delays from top to bottom

## Animating Reordered List Items

Angular animates `*ngFor` list items correctly by default; however, if item order changes, animation tracking fails. Assign a `TrackByFunction` to `NgForOf` to maintain correct element tracking.

> **Critical:** Always use `TrackByFunction` when animating `*ngFor` lists with potential runtime reordering.

## Animations and View Encapsulation

Angular animations depend on DOM structure and don't directly account for view encapsulation. Components using `ViewEncapsulation.Emulated` behave identically to `ViewEncapsulation.None`.

The `query()` function can identify elements at any depth within emulated encapsulation. However, `ViewEncapsulation.ShadowDom` and `ViewEncapsulation.ExperimentalIsolatedShadowDom` hide DOM elements within `ShadowRoot`, preventing some animations from functioning properly. Avoid applying animations to views incorporating ShadowDom encapsulation.

## Animation Sequence Summary

Complex animation sequences begin with `query()` to locate inner elements, followed by `stagger()`, `group()`, and `sequence()` to control timing and execution order -- either cascading, parallel, or sequential.

## Related Resources

- [Introduction to Angular Animations](/guide/animations)
- [Transition and Triggers](/guide/animations/transition-and-triggers)
- [Reusable Animations](/guide/animations/reusable-animations)
- [Route Transition Animations](/guide/animations/route-animations)
