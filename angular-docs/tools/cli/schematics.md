# Generating Code Using Schematics
> Source: https://angular.dev/tools/cli/schematics

## Overview

A schematic represents a template-based code generator supporting complex logic. It is a set of instructions for transforming a software project by generating or modifying code. Schematics are packaged into collections and installed with npm.

Schematics serve as powerful tools for creating, modifying, and maintaining software projects, particularly useful for customizing Angular projects to organizational needs. Common use cases include generating UI patterns, enforcing architectural rules, and maintaining consistency across projects.

## Schematics in the Angular CLI

The Angular CLI integrates schematics to apply transformations to web applications. The `@schematics/angular` collection provides default schematics executed by `ng generate` and `ng add` commands.

### Basic Command Syntax

To invoke a specific schematic or collection, use either format:

```bash
ng generate my-schematic-collection:my-schematic-name
```

Or alternatively:

```bash
ng generate my-schematic-name --collection collection-name
```

### Configuring CLI Schematics

JSON schemas associated with schematics inform the Angular CLI about available options and defaults. These defaults can be overridden by providing a different value for an option on the command line.

The `@schematics/angular` package contains JSON schemas for default schematics, describing options available for each `ng generate` sub-command.

## Three Categories of Schematics

### Add Schematics

Add schematics enable installation of libraries into Angular workspaces via `ng add`. The add command uses your package manager to download new dependencies, and invokes an installation script that is implemented as a schematic.

Examples include:
- `@angular/material` - Installs Material design components and theming
- `@ng-bootstrap/schematics` - Adds ng-bootstrap functionality
- `@clr/angular` - Installs Clarity design system
- `@angular/pwa` - Converts applications to Progressive Web Apps

### Generation Schematics

Generation schematics provide instructions for the `ng generate` command. They enable creation of artifacts defined within libraries.

Example usage:

```bash
ng generate @angular/material:table <component-name>
```

This command creates an Angular Material table pre-configured with datasource functionality for sorting and pagination.

### Update Schematics

The `ng update` command analyzes workspace dependencies and suggests available updates:

```bash
ng update
```

Output includes package versions and update commands. When specified with library names:

```bash
ng update @angular/material
```

The command updates both the specified package and its dependencies. If update schematics exist, they automatically resolve breaking changes during migration.

## Important Considerations

We recommend that you do not force an update of all dependencies by default. Try updating specific dependencies first.

If peer dependency conflicts arise (preventing semver range matching), the command generates an error without modifying the workspace, protecting against inconsistent states.

## Learning Resources

For comprehensive details on creating custom schematics, refer to:
- Authoring Schematics guide
- Schematics for Libraries guide

These resources provide detailed instructions for library developers implementing custom add, generation, and update schematics.
