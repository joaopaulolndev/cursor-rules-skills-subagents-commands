---
description: laravel-cashier
---

# Laravel Cashier (Stripe)

> Expressive, fluent interface to Stripe's subscription billing services. Handles subscriptions, single charges, invoices, Stripe Checkout, webhooks, SCA, trials, and more.

## Installation

```bash
composer require laravel/cashier
php artisan vendor:publish --tag="cashier-migrations"
php artisan migrate
# Optional: publish config
php artisan vendor:publish --tag="cashier-config"
```

## Configuration

### Environment Variables

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
CASHIER_CURRENCY=usd
CASHIER_CURRENCY_LOCALE=en_US
CASHIER_LOGGER=stack
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

### Billable Model

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Custom billable model (in `AppServiceProvider::boot()`):

```php
use Laravel\Cashier\Cashier;
Cashier::useCustomerModel(\App\Models\Cashier\User::class);
```

### Custom Models

```php
use Laravel\Cashier\Cashier;
Cashier::useSubscriptionModel(\App\Models\Cashier\Subscription::class);
Cashier::useSubscriptionItemModel(\App\Models\Cashier\SubscriptionItem::class);
```

### Tax Configuration (Stripe Tax)

```php
// AppServiceProvider::boot()
Cashier::calculateTaxes();
```

### Subscription State Configuration

```php
// AppServiceProvider::register()
Cashier::keepPastDueSubscriptionsActive();
Cashier::keepIncompleteSubscriptionsActive();
```

## Customers

### Creating & Retrieving

```php
// Find by Stripe ID
$user = Cashier::findBillable($stripeId);

// Create in Stripe
$stripeCustomer = $user->createAsStripeCustomer($options);

// Get or create
$stripeCustomer = $user->createOrGetStripeCustomer();

// Get existing
$stripeCustomer = $user->asStripeCustomer();

// Update
$stripeCustomer = $user->updateStripeCustomer($options);
```

### Balances

```php
$balance = $user->balance();
$user->creditBalance(500, 'Top-up.');
$user->debitBalance(300, 'Penalty.');
$transactions = $user->balanceTransactions();
```

### Tax IDs

```php
$taxIds = $user->taxIds();
$taxId = $user->findTaxId('txi_belgium');
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
$user->deleteTaxId('txi_belgium');
```

### Syncing Customer Data

```php
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}

// Override sync methods:
public function stripeName(): string|null { return $this->company_name; }
// Also: stripeEmail(), stripePhone(), stripeAddress(), stripePreferredLocales()
```

### Billing Portal

```php
return $request->user()->redirectToBillingPortal(route('billing'));
$url = $request->user()->billingPortalUrl(route('billing'));
```

## Payment Methods

### Setup Intent (for subscriptions)

```php
// Controller
$intent = $user->createSetupIntent();

// Blade
<button data-secret="{{ $intent->client_secret }}">Update</button>

// JS (Stripe.js)
const { setupIntent, error } = await stripe.confirmCardSetup(clientSecret, {
    payment_method: { card: cardElement, billing_details: { name: 'John' } }
});
```

### Managing Payment Methods

```php
$paymentMethods = $user->paymentMethods();          // all types
$paymentMethods = $user->paymentMethods('sepa_debit'); // specific type
$paymentMethod = $user->defaultPaymentMethod();
$paymentMethod = $user->findPaymentMethod($id);

// Presence checks
$user->hasDefaultPaymentMethod();
$user->hasPaymentMethod();
$user->hasPaymentMethod('sepa_debit');

// Update/Add/Delete
$user->updateDefaultPaymentMethod($paymentMethod);
$user->updateDefaultPaymentMethodFromStripe();
$user->addPaymentMethod($paymentMethod);
$paymentMethod->delete();
$user->deletePaymentMethod('pm_visa');
$user->deletePaymentMethods();               // all
$user->deletePaymentMethods('sepa_debit');   // specific type
```

## Subscriptions

### Creating

```php
// With payment method
$request->user()->newSubscription('default', 'price_monthly')
    ->create($request->paymentMethodId);

// With quantity
$user->newSubscription('default', 'price_monthly')
    ->quantity(5)
    ->create($paymentMethod);

// With coupon/promotion
$user->newSubscription('default', 'price_monthly')
    ->withCoupon('code')
    ->create($paymentMethod);

$user->newSubscription('default', 'price_monthly')
    ->withPromotionCode('promo_code_id')
    ->create($paymentMethod);

// Via invoice (no payment method upfront)
$user->newSubscription('default', 'price_monthly')
    ->createAndSendInvoice([], ['days_until_due' => 30]);

// Additional customer/subscription options
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => 'Extra info.'],
]);

// Add to existing customer with default payment method
$user->newSubscription('default', 'price_monthly')->add();
```

### Checking Status

```php
$user->subscribed('default');                    // active (incl. trial)
$user->subscribedToProduct('prod_premium', 'default');
$user->subscribedToPrice('price_basic_monthly', 'default');
$user->subscription('default')->onTrial();
$user->subscription('default')->recurring();     // active, not on trial
$user->subscription('default')->canceled();
$user->subscription('default')->onGracePeriod();
$user->subscription('default')->ended();
$user->hasIncompletePayment('default');
$user->subscription('default')->hasIncompletePayment();
```

### Subscription Scopes

```php
Subscription::query()->active();
Subscription::query()->canceled();
Subscription::query()->ended();
Subscription::query()->incomplete();
Subscription::query()->notCanceled();
Subscription::query()->notOnGracePeriod();
Subscription::query()->notOnTrial();
Subscription::query()->onGracePeriod();
Subscription::query()->onTrial();
Subscription::query()->pastDue();
Subscription::query()->recurring();
```

### Changing Prices

```php
$user->subscription('default')->swap('price_yearly');
$user->subscription('default')->skipTrial()->swap('price_yearly');
$user->subscription('default')->swapAndInvoice('price_yearly');
$user->subscription('default')->noProrate()->swap('price_yearly');
```

### Subscription Quantity

```php
$user->subscription('default')->incrementQuantity();
$user->subscription('default')->incrementQuantity(5);
$user->subscription('default')->decrementQuantity();
$user->subscription('default')->decrementQuantity(5);
$user->subscription('default')->updateQuantity(10);
$user->subscription('default')->noProrate()->updateQuantity(10);
```

### Multiple Products per Subscription

```php
// Create with multiple prices
$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);

// Add/remove prices
$user->subscription('default')->addPrice('price_chat');
$user->subscription('default')->addPriceAndInvoice('price_chat');
$user->subscription('default')->addPrice('price_chat', 5); // with quantity
$user->subscription('default')->removePrice('price_chat');

// Swap prices
$user->subscription('default')->swap(['price_pro', 'price_chat']);
$user->subscription('default')->swap([
    'price_pro' => ['quantity' => 5],
    'price_chat'
]);

// Swap single item
$user->subscription('default')->findItemOrFail('price_basic')->swap('price_pro');

// Access subscription items
$subscriptionItem = $user->subscription('default')->items->first();
$subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');
```

### Multiple Subscriptions

```php
$user->newSubscription('swimming')->price('price_swimming_monthly')
    ->create($paymentMethodId);
$user->subscription('swimming')->swap('price_swimming_yearly');
$user->subscription('swimming')->cancel();
```

### Usage Based Billing (Metered)

```php
// Create metered subscription
$user->newSubscription('default')
    ->meteredPrice('price_metered')
    ->create($paymentMethodId);

// Report usage
$user->reportMeterEvent('emails-sent');
$user->reportMeterEvent('emails-sent', quantity: 15);

// Get usage summaries
$meterUsage = $user->meterEventSummaries($meterId);
$meterUsage->first()->aggregated_value;

// List all meters
$user->meters();
```

### Subscription Taxes

```php
// On billable model
public function taxRates(): array
{
    return ['txr_id'];
}

// Per-price tax rates
public function priceTaxRates(): array
{
    return ['price_monthly' => ['txr_id']];
}

// Sync tax rates
$user->subscription('default')->syncTaxRates();

// Tax exemption checks
$user->isTaxExempt();
$user->isNotTaxExempt();
$user->reverseChargeApplies();
```

### Anchor Date

```php
$user->newSubscription('default', 'price_monthly')
    ->anchorBillingCycleOn(Carbon::parse('first day of next month')->startOfDay())
    ->create($paymentMethodId);
```

### Cancelling

```php
$user->subscription('default')->cancel();          // end of period (grace)
$user->subscription('default')->cancelNow();       // immediate
$user->subscription('default')->cancelNowAndInvoice(); // immediate + final invoice
$user->subscription('default')->cancelAt(now()->addDays(10));

// Always cancel before deleting user
$user->subscription('default')->cancelNow();
$user->delete();
```

### Resuming

```php
$user->subscription('default')->resume(); // must be within grace period
```

## Subscription Trials

### With Payment Method

```php
$user->newSubscription('default', 'price_monthly')
    ->trialDays(10)
    ->create($paymentMethodId);

$user->newSubscription('default', 'price_monthly')
    ->trialUntil(Carbon::now()->addDays(10))
    ->create($paymentMethod);

// Check trial status
$user->onTrial('default');
$user->subscription('default')->onTrial();
$user->subscription('default')->endTrial();
$user->hasExpiredTrial('default');
$user->subscription('default')->hasExpiredTrial();
```

### Without Payment Method (Generic Trial)

```php
$user = User::create([
    // ...
    'trial_ends_at' => now()->addDays(10),
]);

$user->onTrial();          // true during trial
$user->onGenericTrial();   // true only for generic trials
$trialEndsAt = $user->trialEndsAt('main');
```

### Extending Trials

```php
$subscription->extendTrial(now()->addDays(7));
$subscription->extendTrial($subscription->trial_ends_at->addDays(5));
```

## Webhooks

### Setup

```bash
php artisan cashier:webhook
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
php artisan cashier:webhook --disabled
```

Default route: `/stripe/webhook`

Required webhook events:
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

### CSRF Exclusion

```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

### Custom Event Handlers

```php
use Laravel\Cashier\Events\WebhookReceived;
use Laravel\Cashier\Events\WebhookHandled;

class StripeEventListener
{
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'invoice.payment_succeeded') {
            // Handle event...
        }
    }
}
```

## Single Charges

### Simple Charge

```php
// Amount in lowest currency denominator (cents for USD)
$stripeCharge = $user->charge(100, $paymentMethodId);
$stripeCharge = $user->charge(100, $paymentMethod, ['custom_option' => $value]);

// Without existing customer
$stripeCharge = (new User)->charge(100, $paymentMethod);

// Returns Laravel\Cashier\Payment, throws on failure
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) { /* ... */ }
```

### Charge With Invoice

```php
$user->invoicePrice('price_tshirt', 5);
$user->invoicePrice('price_tshirt', 5, [
    'discounts' => [['coupon' => 'SUMMER21SALE']],
], [
    'default_tax_rates' => ['txr_id'],
]);

// Tab (up to 250 items)
$user->tabPrice('price_tshirt', 5);
$user->tabPrice('price_mug', 2);
$user->invoice();

// Ad-hoc invoice
$user->invoiceFor('One Time Fee', 500);
```

### Payment Intents

```php
$payment = $user->pay($amount);
return $payment->client_secret;

// Specific payment methods only
$payment = $user->payWith($amount, ['card', 'bancontact']);
```

### Refunds

```php
$payment = $user->charge(100, $paymentMethodId);
$user->refund($payment->id);
```

## Invoices

```php
$invoices = $user->invoices();
$invoices = $user->invoicesIncludingPending();
$invoice = $user->findInvoice($invoiceId);
$invoice = $user->upcomingInvoice();
$invoice = $user->subscription('default')->upcomingInvoice();

// Preview
$invoice = $user->subscription('default')->previewInvoice('price_yearly');
$invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);

// Invoice data
$invoice->date()->toFormattedDateString();
$invoice->total();
```

### PDF Generation

```bash
composer require dompdf/dompdf
```

```php
return $request->user()->downloadInvoice($invoiceId);

return $request->user()->downloadInvoice($invoiceId, [
    'vendor' => 'Your Company',
    'product' => 'Your Product',
    'street' => 'Main Str. 1',
    'location' => '2000 Antwerp, Belgium',
    'phone' => '+32 499 00 00 00',
    'email' => 'info@example.com',
    'url' => 'https://example.com',
    'vendorVat' => 'BE123456789',
]);

return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');
```

### Custom Invoice Renderer

```php
use Laravel\Cashier\Contracts\InvoiceRenderer;
use Laravel\Cashier\Invoice;

class ApiInvoiceRenderer implements InvoiceRenderer
{
    public function render(Invoice $invoice, array $data = [], array $options = []): string
    {
        $html = $invoice->view($data)->render();
        return Http::get('https://example.com/html-to-pdf', ['html' => $html])->body();
    }
}
// Set in config: cashier.invoices.renderer
```

## Stripe Checkout

### Product Checkout

```php
return $request->user()->checkout('price_tshirt');
return $request->user()->checkout(['price_tshirt' => 15]);

return $request->user()->checkout(['price_tshirt' => 1], [
    'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
    'cancel_url' => route('checkout-cancel'),
    'metadata' => ['order_id' => $order->id],
]);

// Retrieve session on success
$session = Cashier::stripe()->checkout->sessions->retrieve($sessionId);
$orderId = $session['metadata']['order_id'];
```

### Single Charge Checkout

```php
return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
```

### Subscription Checkout

```php
return $request->user()
    ->newSubscription('default', 'price_monthly')
    ->trialDays(3)        // min 48 hours for Checkout
    ->allowPromotionCodes()
    ->checkout([
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
```

### Promotion Codes

```php
return $request->user()->allowPromotionCodes()->checkout('price_tshirt');

// Find promotion codes
$promotionCode = $user->findPromotionCode('SUMMERSALE');
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
$coupon = $promotionCode->coupon();

if ($coupon->isPercentage()) {
    $coupon->percentOff(); // 21.5%
} else {
    $coupon->amountOff();  // $5.99
}

// Apply to customer/subscription
$billable->applyCoupon('coupon_id');
$billable->applyPromotionCode('promotion_code_id');
$subscription->applyCoupon('coupon_id');
```

### Guest Checkout

```php
use Laravel\Cashier\Checkout;

return Checkout::guest()->create('price_tshirt', [
    'success_url' => route('your-success-route'),
    'cancel_url' => route('your-cancel-route'),
]);

return Checkout::guest()
    ->withPromotionCode('promo-code')
    ->create('price_tshirt', [...]);
```

### Collecting Tax IDs

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

## Handling Failed Payments

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $subscription = $user->newSubscription('default', 'price_monthly')
        ->create($paymentMethod);
} catch (IncompletePayment $exception) {
    return redirect()->route('cashier.payment', [
        $exception->payment->id,
        'redirect' => route('home'),
    ]);
}

// Payment confirmation with extra data (e.g., SEPA mandate)
$subscription->withPaymentConfirmationOptions([
    'mandate_data' => '...',
])->swap('price_xxx');
```

### Incomplete Payment Status

```php
$exception->payment->status;
$exception->payment->requiresPaymentMethod();
$exception->payment->requiresConfirmation();

// Latest payment link
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    Please confirm your payment.
</a>
```

## Stripe SDK Direct Access

```php
$stripeSubscription = $subscription->asStripeSubscription();
$stripeSubscription->application_fee_percent = 5;
$stripeSubscription->save();

$subscription->updateStripeSubscription(['application_fee_percent' => 5]);

$prices = Cashier::stripe()->prices->all();
```

## Common Middleware Pattern

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            return redirect('/billing');
        }
        return $next($request);
    }
}

// Usage
Route::get('/dashboard', fn () => ...)->middleware([Subscribed::class]);
```

## Testing

- Use Stripe **test** API keys in `phpunit.xml`:
  ```xml
  <env name="STRIPE_SECRET" value="sk_test_<your-key>"/>
  ```
- Tests hit real Stripe test API (not mocked) for best confidence
- Use [Stripe test cards](https://stripe.com/docs/testing) for different scenarios
- Use [Stripe CLI](https://stripe.com/docs/stripe-cli) for local webhook testing
- Focus tests on YOUR subscription/payment flow, not Cashier internals

## Important Notes

- Cashier 16 uses Stripe API version `2025-06-30.basil`
- `stripe_id` column collation should be `utf8_bin` (MySQL) for case-sensitive Stripe IDs
- Charge amounts are in **lowest currency denominator** (cents for USD)
- Default payment method can only be used for invoicing/subscriptions, NOT single charges
- `anchorBillingCycleOn`, proration, payment behavior do NOT work with Stripe Checkout
- Stripe Checkout trial minimum is **48 hours**
- Cancel subscriptions before deleting user models
- Never remove the last price from a multi-product subscription (cancel instead)
