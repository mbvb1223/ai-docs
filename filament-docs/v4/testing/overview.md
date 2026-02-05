# Filament Testing Documentation (v4.x)

## Overview

### Introduction

The Filament testing guide uses **Pest** as the primary testing framework, with examples adaptable to PHPUnit by substituting Pest's `livewire()` function with `Livewire::test()`.

> "All Filament components are mounted to a Livewire component, we're just using Livewire testing helpers everywhere."

For foundational knowledge, the Livewire documentation provides comprehensive [testing guidance](https://livewire.laravel.com/docs/testing).

### Available Testing Resources

The documentation provides specialized guides for:

- **Testing resources** — Full panel resource testing examples
- **Testing tables** — Methods and techniques for table component testing
- **Testing schemas** — Covers both form and infolist schemas
- **Testing actions** — Table and schema-integrated action testing
- **Testing notifications** — Validation of sent notifications
- **Custom pages** — Livewire components requiring standard Livewire testing approaches

### Livewire Component Identification

Understanding which Filament classes function as Livewire components is essential for effective testing.

**Livewire components include:**
- Panel pages (including resource page classes)
- Relation managers
- Widgets

**Non-Livewire components include:**
- Resource classes
- Schema components
- Actions

> "You can still test them, for example, by calling various methods and using the Pest expectation API to assert the expected behavior."

While non-Livewire classes remain testable through direct method invocation, comprehensive end-to-end coverage requires testing Livewire components directly.
