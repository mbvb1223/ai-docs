# Angular CLI Overview
> Source: https://angular.dev/tools/cli

## Overview

The Angular CLI is a command-line interface tool that enables developers to scaffold, develop, test, deploy, and maintain Angular applications directly from a command shell.

**Key Information:**
- Published on npm as the `@angular/cli` package
- Includes a binary named `ng`
- Commands invoking `ng` utilize the Angular CLI

## Getting Started Without Local Setup

New users can begin with "Try it now!" tutorial, which introduces Angular essentials through a ready-made basic online store app. This standalone tutorial leverages the interactive StackBlitz environment for online development, eliminating the need for local environment setup initially.

## Main Features and Resources

### Four Core Areas

1. **Getting Started** - Install Angular CLI to create and build your first application

2. **Command Reference** - Discover CLI commands for increased productivity with Angular

3. **Schematics** - Create and run schematics to automatically generate and modify source files in your application

4. **Builders** - Create and run builders to perform complex transformations from source code to generated build outputs

## CLI Command-Language Syntax

Angular CLI follows Unix/POSIX conventions for option syntax.

### Boolean Options

Boolean options have two forms: `--this-option` sets the flag to `true`, `--no-this-option` sets it to `false`. You can also use `--this-option=false` or `--this-option=true`. The flag remains in its default state if neither option is supplied.

### Array Options

Array options can be provided in two forms: `--option value1 value2` or `--option value1 --option value2`.

### Key/Value Options

Some options (like `--define`) expect an array of `key=value` pairs. These can be provided as: `--define 'KEY_1="value1"' KEY_2=true` or `--define 'KEY_1="value1"' --define KEY_2=true`.

### Relative Paths

Options specifying files accept absolute paths or paths relative to the current working directory (typically the workspace or project root).
