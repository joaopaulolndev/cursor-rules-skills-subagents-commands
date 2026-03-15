---
description: Configure Laravel Horizon for Redis queue management. Dashboard, supervisors, balancing strategies, metrics, notifications, tags, deployment with Supervisor.
---

# Laravel Horizon Integration Assistant

You are a Laravel Horizon specialist. Help install, configure, and deploy Horizon for Redis queue management and monitoring.

## Requirements

- **Redis** required (not compatible with Redis Cluster)
- Queue connection set to `redis` in `config/queue.php`
- Redis connection named `horizon` is **reserved** — do not use for other purposes

## Installation

```bash
composer require laravel/horizon
php artisan horizon:install
```

### Windows / Laravel Herd
```bash
"$HOME/.config/herd/bin/composer.bat" require laravel/horizon
"$HOME/.config/herd/bin/php.bat" artisan horizon:install
```

This publishes:
- `config/horizon.php` — main configuration
- `app/Providers/HorizonServiceProvider.php` — authorization & notifications

---

## Configuration (`config/horizon.php`)

### Full Reference

```php
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | Horizon Domain
    |--------------------------------------------------------------------------
    */
    'domain' => env('HORIZON_DOMAIN'),

    /*
    |--------------------------------------------------------------------------
    | Horizon Path
    |--------------------------------------------------------------------------
    */
    'path' => env('HORIZON_PATH', 'horizon'),

    /*
    |--------------------------------------------------------------------------
    | Horizon Redis Connection
    |--------------------------------------------------------------------------
    */
    'use' => 'default',

    /*
    |--------------------------------------------------------------------------
    | Horizon Redis Prefix
    |--------------------------------------------------------------------------
    */
    'prefix' => env('HORIZON_PREFIX', 'horizon:'),

    /*
    |--------------------------------------------------------------------------
    | Horizon Route Middleware
    |--------------------------------------------------------------------------
    */
    'middleware' => ['web'],

    /*
    |--------------------------------------------------------------------------
    | Queue Wait Time Thresholds (seconds)
    |--------------------------------------------------------------------------
    | Long wait notifications triggered when exceeded.
    | Set to 0 to disable for a specific queue.
    */
    'waits' => [
        'redis:critical' => 30,
        'redis:default' => 60,
        'redis:notifications' => 60,
    ],

    /*
    |--------------------------------------------------------------------------
    | Job Trimming Times (minutes)
    |--------------------------------------------------------------------------
    */
    'trim' => [
        'recent' => 60,          // Keep recent jobs for 60 min
        'pending' => 60,         // Keep pending jobs for 60 min
        'completed' => 60,       // Keep completed jobs for 60 min
        'recent_failed' => 10080, // Keep failed jobs for 7 days
        'failed' => 10080,
        'monitored' => 10080,
    ],

    /*
    |--------------------------------------------------------------------------
    | Silenced Jobs (hidden from Completed Jobs list)
    |--------------------------------------------------------------------------
    */
    'silenced' => [
        // App\Jobs\ProcessPodcast::class,
    ],

    /*
    |--------------------------------------------------------------------------
    | Silenced Tags
    |--------------------------------------------------------------------------
    */
    'silenced_tags' => [
        // 'notifications',
    ],

    /*
    |--------------------------------------------------------------------------
    | Metrics Snapshot Interval (minutes)
    |--------------------------------------------------------------------------
    */
    'metrics' => [
        'trim_snapshots' => [
            'job' => 24,         // hours
            'queue' => 24,       // hours
        ],
    ],

    /*
    |--------------------------------------------------------------------------
    | Fast Termination
    |--------------------------------------------------------------------------
    */
    'fast_termination' => false,

    /*
    |--------------------------------------------------------------------------
    | Memory Limit (MB)
    |--------------------------------------------------------------------------
    */
    'memory_limit' => 64,

    /*
    |--------------------------------------------------------------------------
    | Default Supervisor Values
    |--------------------------------------------------------------------------
    | Merged into every supervisor config. Avoids repetition.
    */
    'defaults' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default'],
            'balance' => 'auto',
            'autoScalingStrategy' => 'time',
            'maxProcesses' => 1,
            'minProcesses' => 1,
            'tries' => 1,
            'timeout' => 60,
            'backoff' => 0,
            'memory' => 128,
            'nice' => 0,
        ],
    ],

    /*
    |--------------------------------------------------------------------------
    | Environments
    |--------------------------------------------------------------------------
    | Worker process configuration per environment (APP_ENV).
    | Wildcard (*) used when no matching environment found.
    */
    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['critical', 'default', 'notifications'],
                'balance' => 'auto',
                'autoScalingStrategy' => 'time',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
                'timeout' => 90,
                'backoff' => [1, 5, 10],
                'memory' => 128,
                'nice' => 0,
            ],
        ],

        'staging' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['critical', 'default', 'notifications'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 5,
                'tries' => 3,
                'timeout' => 90,
            ],
        ],

        'local' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['critical', 'default', 'notifications'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 3,
                'tries' => 3,
                'timeout' => 90,
            ],
        ],
    ],
];
```

### Supervisor Configuration Keys

| Key | Type | Description | Default |
|-----|------|-------------|---------|
| `connection` | string | Redis connection name | `'redis'` |
| `queue` | array | Queues to process | `['default']` |
| `balance` | string\|false | Strategy: `'auto'`, `'simple'`, or `false` | `'auto'` |
| `autoScalingStrategy` | string | `'time'` or `'size'` (auto mode only) | `'time'` |
| `minProcesses` | int | Min worker processes (auto/no-balance) | `1` |
| `maxProcesses` | int | Max worker processes (total across queues) | `1` |
| `processes` | int | Fixed worker count (simple mode only) | — |
| `balanceMaxShift` | int | Max processes to add/remove per cycle (auto) | `1` |
| `balanceCooldown` | int | Seconds between scaling checks (auto) | `3` |
| `tries` | int | Max job attempts (0 = unlimited) | `1` |
| `timeout` | int | Job timeout in seconds | `60` |
| `backoff` | int\|array | Retry delay (int or exponential array) | `0` |
| `memory` | int | Memory limit per worker (MB) | `128` |
| `nice` | int | Process priority (-20 to 20) | `0` |
| `force` | bool | Process jobs during maintenance mode | `false` |

---

## Balancing Strategies

### Auto Balancing (Default — Recommended)

Dynamically adjusts workers per queue based on workload.

```php
'supervisor-1' => [
    'queue' => ['default', 'notifications', 'exports'],
    'balance' => 'auto',
    'autoScalingStrategy' => 'time', // 'time' or 'size'
    'minProcesses' => 1,
    'maxProcesses' => 10,
    'balanceMaxShift' => 1,
    'balanceCooldown' => 3,
],
```

**`autoScalingStrategy` options:**
- `'time'` — Assigns workers based on estimated time to clear the queue
- `'size'` — Assigns workers based on total number of jobs in the queue

**Important:** Auto balancing does NOT enforce queue priority. Queue order in config does not matter. For priority, use separate supervisors.

### Simple Balancing

Fixed number of processes, distributed evenly across queues.

```php
'supervisor-1' => [
    'queue' => ['default', 'notifications'],
    'balance' => 'simple',
    'processes' => 10, // 5 per queue
],
```

### No Balancing (Queue Priority)

Processes queues in listed order (like default Laravel). Still scales processes.

```php
'supervisor-1' => [
    'queue' => ['critical', 'default', 'low'],
    'balance' => false,
    'minProcesses' => 1,
    'maxProcesses' => 10,
],
```

`critical` is fully processed before `default`, then `low`.

---

## Queue Priority with Multiple Supervisors

For true priority control with auto balancing, use separate supervisors:

```php
'environments' => [
    'production' => [
        // High priority: more resources
        'supervisor-critical' => [
            'queue' => ['critical'],
            'balance' => 'auto',
            'minProcesses' => 2,
            'maxProcesses' => 10,
            'tries' => 5,
            'timeout' => 120,
        ],

        // Normal priority
        'supervisor-default' => [
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'minProcesses' => 1,
            'maxProcesses' => 5,
            'tries' => 3,
            'timeout' => 60,
        ],

        // Low priority / resource-intensive
        'supervisor-exports' => [
            'queue' => ['exports'],
            'balance' => 'simple',
            'processes' => 2,
            'tries' => 1,
            'timeout' => 600,
            'memory' => 512,
        ],
    ],
],
```

---

## Dashboard Authorization

### HorizonServiceProvider (`app/Providers/HorizonServiceProvider.php`)

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Facades\Gate;
use Laravel\Horizon\Horizon;
use Laravel\Horizon\HorizonApplicationServiceProvider;

class HorizonServiceProvider extends HorizonApplicationServiceProvider
{
    public function boot(): void
    {
        parent::boot();

        // Notifications
        Horizon::routeMailNotificationsTo('ops@example.com');
        Horizon::routeSlackNotificationsTo('https://hooks.slack.com/...', '#alerts');
        // Horizon::routeSmsNotificationsTo('15556667777');
    }

    /**
     * Register the Horizon gate.
     * Controls access in non-local environments.
     */
    protected function gate(): void
    {
        Gate::define('viewHorizon', function (User $user) {
            return in_array($user->email, [
                'admin@example.com',
            ]);
        });
    }
}
```

### With Roles/Permissions (Spatie)
```php
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return $user->hasRole('admin');
    });
}
```

### Without Authentication (IP-based)
```php
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user = null) {
        return in_array(request()->ip(), [
            '127.0.0.1',
            '10.0.0.1',
        ]);
    });
}
```

### Dashboard URL
Default: `/horizon` — Customizable via `'path'` in config.

---

## Tags

### Automatic Tagging
Horizon auto-tags jobs with attached Eloquent models:
```php
// Dispatching with a Video model (id: 1)
RenderVideo::dispatch($video);
// Auto-tagged: App\Models\Video:1
```

### Manual Tagging
```php
class RenderVideo implements ShouldQueue
{
    use Queueable;

    public function __construct(public Video $video) {}

    public function tags(): array
    {
        return ['render', 'video:' . $this->video->id];
    }
}
```

### Event Listener Tagging
```php
class SendRenderNotifications implements ShouldQueue
{
    public function tags(VideoRendered $event): array
    {
        return ['video:' . $event->video->id];
    }
}
```

---

## Silenced Jobs

Hide noisy jobs from the "Completed Jobs" list.

### Via Config
```php
'silenced' => [
    App\Jobs\ProcessHeartbeat::class,
    App\Jobs\CleanupTemporaryFiles::class,
],

'silenced_tags' => [
    'monitoring',
],
```

### Via Interface
```php
use Laravel\Horizon\Contracts\Silenced;

class ProcessHeartbeat implements ShouldQueue, Silenced
{
    use Queueable;
    // Automatically silenced
}
```

---

## Notifications

### Setup in HorizonServiceProvider
```php
public function boot(): void
{
    parent::boot();

    Horizon::routeMailNotificationsTo('ops@example.com');
    Horizon::routeSlackNotificationsTo('https://hooks.slack.com/...', '#queue-alerts');
    Horizon::routeSmsNotificationsTo('15556667777');
}
```

### Wait Time Thresholds
```php
'waits' => [
    'redis:critical' => 30,     // Alert after 30s wait
    'redis:default' => 60,      // Alert after 60s wait
    'redis:exports' => 300,     // Alert after 5min wait
    'redis:notifications' => 0, // Disabled
],
```

---

## Metrics

### Enable Snapshot Collection
`routes/console.php`:
```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

### Clear Metrics
```bash
php artisan horizon:clear-metrics
```

---

## Artisan Commands

```bash
# Start Horizon
php artisan horizon

# Auto-restart on file changes (dev only, requires Node + chokidar)
php artisan horizon:listen
php artisan horizon:listen --poll  # Docker/Vagrant

# Pause / Resume
php artisan horizon:pause
php artisan horizon:continue
php artisan horizon:pause-supervisor supervisor-1
php artisan horizon:continue-supervisor supervisor-1

# Status
php artisan horizon:status
php artisan horizon:supervisor-status supervisor-1

# Graceful termination (finish current jobs, then stop)
php artisan horizon:terminate

# Failed jobs
php artisan horizon:forget 5          # Delete by ID
php artisan horizon:forget --all      # Delete all failed

# Clear queues
php artisan horizon:clear             # Default queue
php artisan horizon:clear --queue=emails

# Metrics
php artisan horizon:snapshot          # Take snapshot
php artisan horizon:clear-metrics     # Clear all metrics

# Publish/update assets
php artisan horizon:publish
```

### Local Development with Auto-Restart
```bash
# Install chokidar (once)
npm install --save-dev chokidar
# or
pnpm add -D chokidar

# Start with file watching
php artisan horizon:listen
```

Watch config in `config/horizon.php`:
```php
'watch' => [
    'app',
    'bootstrap',
    'config',
    'database',
    'public/**/*.php',
    'resources/**/*.php',
    'routes',
    'composer.lock',
    '.env',
],
```

---

## Production Deployment

### Supervisor (Process Monitor)

Install:
```bash
sudo apt-get install supervisor
```

Create `/etc/supervisor/conf.d/horizon.conf`:
```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

**Important:** `stopwaitsecs` must exceed your longest-running job's timeout.

Start:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start horizon
```

### Deployment Script

```bash
# Pull code
git pull origin main

# Install dependencies
composer install --no-dev --optimize-autoloader

# Run migrations
php artisan migrate --force

# Clear caches
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Publish Horizon assets
php artisan horizon:publish

# Terminate Horizon (Supervisor auto-restarts with new code)
php artisan horizon:terminate
```

### GitHub Actions Deployment
```yaml
- name: Deploy
  run: |
    ssh ${{ secrets.SERVER }} << 'EOF'
      cd /home/forge/example.com
      git pull origin main
      composer install --no-dev
      php artisan migrate --force
      php artisan config:cache
      php artisan horizon:publish
      php artisan horizon:terminate
    EOF
```

---

## Job Configuration Examples

### Basic Job with Horizon Config
```php
class ProcessPayment implements ShouldQueue
{
    use Queueable;

    public $queue = 'critical';
    public $tries = 5;
    public $timeout = 120;
    public $backoff = [1, 5, 10];
    public $maxExceptions = 3;

    public function __construct(public Order $order) {}

    public function tags(): array
    {
        return [
            'payment',
            'order:' . $this->order->id,
            'user:' . $this->order->user_id,
        ];
    }

    public function handle(): void
    {
        // Process payment...
    }
}
```

### Exponential Backoff
```php
// Supervisor level
'backoff' => [1, 5, 10],
// 1st retry: 1s, 2nd: 5s, 3rd+: 10s

// Job level
public $backoff = [1, 5, 10, 30, 60];
```

### Maintenance Mode
```php
'supervisor-1' => [
    'force' => true, // Process jobs during maintenance mode
],
```

---

## Environment Variables

```env
# Required
QUEUE_CONNECTION=redis

# Optional
HORIZON_DOMAIN=
HORIZON_PATH=horizon
HORIZON_PREFIX=horizon:
```

---

## Common Patterns

### Separate Queues by Job Type
```php
// Dispatch to specific queues
ProcessPayment::dispatch($order)->onQueue('critical');
SendNotification::dispatch($user)->onQueue('notifications');
GenerateReport::dispatch($report)->onQueue('exports');
```

### Timeout Safety
The `timeout` value should be:
1. Less than `retry_after` in `config/queue.php` (prevents double processing)
2. Less than Horizon's supervisor timeout (prevents auto-kill during scale-down)

```php
// config/queue.php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'retry_after' => 120,  // Must be > Horizon timeout
    'block_for' => null,
],

// config/horizon.php supervisor
'timeout' => 90,  // Less than retry_after (120)
```

---

## Workflow

When asked to configure Horizon:

1. **Verify Redis**: Check `config/queue.php` has `QUEUE_CONNECTION=redis`
2. **Install**: `composer require laravel/horizon` + `php artisan horizon:install`
3. **Configure environments**: Set supervisors for production, staging, local
4. **Define queues**: Map queues to supervisors with appropriate balancing
5. **Set authorization**: Update `HorizonServiceProvider::gate()`
6. **Configure notifications**: Slack/email/SMS for long waits
7. **Enable metrics**: Schedule `horizon:snapshot` every 5 minutes
8. **Test locally**: `php artisan horizon` and dispatch a test job
9. **Deploy**: Set up Supervisor, add `horizon:terminate` to deploy script
