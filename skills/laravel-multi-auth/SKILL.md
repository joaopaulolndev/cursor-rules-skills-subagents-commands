---
description: Configure multi auth guard in Laravel with separate tables, models, guards, providers, middleware, and password reset for different user types.
---

# Laravel Multi Auth Guard Assistant

You are a Laravel multi-authentication specialist. Help configure multiple auth guards with separate tables for different user types (admin, user, member, etc.).

## Overview

Multi auth guard permite autenticar diferentes tipos de usuarios com tabelas, models, login pages e dashboards separados. Cada guard tem seu proprio provider, model e configuracao de password reset.

## Quick Setup Checklist

1. Criar migration e model para cada tipo de usuario
2. Configurar guards e providers em `config/auth.php`
3. Configurar password reset brokers
4. Criar middleware customizado
5. Definir rotas separadas por guard
6. Criar controllers de autenticacao por guard

## Step 1: Migration

```php
<?php
// database/migrations/xxxx_create_admins_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('admins', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('admins');
    }
};
```

## Step 2: Model

```php
<?php
// app/Models/Admin.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class Admin extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
```

**Importante:** O model DEVE estender `Illuminate\Foundation\Auth\User as Authenticatable`, NAO `Illuminate\Database\Eloquent\Model`.

## Step 3: Guards e Providers (`config/auth.php`)

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'admin' => [
        'driver' => 'session',
        'provider' => 'admins',
    ],
],

'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => App\Models\User::class,
    ],
    'admins' => [
        'driver' => 'eloquent',
        'model' => App\Models\Admin::class,
    ],
],

'passwords' => [
    'users' => [
        'provider' => 'users',
        'table' => 'password_reset_tokens',
        'expire' => 60,
        'throttle' => 60,
    ],
    'admins' => [
        'provider' => 'admins',
        'table' => 'password_reset_tokens',
        'expire' => 60,
        'throttle' => 60,
    ],
],
```

## Step 4: Middleware Customizado

### Authenticate Middleware

```php
<?php
// app/Http/Middleware/Authenticate.php

namespace App\Http\Middleware;

use Illuminate\Auth\AuthenticationException;
use Illuminate\Auth\Middleware\Authenticate as MiddlewareAuthenticate;

class Authenticate extends MiddlewareAuthenticate
{
    protected function unauthenticated($request, array $guards): void
    {
        throw new AuthenticationException(
            redirectTo: match (true) {
                in_array('admin', $guards) => route('admin.login'),
                default => route('login'),
            }
        );
    }
}
```

### RedirectIfAuthenticated Middleware

```php
<?php
// app/Http/Middleware/RedirectIfAuthenticated.php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Auth\Middleware\RedirectIfAuthenticated as MiddlewareRedirectIfAuthenticated;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpFoundation\Response;

class RedirectIfAuthenticated extends MiddlewareRedirectIfAuthenticated
{
    public function handle(Request $request, Closure $next, string ...$guards): Response
    {
        $guards = empty($guards) ? [null] : $guards;

        foreach ($guards as $guard) {
            if (Auth::guard($guard)->check()) {
                return redirect()->route(match ($guard) {
                    'admin' => 'admin.dashboard',
                    default => 'dashboard',
                });
            }
        }

        return $next($request);
    }
}
```

### Registrar em `bootstrap/app.php`

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'auth' => \App\Http\Middleware\Authenticate::class,
    ]);
})
```

## Step 5: Rotas

```php
// routes/web.php

// === User Routes ===
Route::middleware('guest')->group(function () {
    Route::get('login', [LoginController::class, 'showForm'])->name('login');
    Route::post('login', [LoginController::class, 'login']);
});

Route::middleware('auth')->group(function () {
    Route::get('dashboard', DashboardController::class)->name('dashboard');
    Route::post('logout', [LoginController::class, 'logout'])->name('logout');
});

// === Admin Routes ===
Route::prefix('admin')->group(function () {
    Route::middleware('guest:admin')->group(function () {
        Route::get('login', [AdminLoginController::class, 'showForm'])->name('admin.login');
        Route::post('login', [AdminLoginController::class, 'login'])->name('admin.login.submit');
    });

    Route::middleware('auth:admin')->group(function () {
        Route::get('dashboard', AdminDashboardController::class)->name('admin.dashboard');
        Route::post('logout', [AdminLoginController::class, 'logout'])->name('admin.logout');
    });
});
```

## Step 6: Controllers

### AdminLoginController

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AdminLoginController extends Controller
{
    public function showForm()
    {
        return view('admin.login');
    }

    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (Auth::guard('admin')->attempt($credentials, $request->boolean('remember'))) {
            $request->session()->regenerate();
            return redirect()->intended(route('admin.dashboard'));
        }

        return back()->withErrors([
            'email' => __('auth.failed'),
        ])->onlyInput('email');
    }

    public function logout(Request $request)
    {
        Auth::guard('admin')->logout();
        $request->session()->invalidate();
        $request->session()->regenerateToken();
        return redirect()->route('admin.login');
    }
}
```

## API Guards (Sanctum)

```php
// config/auth.php
'guards' => [
    'api' => [
        'driver' => 'sanctum',
        'provider' => 'users',
    ],
    'admin-api' => [
        'driver' => 'sanctum',
        'provider' => 'admins',
    ],
],
```

```php
// routes/api.php
Route::middleware('auth:admin-api')->prefix('admin')->group(function () {
    Route::get('/profile', fn (Request $request) => $request->user());
});
```

## Helpers

```php
// Verificar autenticacao por guard
Auth::guard('admin')->check();

// Obter usuario do guard
Auth::guard('admin')->user();

// Attempt com guard especifico
Auth::guard('admin')->attempt($credentials);

// Login manual
Auth::guard('admin')->login($admin);

// Logout
Auth::guard('admin')->logout();
```

## Reference Files

- `~/.claude/skills/laravel/multi-auth-guard.md`
