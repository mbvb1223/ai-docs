# Getting Started with Filament

## Overview

Filament is a Laravel admin panel framework that provides a complete CRUD interface system. After installation, users can access the admin panel at `/admin` to begin managing their application data.

## Core Components

### Resources

Resources form the foundation of Filament applications. They provide automatically-generated interfaces for managing Eloquent models:

- **List Page**: "A paginated table of all the records in the Eloquent model"
- **Create Page**: Form interface for adding new records
- **Edit Page**: Form interface for modifying existing records
- **View Page** (optional): Read-only record display

Each resource automatically generates a sidebar navigation item upon creation.

### Widgets

Widgets serve as dashboard components for displaying data visualization and statistics. Key characteristics include:

- Built on PHP classes with corresponding Blade views
- Powered by Livewire, enabling interactive server-rendered interfaces
- Support for charts, statistics, tables, and custom implementations
- Pre-installed dashboard widgets for user greeting and Filament information

### Custom Pages

Custom pages provide flexible blank-canvas interfaces for specialized functionality such as settings or documentation. They leverage Livewire's full-page component architecture, offering complete UI customization capabilities.

## Next Steps

The documentation provides dedicated sections for Resources, Widgets, Navigation, Forms, Tables, and various other features to help developers build comprehensive admin interfaces.
