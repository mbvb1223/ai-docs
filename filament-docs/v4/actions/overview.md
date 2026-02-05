# Filament Actions Documentation

## Overview

In Filament, actions represent user interface workflows that perform operations in your application. Unlike traditional action classes, Filament actions handle interactive UI experiences—such as confirming deletions, collecting user input via modals, or redirecting to pages.

## Core Concepts

### Basic Action Structure

Actions can execute server-side logic:

```php
use Filament\Actions\Action;

Action::make('delete')
    ->requiresConfirmation()
    ->action(fn () => $this->client->delete())
```

Or collect additional information through modals:

```php
Action::make('sendEmail')
    ->schema([
        TextInput::make('subject')->required(),
        RichEditor::make('body')->required(),
    ])
    ->action(function (array $data) {
        Mail::to($this->client)->send(
            new GenericEmail(
                subject: $data['subject'],
                body: $data['body'],
            )
        );
    })
```

Simple redirect actions require no modal:

```php
Action::make('edit')
    ->url(fn (): string => route('posts.edit', ['post' => $this->post]))
```

## Trigger Styles

Filament provides four button styles:

- **Button**: Standard button with background, label, and optional icon
- **Link**: No background; resembles inline text links
- **Icon Button**: Circular buttons with icons only
- **Badge**: Colored backgrounds suitable for status indicators

```php
Action::make('edit')->button()        // Default style
Action::make('edit')->link()          // Link style
Action::make('edit')->iconButton()    // Icon-only
Action::make('edit')->badge()         // Badge style
```

### Responsive Label Hiding

Hide labels on mobile devices while keeping them on desktop:

```php
Action::make('edit')
    ->icon('heroicon-m-pencil-square')
    ->button()
    ->labeledFrom('md')
```

## Customization Methods

### Labels and Colors

```php
Action::make('delete')
    ->label('Remove permanently')
    ->color('danger')
```

### Sizing

Actions support three sizes: `Small`, `Medium`, `Large`:

```php
Action::make('create')->size(Size::Large)
```

### Icons

```php
Action::make('edit')
    ->icon('heroicon-m-pencil-square')
    ->iconPosition(IconPosition::After)
```

### Badges

Add corner badges for counts or status:

```php
Action::make('filter')
    ->iconButton()
    ->icon('heroicon-m-funnel')
    ->badge(5)
    ->badgeColor('success')
```

### Outlined Buttons

```php
Action::make('edit')
    ->button()
    ->outlined()
```

## Authorization

Control visibility and access through policies or explicit conditions:

```php
Action::make('edit')
    ->visible(auth()->user()->can('update', $this->post))
    ->authorize('update')
```

Optionally disable buttons with tooltips instead of hiding:

```php
Action::make('edit')
    ->authorize('update')
    ->authorizationTooltip()
```

## Rate Limiting

Prevent abuse by limiting action attempts:

```php
Action::make('delete')->rateLimit(5)  // 5 attempts per minute
```

Customize the rate-limit notification:

```php
Action::make('delete')
    ->rateLimit(5)
    ->rateLimitedNotificationTitle('Slow down!')
```

## Keyboard Shortcuts

Attach keybindings using Mousetrap syntax:

```php
Action::make('save')
    ->action(fn () => $this->save())
    ->keyBindings(['command+s', 'ctrl+s'])
```

## Schema Integration

Actions work within form schemas, accessing state through utility injection:

```php
TextInput::make('title')
    ->afterContent(
        Action::make('generateSlug')
            ->action(function (Get $schemaGet, Set $schemaSet) {
                $schemaSet('slug',
                    str($schemaGet('title'))->slug()
                );
            })
    )

TextInput::make('slug')
```

### Client-Side JavaScript Actions

For instant interactions without server requests:

```php
TextInput::make('title')
    ->afterContent(
        Action::make('generateSlug')
            ->actionJs(<<<'JS'
                $set('slug',
                    $get('title').toLowerCase()
                        .replaceAll(' ', '-'))
            JS)
    )
```

## Built-in Actions

Filament includes pre-configured actions:

- Create, Edit, View, Delete
- Replicate, Force-delete, Restore
- Import, Export

## Utility Injection

Actions accept closures with injected parameters:

- `$action` — Current action instance
- `$arguments` — Arguments passed when triggered
- `$data` — Submitted form field data
- `$livewire` — Livewire component
- `$record` — Associated Eloquent model
- `$selectedRecords` — Bulk action selections
- `$table` — Table instance
- `$schemaGet` / `$schemaSet` — Schema state accessors
- `$schemaOperation` — Current operation (create/edit/view)

## HTML Attributes

Add custom attributes to action elements:

```php
Action::make('edit')
    ->extraAttributes(['title' => 'Edit this post'])
```

Use `merge: true` to combine with existing attributes rather than replacing them.
