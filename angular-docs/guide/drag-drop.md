# Angular Drag and Drop
> Source: https://angular.dev/guide/drag-drop

## Overview

Angular's drag and drop system enables developers to quickly implement interactive interfaces with these core features:

- Free dragging of elements
- Reorderable lists of draggable items
- Transfer elements between connected lists
- Dragging animations
- Axis-locked or boundary-restricted movement
- Custom drag handles, previews, and placeholders

The functionality derives from the Component Dev Kit (CDK), a collection of behavior primitives for building components.

## Installation and Setup

### Installing the CDK

Use the Angular CLI to add the CDK package:

```bash
ng add @angular/cdk
```

### Importing Directives

Import required directives in your component:

```typescript
import {Component} from '@angular/core';
import {CdkDrag} from '@angular/cdk/drag-drop';

@Component({
  selector: 'drag-drop-example',
  templateUrl: 'drag-drop-example.html',
  imports: [CdkDrag],
})
export class DragDropExample {}
```

## Creating Draggable Elements

Add the `cdkDrag` directive to make any element draggable:

```html
<div class="example-box" cdkDrag>Drag me around</div>
```

By default, all elements with `cdkDrag` support free movement in all directions.

## Reorderable Lists

Use `cdkDropList` to group draggable elements into an automatically reordering collection:

```html
<div cdkDropList class="example-list" (cdkDropListDropped)="drop($event)">
  @for (movie of movies; track movie) {
    <div class="example-box" cdkDrag>{{ movie }}</div>
  }
</div>
```

Listen to the `cdkDropListDropped` event and update your data model:

```typescript
import {CdkDrag, CdkDragDrop, CdkDropList, moveItemInArray} from '@angular/cdk/drag-drop';

drop(event: CdkDragDrop<string[]>) {
  moveItemInArray(this.movies, event.previousIndex, event.currentIndex);
}
```

## Transferring Elements Between Lists

Connect multiple drop lists using `cdkDropListConnectedTo`:

```html
<div cdkDropList #todoList="cdkDropList"
     [cdkDropListConnectedTo]="[doneList]"
     (cdkDropListDropped)="drop($event)">
  @for (item of todo; track item) {
    <div class="example-box" cdkDrag>{{ item }}</div>
  }
</div>

<div cdkDropList #doneList="cdkDropList"
     [cdkDropListConnectedTo]="[todoList]"
     (cdkDropListDropped)="drop($event)">
  @for (item of done; track item) {
    <div class="example-box" cdkDrag>{{ item }}</div>
  }
</div>
```

Handle drops by checking whether items moved within a list or transferred between lists:

```typescript
drop(event: CdkDragDrop<string[]>) {
  if (event.previousContainer === event.container) {
    moveItemInArray(event.container.data, event.previousIndex, event.currentIndex);
  } else {
    transferArrayItem(
      event.previousContainer.data,
      event.container.data,
      event.previousIndex,
      event.currentIndex,
    );
  }
}
```

### Automatic Connections with Groups

Use `cdkDropListGroup` to automatically connect all drop lists within a container:

```html
<div cdkDropListGroup>
  @for (list of lists; track list) {
    <div cdkDropList [cdkDropListData]="list">
      <!-- Items here -->
    </div>
  }
</div>
```

### Selective Dropping with Predicates

Control which elements can drop into specific containers using `cdkDropListEnterPredicate`:

```html
<div cdkDropList [cdkDropListEnterPredicate]="evenPredicate">
  @for (number of even; track number) {
    <div cdkDrag [cdkDragData]="number">{{ number }}</div>
  }
</div>
```

```typescript
evenPredicate(item: CdkDrag<number>) {
  return item.data % 2 === 0;
}
```

## Attaching Data

Associate arbitrary data with draggable and droppable elements:

```html
@for (list of lists; track list) {
  <div cdkDropList [cdkDropListData]="list" (cdkDropListDropped)="drop($event)">
    @for (item of list; track item) {
      <div cdkDrag [cdkDragData]="item"></div>
    }
  </div>
}
```

Events fired from these directives include this data, enabling easy identification of interaction origins.

## Customizing Drag Behavior

### Custom Drag Handles

Restrict dragging to specific handle elements using `cdkDragHandle`:

```html
<div class="example-box" cdkDrag>
  I can only be dragged using the handle
  <div class="example-handle" cdkDragHandle>
    <!-- Handle content -->
  </div>
</div>
```

### Custom Previews

Override the default preview element with `*cdkDragPreview`:

```html
<div cdkDropList class="example-list" (cdkDropListDropped)="drop($event)">
  @for (movie of movies; track movie) {
    <div class="example-box" cdkDrag>
      {{ movie.title }}
      <img *cdkDragPreview [src]="movie.poster" [alt]="movie.title" />
    </div>
  }
</div>
```

Use `matchSize="true"` to match the preview size to the original element.

### Preview Container Positioning

Control where the preview element is inserted using `cdkDragPreviewContainer`:

| Value | Behavior |
|-------|----------|
| `global` | Inserts preview into body (default); avoids clipping |
| `parent` | Inserts preview into parent element; inherits styles |
| `ElementRef` or `HTMLElement` | Inserts into specified element |

### Custom Placeholders

Replace the default placeholder with `*cdkDragPlaceholder`:

```html
<div cdkDropList class="example-list" (cdkDropListDropped)="drop($event)">
  @for (movie of movies; track movie) {
    <div class="example-box" cdkDrag>
      <div class="example-custom-placeholder" *cdkDragPlaceholder></div>
      {{ movie }}
    </div>
  }
</div>
```

### Setting Root Elements

Use `cdkDragRootElement` to make elements draggable when direct access isn't available:

```html
<div class="example-dialog-content" cdkDrag cdkDragRootElement=".cdk-overlay-pane">
  Drag the dialog around!
</div>
```

### Free Drag Positioning

Programmatically set an element's position using `cdkDragFreeDragPosition`:

```html
<div class="example-box" cdkDrag [cdkDragFreeDragPosition]="dragPosition">
  Drag me around
</div>
```

```typescript
dragPosition = {x: 0, y: 0};

changePosition() {
  this.dragPosition = {x: this.dragPosition.x + 50, y: this.dragPosition.y + 50};
}
```

### Movement Boundaries

Restrict dragging within a boundary element using `cdkDragBoundary`:

```html
<div class="example-boundary">
  <div class="example-box" cdkDragBoundary=".example-boundary" cdkDrag>
    I can only be dragged within the dotted container
  </div>
</div>
```

### Axis Locking

Restrict movement to a single axis:

```html
<div class="example-box" cdkDragLockAxis="y" cdkDrag>
  I can only be dragged up/down
</div>

<div class="example-box" cdkDragLockAxis="x" cdkDrag>
  I can only be dragged left/right
</div>
```

For multiple draggable elements within a drop list, use `cdkDropListLockAxis` instead.

## Injection Tokens

Several injection tokens allow referencing directive instances:

- `CDK_DROP_LIST` - Reference `cdkDropList` instances
- `CDK_DROP_LIST_GROUP` - Reference `cdkDropListGroup` instances
- `CDK_DRAG_HANDLE` - Reference `cdkDragHandle` instances
- `CDK_DRAG_PREVIEW` - Reference `cdkDragPreview` instances
- `CDK_DRAG_PLACEHOLDER` - Reference `cdkDragPlaceholder` instances
- `CDK_DRAG_CONFIG` - Modify global drag configuration

See the dependency injection guide for implementation details.

## API Reference

For complete API documentation, refer to the Angular CDK drag and drop API reference.
