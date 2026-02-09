# Whitespace in Templates
> Source: https://angular.dev/guide/templates/whitespace

## Overview

Angular templates don't preserve whitespace that the framework considers unnecessary. This optimization occurs in two primary scenarios: whitespace between elements and collapsible whitespace within text content.

## Whitespace Between Elements

### Problem

Developers typically format templates with newlines and indentation for readability:

```html
<section>
  <h3>User profile</h3>
  <label>
    User name
    <input />
  </label>
</section>
```

This formatting introduces numerous whitespace characters between elements that browsers would typically render as text nodes.

### Angular's Approach

By ignoring this whitespace between elements, Angular performs less work when rendering the template on the page, improving overall performance. Angular strips these unnecessary whitespace characters during template compilation, reducing DOM overhead and enhancing rendering efficiency.

## Collapsible Whitespace Inside Text

### Browser Behavior

Web browsers automatically collapse consecutive whitespace characters to a single space. For example:

```html
<!-- Template source -->
<p>Hello         world</p>

<!-- Browser display -->
<p>Hello world</p>
```

### Angular's Optimization

Angular avoids sending unnecessary whitespace to the browser by collapsing multiple consecutive whitespace characters to a single character during template compilation.

## Preserving Whitespace

### Using preserveWhitespaces Property

When necessary, you can enable whitespace preservation in the `@Component` decorator:

```typescript
@Component({
  /* ... */,
  preserveWhitespaces: true,
  template: `
    <p>Hello         world</p>
  `
})
```

### Performance Consideration

Avoid setting this option unless absolutely necessary. Preserving whitespace can cause Angular to produce significantly more nodes while rendering, slowing down your application.

### Alternative: &ngsp; Entity

Angular provides a special HTML entity `&ngsp;` that produces a single preserved space character in compiled output, offering a lightweight alternative to full whitespace preservation.
