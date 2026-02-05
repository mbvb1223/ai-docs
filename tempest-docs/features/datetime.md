# Date and Time

## Overview

Tempest offers an improved alternative to PHP's native `DateTime` and `DateTimeImmutable` classes. The framework provides a better API with a more consistent interface, originally created for the PSL library and adapted for deeper framework integration.

Users retain flexibility—the implementation is optional. Those using third-party libraries like Carbon should explore global casters and serializers for model support.

## Creating Date Instances

### Current Date and Time

The recommended approach uses dependency injection of the `Clock` interface:

```php
final readonly class HomeController
{
    public function __construct(
        private readonly Clock $clock,
    ) {}

    public function __invoke(): View
    {
        return view('./home.view.php', currentTime: $this->clock->now());
    }
}
```

Alternatively, convenience methods are available:

```php
$now = DateTime::now();
// or
$now = Tempest\now();
```

### From String or Timestamp

```php
DateTime::parse('2025-09-19 02:00:00');
```

### From Known Format Pattern

Using standard ICU format syntax:

```php
DateTime::fromPattern('2025-09-19 02:00', pattern: 'yyyy-MM-dd HH:mm');
```

## Manipulating Dates

Add or subtract time using `Duration` instances:

```php
$date->plus(Duration::seconds(30));
$date->plusHour();
$date->plusMinutes(30);
$date->minusDay();
$date->endOfDay();
```

## Converting Timezones

```php
use Tempest\DateTime\DateTime;
use Tempest\DateTime\Timezone;

$date = DateTime::now();
$date->convertToTimezone(Timezone::EUROPE_PARIS);
```

All creation methods accept an optional `timezone` parameter using the `Timezone` enumeration.

## Computing Duration

```php
$date1 = DateTime::now();
$date2 = DateTime::parse('2025-09-19 02:00:00');
$duration = $date1->between($date2);
```

## Comparing Dates

```php
$date->isBefore($other);
$date->isBeforeOrAt($other);
$date->betweenTimeInclusive($otherDate1, $otherDate2);
$date->isFuture();
```

## Formatting Dates

```php
use Tempest\DateTime\FormatPattern;
use Tempest\Intl\Locale;

$date->format();
$date->format(pattern: FormatPattern::COOKIE);
$date->format(locale: Locale::FRENCH);
```

## Clock Interface

The `Clock` interface supports PSR-20 conversion:

```php
$psrClock = $clock->toPsrClock();
```

## Testing Time

Within test cases, replace the framework's `Clock` instance with a testing version:

```php
$clock = $this->clock();
$clock->setNow('2025-09-19 02:00:00');
$clock->sleep(milliseconds: 250);
```
