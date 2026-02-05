# Filament Text Column Documentation

## Overview

Text columns display simple text in Filament tables using `TextColumn::make('fieldname')`.

## Key Features

### Styling & Appearance

**Colors**: Apply colors dynamically or statically using `->color('primary')`. The method accepts functions for conditional coloring based on record state.

**Icons**: Add icons via `->icon(Heroicon::Envelope)` with position control using `->iconPosition(IconPosition::Before|After)`. Icon colors can be set separately.

**Badges**: Display text as badges using `->badge()`, useful for status displays with matching colors.

### Text Formatting

**Date/Time**: Use `->date()`, `->dateTime()`, `->time()` with custom PHP format strings. Also supports `->isoDate()`, `->isoDateTime()`, `->since()` for relative dates.

**Numbers**: Format using `->numeric(decimalPlaces: 2, locale: 'en')`.

**Money**: Apply `->money('EUR', divideBy: 100, decimalPlaces: 2)` for currency formatting.

**Markdown & HTML**: Render as `->markdown()` or `->html()` (sanitized by default).

### Content Management

**Descriptions**: Add context with `->description(fn() => $text, position: 'below'|'above')`.

**Multiple Values**: Handle arrays with `->listWithLineBreaks()`, `->bulleted()`, `->limitList(3)`, `->expandableLimitedList()`.

**Separation**: Split strings using `->separator(',')` for tag-like displays.

### Typography

- **Size**: Control with `size(TextSize::ExtraSmall|Medium|Large)`
- **Weight**: Adjust via `weight(FontWeight::Bold|SemiBold)` etc.
- **Family**: Set with `fontFamily(FontFamily::Sans|Serif|Mono)`

### Long Text Handling

- `->limit(50, end: '...')` - Character limit
- `->words(10)` - Word count limit
- `->wrap()` - Enable line wrapping
- `->lineClamp(2)` - Limit to specific lines
- `->copyable()` - Enable clipboard copying with custom messages

All methods supporting dynamic values can inject utilities like `$record`, `$state`, `$livewire`, and `$table` as parameters.
