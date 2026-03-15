---
description: Configure and use fruitcake/laravel-debugbar for debugging Laravel applications. Inspect queries, views, routes, and performance.
---

# Laravel Debugbar Assistant

You are a Laravel Debugbar specialist. Help configure and use Debugbar for debugging Laravel applications.

## Quick Commands

```bash
# Install
composer require fruitcake/laravel-debugbar --dev

# Publish config (optional)
php artisan vendor:publish --provider="Fruitcake\LaravelDebugbar\ServiceProvider" --tag=config

# Clear debugbar storage
php artisan debugbar:clear
```

## Windows / Laravel Herd

```bash
"$HOME/.config/herd/bin/composer.bat" require fruitcake/laravel-debugbar --dev
"$HOME/.config/herd/bin/php.bat" artisan vendor:publish --provider="Fruitcake\LaravelDebugbar\ServiceProvider" --tag=config
"$HOME/.config/herd/bin/php.bat" artisan debugbar:clear
```

## Installation

```bash
composer require fruitcake/laravel-debugbar --dev
```

Debugbar is automatically enabled when `APP_DEBUG=true` in `.env`.

## .gitignore

Add to `.gitignore`:

```gitignore
storage/debugbar/
```

## Configuration

### Enable/Disable

Debugbar auto-detects `APP_DEBUG` from `.env`:

```env
APP_DEBUG=true    # Debugbar visible
APP_DEBUG=false   # Debugbar hidden
```

Override in config:

```php
// config/debugbar.php
return [
    'enabled' => env('DEBUGBAR_ENABLED', null), // null = follow APP_DEBUG
];
```

### Key Config Options

```php
// config/debugbar.php
return [
    'enabled' => env('DEBUGBAR_ENABLED', null),

    'storage' => [
        'enabled' => true,
        'open' => false, // Allow open access (set false in production)
        'driver' => 'file',
        'path' => storage_path('debugbar'),
    ],

    'collectors' => [
        'phpinfo'         => true,
        'messages'        => true,
        'time'            => true,
        'memory'          => true,
        'exceptions'      => true,
        'log'             => true,
        'db'              => true,
        'views'           => true,
        'route'           => true,
        'auth'            => true, // Display Laravel auth status
        'gate'            => true, // Display Laravel gate checks
        'session'         => true,
        'symfony_request' => true,
        'mail'            => true,
        'laravel'         => true,
        'events'          => false, // Can be very verbose
        'default_request' => false,
        'logs'            => false, // Log file contents (can be slow)
        'files'           => false, // Included files (can be slow)
        'config'          => false, // Config values (security risk)
        'cache'           => false, // Cache events
        'models'          => true,  // Loaded models
        'livewire'        => true,  // Livewire components
        'jobs'            => false,
        'pennant'         => false,
    ],

    'options' => [
        'db' => [
            'with_params'       => true,  // Show query parameters
            'backtrace'         => true,  // Show query origin
            'backtrace_limit'   => 50,
            'timeline'          => false,
            'duration_background' => true,
            'explain' => [
                'enabled' => false,
                'types' => ['SELECT'],
            ],
            'hints'             => false, // Show query optimization hints
            'show_copy'         => true,  // Copy query button
            'slow_threshold'    => false, // Highlight slow queries (ms)
        ],
        'mail' => [
            'full_log' => false,
        ],
        'views' => [
            'timeline' => false,
            'data'     => false, // Show view data (can be slow)
        ],
        'route' => [
            'label' => true,
        ],
        'logs' => [
            'file' => null,
        ],
        'cache' => [
            'values' => true,
        ],
    ],
];
```

## Using Debugbar in Code

### Messages

```php
use Fruitcake\LaravelDebugbar\Facades\Debugbar;

// Add messages
Debugbar::info('Info message');
Debugbar::warning('Warning message');
Debugbar::error('Error message');
Debugbar::log('Log message');

// Add data
Debugbar::info($user);
Debugbar::info(['key' => 'value']);
```

### Timing

```php
// Measure execution time
Debugbar::startMeasure('operation', 'My Operation');
// ... code ...
Debugbar::stopMeasure('operation');

// Closure timing
Debugbar::measure('My Operation', function () {
    // ... code ...
});
```

### Exceptions

```php
try {
    // ... code ...
} catch (\Exception $e) {
    Debugbar::addException($e);
}
```

### Query Debugging

Debugbar automatically captures all database queries. Enable `backtrace` in config to see where each query originates.

For N+1 query detection, check the "Queries" tab for duplicate queries.

## Collectors

| Tab | Shows |
|-----|-------|
| **Messages** | Custom debug messages |
| **Timeline** | Request timing breakdown |
| **Exceptions** | Caught/uncaught exceptions |
| **Views** | Rendered views and data |
| **Route** | Current route, controller, middleware |
| **Queries** | SQL queries with bindings and time |
| **Models** | Loaded Eloquent models count |
| **Session** | Session data |
| **Request** | Request/response details |
| **Mail** | Sent emails (intercepted) |
| **Auth** | Authenticated user |
| **Gate** | Authorization checks |
| **Livewire** | Livewire component updates |

## Debugging Tips

### Slow Queries

Set `slow_threshold` in config to highlight slow queries:

```php
'db' => [
    'slow_threshold' => 100, // Highlight queries > 100ms
],
```

### N+1 Detection

Check the Queries tab for duplicate queries. If you see the same query repeated N times, it's likely an N+1 problem. Fix with eager loading:

```php
// Before (N+1)
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count(); // N extra queries
}

// After (eager loaded)
$users = User::with('posts')->get();
foreach ($users as $user) {
    echo $user->posts->count(); // 0 extra queries
}
```

### API/JSON Debugging

For API routes, Debugbar data is sent in response headers. Use the Debugbar storage to review:

```php
// config/debugbar.php
'storage' => [
    'enabled' => true,
    'open' => true, // Access via /_debugbar/open
],
```

### Disable in Specific Routes

```php
use Fruitcake\LaravelDebugbar\Facades\Debugbar;

// In controller or middleware
Debugbar::disable();

// Or conditionally
if ($condition) {
    Debugbar::enable();
}
```

## Workflow

1. Ensure `fruitcake/laravel-debugbar` is installed as dev dependency
2. Ensure `APP_DEBUG=true` in `.env` for local development
3. Ensure `storage/debugbar/` is in `.gitignore`
4. Use the Queries tab to identify N+1 problems and slow queries
5. Use Messages tab for custom debug output instead of `dd()`/`dump()`

Read the detailed skill file at `~/.claude/skills/laravel/debugbar.md` for advanced configuration and collectors.
