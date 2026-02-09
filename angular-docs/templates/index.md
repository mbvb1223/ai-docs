# Angular Template Syntax Guide
> Source: https://angular.dev/guide/templates

## Overview

Angular templates are HTML chunks enhanced with special syntax to leverage Angular's features. Templates automatically keep pages updated as data changes and are found in either the `template` property of `*.ts` files or `*.html` files.

## How Templates Work

Templates are built on HTML syntax with additional capabilities including data binding, event listening, variables, and built-in template functions. Angular compiles templates into JavaScript to build internal application understanding, enabling automatic rendering optimizations.

### Key Differences from Standard HTML

Several distinctions exist between Angular templates and standard HTML:

- **Comments**: Template source code comments don't appear in rendered output
- **Self-closing elements**: Component and directive elements can be self-closed (e.g., `<UserProfile />`)
- **Special characters**: Attributes with `[]`, `()`, etc. have special Angular meanings (see binding and event listener documentation)
- **@ character**: Has special significance for dynamic behavior like control flow. Literal @ symbols use HTML entity codes (`&commat;` or `&#64;`)
- **Whitespace handling**: Angular ignores and collapses unnecessary whitespace characters
- **Comment nodes**: Angular may add comment nodes as dynamic content placeholders, which developers can ignore
- **Script tags**: Angular does not support `<script>` elements in templates (see Security documentation)

## Next Steps

The documentation provides comprehensive guides on related topics:

| Topic | Purpose |
|-------|---------|
| [Binding dynamic text, properties, and attributes](binding.md) | Bind dynamic data to text, properties and attributes |
| [Adding event listeners](event-listeners.md) | Respond to template events |
| [Two-way binding](two-way-binding.md) | Simultaneously bind values and propagate changes |
| [Control flow](control-flow.md) | Conditionally show, hide, and repeat elements |
| [Pipes](pipes.md) | Transform data declaratively |
| [Slotting child content with ng-content](ng-content.md) | Control component content rendering |
| [Create template fragments with ng-template](ng-template.md) | Declare template fragments |
| [Grouping elements with ng-container](ng-container.md) | Group multiple elements or mark rendering locations |
| [Variables in templates](variables.md) | Learn variable declarations |
| [Deferred loading with @defer](defer.md) | Create deferrable views |
| [Expression syntax](expression-syntax.md) | Angular vs. JavaScript expression differences |
| [Whitespace in templates](whitespace.md) | Understand whitespace handling |
