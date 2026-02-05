# Highlight Package

## Overview

Tempest's Highlight is a server-side code highlighting package focused on performance and flexibility. Installation via Composer is straightforward:

```bash
composer require tempest/highlight
```

Basic usage involves creating a Highlighter instance and parsing code:

```php
$highlighter = new \Tempest\Highlight\Highlighter();
$code = $highlighter->parse($code, 'php');
```

## Supported Languages & Themes

The package supports numerous languages documented in its GitHub repository. Multiple CSS themes are provided, loadable either through imports or manual CSS copying.

For projects unable to use external CSS files, the `InlineTheme` class converts stylesheets into inline styles. Terminal environments can utilize `LightTerminalTheme` with basic styling capabilities.

## Code Display Features

### Gutter Rendering

Line numbers can be displayed with optional starting positions:

```php
$highlighter = new Highlighter()->withGutter(startAt: 10);
```

Gutters indicate additions and deletions while rendering line numbers.

### Special Highlighting Tags

The package provides syntax for emphasis, strength, and obscuration within code snippets:
- `{_ content _}` applies emphasis
- `{* content *}` applies strong styling
- `{~ content ~}` applies blur effect

Additional tags mark additions and deletions, while custom classes enable developer-defined styling through `{:classname: content :}` syntax.

## Implementation Details

### Core Concepts: Patterns, Injections, Languages

**Patterns** match code segments using regex and assign token types for styling. Each pattern requires a named capture group: `(?<match>…)`

**Injections** highlight different languages nested within code blocks—for example, CSS within HTML `<style>` tags.

**Languages** combine patterns and injections into cohesive highlighting systems.

### Custom Language Example

Creating support for new languages involves extending existing language classes and implementing custom patterns and injections. The documentation illustrates building Blade template support by extending `HtmlLanguage` with Blade-specific patterns and injections.

### Dynamic Token Types

Fine-grained control over token styling uses `DynamicTokenType`, enabling custom CSS class application to specific tokens without modifying core language definitions.

## CommonMark Integration

The package integrates with `league/commonmark` for highlighting code blocks and inline code within Markdown:

```php
$environment = new Environment();
$environment
    ->addExtension(new CommonMarkCoreExtension())
    ->addExtension(new HighlightExtension());
```

Inline Markdown syntax supports language specification: `` `{php}code` ``

## Optional Features

**Markdown Support** requires installing `league/commonmark` separately.

**Word Complexity** analysis integrates via the `assertchris/ellison` package, identifying complex sentences and suggesting improvements with visual indicators.
