# LLM Prompts and AI IDE Setup
> Source: https://angular.dev/ai/develop-with-ai

## Overview

This guide helps developers generate accurate Angular code using Large Language Models (LLMs) through curated prompts, system instructions, and IDE configurations. The Angular team provides domain-specific resources to improve code generation quality.

## Custom Prompts and System Instructions

### Best Practices File

The Angular team offers a comprehensive best-practices markdown file for use as system instructions in AI tools. This resource covers several critical areas:

**TypeScript Guidelines:**
- Enforce strict type checking throughout projects
- Leverage type inference for obvious types
- Avoid `any` types; use `unknown` when uncertainty exists

**Angular Principles:**
- Prioritize standalone components over NgModules
- Don't explicitly set `standalone: true` (default in v20+)
- Implement signals for state management
- Use lazy loading for feature routes
- Place host bindings in component decorators instead of `@HostBinding`/`@HostListener`
- Apply `NgOptimizedImage` for static images

**Component Development:**
- Keep components focused on single responsibilities
- Use `input()` and `output()` functions instead of decorators
- Apply `computed()` for derived state
- Set `changeDetection: ChangeDetectionStrategy.OnPush`
- Use reactive forms over template-driven approaches
- Employ class and style bindings instead of `ngClass`/`ngStyle`

**State Management:**
- Use signals for local component state
- Apply `computed()` for derived values
- Maintain pure state transformations
- Use `update()` or `set()` instead of `mutate()`

**Template Standards:**
- Keep templates minimal and logic-light
- Use native control flow (`@if`, `@for`, `@switch`)
- Handle observables with async pipe
- Avoid arrow functions in templates

**Service Design:**
- Follow single-responsibility principle
- Use `providedIn: 'root'` for singletons
- Employ `inject()` for dependency injection

**Accessibility Requirements:**
- Pass all AXE checks
- Meet WCAG AA standards for contrast, focus management, and ARIA attributes

## IDE Configuration Files

The Angular team provides rules files for multiple development environments:

| Environment | File | Configuration |
|---|---|---|
| Firebase Studio | airules.md | Configure via Gemini settings |
| Copilot IDEs | copilot-instructions.md | Place in `.github/copilot-instructions.md` |
| Cursor | cursor.md | Configure via cursorrules.md |
| JetBrains IDEs | guidelines.md | Configure via IDE guidelines |
| VS Code | .instructions.md | Configure via copilot customization |
| Windsurf | guidelines.md | Configure via cascade memories |

## Angular CLI MCP Server

The Angular CLI includes an experimental Model Context Protocol (MCP) server enabling AI assistants to interact with CLI tooling. This facilitates more accurate code generation and scaffolding suggestions.

## Context via llms.txt

Two versions of `llms.txt` files are available:

- **llms.txt** - Index file with links to key resources
- **llms-full.txt** - Comprehensive resource describing Angular architecture and development practices

These files follow a proposed standard helping LLMs better comprehend and process website content.

## Web Codegen Scorer

The Angular team open-sourced the Web Codegen Scorer tool for evaluating AI-generated code quality. Developers can use it to refine prompts, compare model outputs, and track quality improvements over time.

## Key Takeaways

Effective AI-assisted Angular development requires:
- Well-structured system prompts covering TypeScript, Angular, and accessibility standards
- IDE-specific configuration files matching your development environment
- Regular updates to rules as Angular conventions evolve
- Quality evaluation tools for generated code assessment
