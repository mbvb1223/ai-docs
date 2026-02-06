# Laravel 12.x Cashier (Stripe) Documentation

## Introduction

Laravel Cashier provides an expressive, fluent interface to Stripe's subscription billing services. It handles subscriptions, coupons, subscription swaps, quantities, cancellation grace periods, and invoice PDF generation.

## Installation

```bash
composer require laravel/cashier
php artisan vendor:publish --tag="cashier-migrations"
php artisan migrate
```

## Configuration

### Billable Model

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

### Environment Variables

```env
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
CASHIER_CURRENCY=eur
```

## Quickstart

### Selling Products

```php
Route::get('/checkout', function (Request $request) {
    return $request->user()->checkout(['price_deluxe_album' => 1], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
});
```

### Selling Subscriptions

```php
Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_basic_monthly')
        ->trialDays(5)
        ->allowPromotionCodes()
        ->checkout([
            'success_url' => route('success'),
            'cancel_url' => route('cancel'),
        ]);
});
```

## Customers

```php
$stripeCustomer = $user->createAsStripeCustomer();
$stripeCustomer = $user->createOrGetStripeCustomer();

// Balance
$balance = $user->balance();
$user->creditBalance(500, 'Top-up');
$user->debitBalance(300, 'Penalty');

// Tax IDs
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

### Billing Portal

```php
Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
});
```

## Payment Methods

### Storing Payment Methods

```php
return view('update-payment-method', [
    'intent' => $user->createSetupIntent()
]);
```

### Retrieving Payment Methods

```php
$paymentMethods = $user->paymentMethods();
$paymentMethod = $user->defaultPaymentMethod();
$user->updateDefaultPaymentMethod($paymentMethod);
```

## Subscriptions

### Creating Subscriptions

```php
$request->user()->newSubscription('default', 'price_monthly')
    ->create($request->paymentMethodId);

// With coupon
$user->newSubscription('default', 'price_monthly')
    ->withCoupon('code')
    ->create($paymentMethod);
```

### Checking Status

```php
if ($user->subscribed('default')) { }
if ($user->subscription('default')->onTrial()) { }
if ($user->subscription('default')->canceled()) { }
if ($user->subscription('default')->onGracePeriod()) { }
```

### Changing Prices

```php
$user->subscription('default')->swap('price_yearly');
$user->subscription('default')->swapAndInvoice('price_yearly');
```

### Quantities

```php
$user->subscription('default')->incrementQuantity();
$user->subscription('default')->updateQuantity(10);
```

### Multiple Products

```php
$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);

$user->subscription('default')->addPrice('price_chat');
$user->subscription('default')->removePrice('price_chat');
```

### Canceling

```php
$user->subscription('default')->cancel();
$user->subscription('default')->cancelAt(now()->addMonths(1));
```

### Resuming

```php
$user->subscription('default')->resume();
```

## Subscription Trials

```php
$user->newSubscription('default', 'price_monthly')
    ->trialDays(10)
    ->create($paymentMethod);

// Without payment method
$user->newSubscription('default', 'price_monthly')
    ->trialDays(10)
    ->createAndSendInvoice();
```

## Single Charges

```php
$payment = $user->charge(100, $paymentMethod);
$payment = $user->invoiceFor('Product', 100);

// Refunds
$refund = $user->refund($paymentIntentId);
```

## Invoices

```php
$invoices = $user->invoices();
$invoice = $user->findInvoice($invoiceId);
$upcomingInvoice = $user->upcomingInvoice();

// PDF
return $invoice->download(['filename' => 'invoice.pdf']);
```

## Checkout

```php
// Product checkout
$checkout = $user->checkout(['price_id' => 1], [
    'success_url' => route('success'),
    'cancel_url' => route('cancel'),
]);

// Guest checkout
$checkout = Cashier::guest()->checkout(['price_id' => 1], [
    'success_url' => route('success'),
    'cancel_url' => route('cancel'),
    'customer_email' => 'guest@example.com',
]);
```

## Webhooks

```php
use Laravel\Cashier\Events\WebhookReceived;

Event::listen(WebhookReceived::class, function (WebhookReceived $event) {
    if ($event->payload['type'] === 'charge.succeeded') {
        // Handle charge succeeded
    }
});
```

## Stripe SDK Access

```php
use Laravel\Cashier\Cashier;

$stripe = Cashier::stripe();
$customer = $stripe->customers->retrieve($customerId);
```

---

**Source**: [Laravel Documentation](https://laravel.com/docs/12.x/billing)

**License**: This work is licensed under the [MIT License](https://opensource.org/licenses/MIT).
