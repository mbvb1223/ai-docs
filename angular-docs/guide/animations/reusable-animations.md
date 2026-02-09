# Reusable Animations
> Source: https://angular.dev/guide/animations/reusable-animations
> (Redirects to legacy animations: https://angular.dev/guide/legacy-animations/reusable-animations)

> **DEPRECATION NOTICE:** The `@angular/animations` package is now deprecated. Angular recommends using native CSS with `animate.enter` and `animate.leave` for new code.

## Overview

This guide demonstrates how to create reusable animations in Angular.

## Creating Reusable Animations

To build a reusable animation, use the `animation()` function to define an animation in a separate `.ts` file and export it as a `const` variable. You can then import and reuse this animation in application components using the `useAnimation()` function.

### animations.ts Example

```typescript
import {animation, style, animate, trigger, transition, useAnimation} from '@angular/animations';

export const transitionAnimation = animation([
  style({
    height: '{{ height }}',
    opacity: '{{ opacity }}',
    backgroundColor: '{{ backgroundColor }}',
  }),
  animate('{{ time }}'),
]);
```

In this example, `transitionAnimation` becomes reusable through export. The `height`, `opacity`, `backgroundColor`, and `time` inputs are replaced at runtime.

### Exporting Animation Triggers

You can also export animation triggers separately:

```typescript
import {animation, style, animate, trigger, transition, useAnimation} from '@angular/animations';

export const triggerAnimation = trigger('openClose', [
  transition('open => closed', [
    useAnimation(transitionAnimation, {
      params: {
        height: 0,
        opacity: 1,
        backgroundColor: 'red',
        time: '1s',
      },
    }),
  ]),
]);
```

### Using Reusable Animations in Components

```typescript
import {Component, input} from '@angular/core';
import {transition, trigger, useAnimation, AnimationEvent} from '@angular/animations';
import {transitionAnimation} from './animations';

@Component({
  selector: 'app-open-close-reusable',
  animations: [
    trigger('openClose', [
      transition('open => closed', [
        useAnimation(transitionAnimation, {
          params: {
            height: 0,
            opacity: 1,
            backgroundColor: 'red',
            time: '1s',
          },
        }),
      ]),
    ]),
  ],
  templateUrl: 'open-close.html',
  styleUrls: ['open-close.css'],
})
```

## Related Resources

- [Introduction to Angular Animations](/guide/animations)
- [Transition and Triggers](/guide/animations/transition-and-triggers)
- [Complex Animation Sequences](/guide/animations/complex-sequences)
- [Route Transition Animations](/guide/animations/route-animations)
