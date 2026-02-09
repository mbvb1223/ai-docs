# Custom Build Pipeline
> Source: https://angular.dev/ecosystem/custom-build-pipeline

## Overview

When building an Angular app it is strongly recommended to use the Angular CLI to leverage its structure-dependent update functionality and build system abstraction.

This guide addresses rare scenarios where developers need custom build pipelines outside the Angular CLI ecosystem. Community-maintained tools provide alternatives for specific use cases.

## When to Use a Custom Build Pipeline

Consider a custom pipeline for these niche situations:

- Adding Angular to existing applications using different toolchains
- Projects tightly coupled to module federation unable to adopt bundler-agnostic native federation
- Short-lived experiments with preferred build tools

## Available Options

Two well-supported community tools enable custom build pipelines using underlying Angular CLI abstractions. Both require manual maintenance without automated updates.

### Rspack

Rspack is a Rust-based bundler providing webpack plugin ecosystem compatibility.

**Use case:** Projects heavily relying on custom webpack configurations seeking improved build times.

**Resources:** Full details available on the [Rspack documentation website](https://nx.dev/recipes/angular/rspack/introduction)

### Vite

Vite delivers faster, leaner development experiences for modern web projects through an extensible plugin system. The Angular CLI itself uses Vite as its development server.

**Features:**
- Ecosystem integrations: Vitest (testing), Storybook (component authoring)
- AnalogJS plugin enables Angular adoption in Vite-based projects
- Support for adding Angular to existing pipelines

**Examples:** Integrating Angular UI components into documentation sites using Astro and Starlight

**Resources:** Learn more via the [AnalogJS documentation](https://analogjs.org/docs/packages/vite-plugin-angular/overview)

Both tools use abstractions powering Angular CLI while allowing flexible, custom build configurations.
