# Angular Language Service
> Source: https://angular.dev/tools/language-service

## Overview

The Angular Language Service enables code editors to provide intelligent assistance within Angular templates. It delivers completions, error detection, hints, and navigation features for both external HTML files and inline templates.

## Configuration

To activate advanced Language Service capabilities, modify your `tsconfig.json`:

```json
"angularCompilerOptions": {
  "strictTemplates": true
}
```

This setting enables stricter type checking in templates. For additional details, consult the Angular compiler options guide.

## Core Features

The service automatically detects Angular files and reads your project configuration to identify templates, then offers:

- **Completion suggestions** - Context-aware recommendations as you type
- **Diagnostic messages** - Ahead-of-time compilation warnings
- **Hover information** - Details about components, directives, and modules
- **Definition navigation** - Jump to source declarations

### Autocompletion

The service suggests completions during template interpolations and element declarations. Component selectors automatically appear in suggestion lists.

### Error Detection

The Language Service warns about undefined variables and incorrect references before runtime, helping catch mistakes during development.

### Navigation Features

Hovering over identifiers reveals their origin and type information. Pressing F12 or selecting "Go to definition" takes you directly to the source.

## Editor Support

The Language Service is available as extensions for:

- **Visual Studio Code** - Install from the Extensions Marketplace (maintained by Angular team)
- **Visual Studio** - Available through Extensions Marketplace (maintained by Microsoft)
- **WebStorm** - Enable the built-in Angular and AngularJS plugin
- **Sublime Text** - Requires TypeScript plugin plus Angular Language Service package
- **Eclipse IDE** - Available through Eclipse Marketplace via Wild Web Developer extension
- **Neovim** - Supported via nvim-lspconfig plugin or Conquer of Completion
- **Zed** - Available from the Extensions Marketplace

## How It Works

The language service operates through an RPC connection using the Language Server Protocol. When you open a template, the editor transmits project state information to the language service process.

Upon triggering completions, the service parses the template into an HTML abstract syntax tree. The Angular compiler analyzes this structure to understand context -- including module membership, scope, component selectors, and cursor position -- to determine possible symbols at that location.

For interpolations like `{{data.---}}`, the parser generates an expression AST within the template AST. The service queries the TypeScript Language Service about `data`'s members and returns appropriate suggestions.

## Additional Resources

- Source code: Angular Language Service repository on GitHub
- Design documentation: Available in the vscode-ng-language-service wiki
