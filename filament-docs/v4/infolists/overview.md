# Filament Infolists Documentation

## Overview

Filament's infolists package displays read-only data for specific entities. It integrates with panel resources, relation managers, and action modals. Understanding the infolist builder streamlines custom Livewire application development.

## Defining Entries

Entry classes reside in the `Filament\Infolists\Components` namespace within the schema array. Built-in entries include:

- Text entry
- Icon entry
- Image entry
- Color entry
- Code entry
- Key-value entry
- Repeatable entry

Create entries using the static `make()` method with a unique name corresponding to model attributes. Use "dot notation" for relationship attributes:

```php
use Filament\Infolists\Components\TextEntry;

TextEntry::make('title')
TextEntry::make('author.name')
```

## Entry Content (State)

The displayed data is called "state." In panel resources, state derives from the record's attribute values. Access relationship values or JSON columns using dot notation:

```php
TextEntry::make('author.name')
TextEntry::make('meta.title')
```

### Setting Entry State

Pass custom state using the `state()` method:

```php
TextEntry::make('title')
    ->state('Hello, world!')
```

The method accepts functions with injectable utilities (Get, $record, $livewire, etc.).

### Default and Placeholder Values

```php
TextEntry::make('title')
    ->default('Untitled')
    ->placeholder('Untitled')
```

Defaults display as real state; placeholders appear as lighter gray text only when empty.

## Labels

Generate labels automatically from entry names or customize them:

```php
TextEntry::make('name')
    ->label('Full name')
```

Support translation strings for localization:

```php
TextEntry::make('name')
    ->label(__('entries.name'))
```

### Hiding Labels

Use `hiddenLabel()` for visual hiding while maintaining screen reader access:

```php
TextEntry::make('name')
    ->hiddenLabel()
```

## URL Navigation

Open URLs when entries are clicked:

```php
TextEntry::make('title')
    ->url('/about/titles')
    ->openUrlInNewTab()
```

Generate dynamic URLs with injected parameters:

```php
TextEntry::make('title')
    ->url(fn (Post $record): string =>
        PostResource::getUrl('edit', ['record' => $record])
    )
```

## Visibility Control

Hide entries conditionally:

```php
TextEntry::make('role')
    ->hidden()
    ->hidden(! FeatureFlag::active())
    ->visible(FeatureFlag::active())
```

Hide based on operations (create, edit, view):

```php
IconEntry::make('is_admin')
    ->hiddenOn('edit')
    ->visibleOn(['create', 'edit'])
```

JavaScript-based hiding requires `hiddenJs()` or `visibleJs()` without network requests:

```php
IconEntry::make('is_admin')
    ->visibleJs(<<<'JS'
        $get('role') === 'staff'
        JS)
```

## Inline Labels

Display labels alongside content rather than above:

```php
TextEntry::make('name')
    ->inlineLabel()
```

Apply globally to sections or entire schemas:

```php
Section::make('Details')
    ->inlineLabel()
    ->schema([...])
```

## Tooltips

Add hover tooltips:

```php
TextEntry::make('title')
    ->tooltip('Shown at the top of the page')
```

## Content Alignment

Align entry content using:

```php
TextEntry::make('title')
    ->alignStart()    // default
    ->alignCenter()
    ->alignEnd()
```

Or use the Alignment enum:

```php
TextEntry::make('title')
    ->alignment(Alignment::Center)
```

## Extra Content

Insert content around entries via slot methods:

- `aboveLabel()` / `belowLabel()`
- `beforeLabel()` / `afterLabel()`
- `aboveContent()` / `belowContent()`
- `beforeContent()` / `afterContent()`

Pass strings, schema components, actions, or arrays:

```php
TextEntry::make('name')
    ->belowContent('This is the user\'s full name.')
    ->belowContent(Text::make('...')->weight(FontWeight::Bold))
    ->belowContent(Action::make('generate'))
```

Align content using `Schema::start()`, `Schema::end()`, or `Schema::between()`.

## HTML Attributes

Add custom attributes to entries:

```php
TextEntry::make('slug')
    ->extraAttributes(['class' => 'bg-gray-200'])
    ->extraEntryWrapperAttributes(['class' => 'components-locked'])
```

Merge instead of overwrite using `merge: true`.

## Utility Injection

Functions accept injectable parameters:

- `$state` - current entry value
- `Get $get` - retrieve other field values
- `$record` - Eloquent model instance
- `string $operation` - current operation (create, edit, view)
- `Livewire $livewire` - Livewire component
- `Entry $component` - current entry instance

Combine parameters in any order:

```php
function (Livewire $livewire, Get $get, User $record) {
    // ...
}
```

Inject Laravel container dependencies alongside utilities.

## Global Configuration

Configure all entries globally via service provider:

```php
use Filament\Infolists\Components\TextEntry;

TextEntry::configureUsing(function (TextEntry $entry): void {
    $entry->words(10);
});
```

Override individual entries as needed.
