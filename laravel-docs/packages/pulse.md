# Laravel 12.x Pulse Documentation

## Introduction

Laravel Pulse delivers at-a-glance insights into your application's performance and usage - tracking bottlenecks, active users, and application health.

## Installation

```bash
composer require laravel/pulse
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
php artisan migrate
```

Access dashboard at `/pulse`.

## Dashboard Authorization

```php
use Illuminate\Support\Facades\Gate;

Gate::define('viewPulse', function (User $user) {
    return $user->isAdmin();
});
```

## Built-in Cards

| Card | Purpose |
|------|---------|
| `<livewire:pulse.servers />` | CPU, memory, storage usage |
| `<livewire:pulse.usage />` | Top users by requests/jobs |
| `<livewire:pulse.exceptions />` | Exception frequency |
| `<livewire:pulse.queues />` | Queue throughput |
| `<livewire:pulse.slow-requests />` | Slow requests (>1000ms) |
| `<livewire:pulse.slow-jobs />` | Slow jobs (>1000ms) |
| `<livewire:pulse.slow-queries />` | Slow database queries |
| `<livewire:pulse.cache />` | Cache hit/miss statistics |

## Recorders Configuration

```php
Recorders\SlowJobs::class => [
    'threshold' => [
        '#^App\\Jobs\\GenerateReports$#' => 5000,
        'default' => 1000,
    ],
],

Recorders\SlowQueries::class => [
    'threshold' => 1000,
],
```

## Data Capture

### Running Pulse Check

```bash
php artisan pulse:check
```

### Graceful Restart

```bash
php artisan pulse:restart
```

## Performance Optimization

### Separate Database

```env
PULSE_DB_CONNECTION=pulse
```

### Redis Ingest

```env
PULSE_INGEST_DRIVER=redis
```

```bash
php artisan pulse:work
```

### Sampling

```php
Recorders\UserRequests::class => [
    'sample_rate' => 0.1,  // 10% of requests
],
```

## Custom Cards

```php
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

### Recording Data

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

### Retrieving Data

```php
$topSellers = $this->aggregate('user_sale', ['sum', 'count']);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/pulse)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
