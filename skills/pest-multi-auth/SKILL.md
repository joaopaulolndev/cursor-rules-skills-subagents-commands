---
description: Write Pest tests for multi auth guard covering login, logout, middleware, routes, Filament panels, and Sanctum API with separate user tables.
---

# Pest Multi Auth Guard Testing Assistant

You are a Pest testing specialist for multi auth guard scenarios. Help write comprehensive tests for applications with multiple authentication guards, separate user tables, and Filament panels.

## Quick Commands

```bash
# Run all auth tests
vendor/bin/pest tests/Feature/Auth

# Run specific guard tests
vendor/bin/pest --filter="admin guard"

# Run with group
vendor/bin/pest --group=multi-auth

# Stop on first failure
vendor/bin/pest --stop-on-failure
```

## Setup: tests/Pest.php

```php
<?php

use App\Models\Admin;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class)->in('Feature');

// Helper: criar user autenticado
function asUser(?User $user = null): User
{
    $user ??= User::factory()->create();
    test()->actingAs($user, 'web');

    return $user;
}

// Helper: criar admin autenticado
function asAdmin(?Admin $admin = null): Admin
{
    $admin ??= Admin::factory()->create();
    test()->actingAs($admin, 'admin');

    return $admin;
}
```

---

## Testando Guard Web (Users)

```php
<?php
// tests/Feature/Auth/UserAuthTest.php

use App\Models\User;

describe('User Authentication (web guard)', function () {
    it('shows login page to guests', function () {
        $this->get(route('login'))
            ->assertOk()
            ->assertSee('Login');
    });

    it('can login with valid credentials', function () {
        $user = User::factory()->create([
            'password' => bcrypt('password'),
        ]);

        $this->post(route('login'), [
            'email' => $user->email,
            'password' => 'password',
        ])
            ->assertRedirect(route('dashboard'));

        $this->assertAuthenticatedAs($user, 'web');
    });

    it('cannot login with wrong password', function () {
        $user = User::factory()->create();

        $this->post(route('login'), [
            'email' => $user->email,
            'password' => 'wrong-password',
        ])
            ->assertSessionHasErrors('email');

        $this->assertGuest('web');
    });

    it('can access dashboard when authenticated', function () {
        asUser();

        $this->get(route('dashboard'))
            ->assertOk();
    });

    it('cannot access dashboard as guest', function () {
        $this->get(route('dashboard'))
            ->assertRedirect(route('login'));
    });

    it('can logout', function () {
        asUser();

        $this->post(route('logout'))
            ->assertRedirect(route('login'));

        $this->assertGuest('web');
    });
});
```

## Testando Guard Admin

```php
<?php
// tests/Feature/Auth/AdminAuthTest.php

use App\Models\Admin;

describe('Admin Authentication (admin guard)', function () {
    it('shows admin login page to guests', function () {
        $this->get(route('admin.login'))
            ->assertOk();
    });

    it('can login with valid credentials', function () {
        $admin = Admin::factory()->create([
            'password' => bcrypt('password'),
        ]);

        $this->post(route('admin.login.submit'), [
            'email' => $admin->email,
            'password' => 'password',
        ])
            ->assertRedirect(route('admin.dashboard'));

        $this->assertAuthenticatedAs($admin, 'admin');
    });

    it('cannot login with wrong password', function () {
        $admin = Admin::factory()->create();

        $this->post(route('admin.login.submit'), [
            'email' => $admin->email,
            'password' => 'wrong-password',
        ])
            ->assertSessionHasErrors('email');

        $this->assertGuest('admin');
    });

    it('can access admin dashboard when authenticated', function () {
        asAdmin();

        $this->get(route('admin.dashboard'))
            ->assertOk();
    });

    it('cannot access admin dashboard as guest', function () {
        $this->get(route('admin.dashboard'))
            ->assertRedirect(route('admin.login'));
    });

    it('can logout', function () {
        asAdmin();

        $this->post(route('admin.logout'))
            ->assertRedirect(route('admin.login'));

        $this->assertGuest('admin');
    });
});
```

## Testando Isolamento entre Guards

```php
<?php
// tests/Feature/Auth/GuardIsolationTest.php

use App\Models\Admin;
use App\Models\User;

describe('Guard Isolation', function () {
    it('user cannot access admin routes', function () {
        asUser();

        $this->get(route('admin.dashboard'))
            ->assertRedirect(route('admin.login'));
    });

    it('admin cannot access user routes via web guard', function () {
        asAdmin();

        $this->get(route('dashboard'))
            ->assertRedirect(route('login'));
    });

    it('user authenticated in web guard is guest in admin guard', function () {
        asUser();

        $this->assertAuthenticated('web');
        $this->assertGuest('admin');
    });

    it('admin authenticated in admin guard is guest in web guard', function () {
        asAdmin();

        $this->assertAuthenticated('admin');
        $this->assertGuest('web');
    });

    it('user credentials do not work on admin login', function () {
        $user = User::factory()->create([
            'email' => 'user@test.com',
            'password' => bcrypt('password'),
        ]);

        $this->post(route('admin.login.submit'), [
            'email' => 'user@test.com',
            'password' => 'password',
        ])
            ->assertSessionHasErrors();

        $this->assertGuest('admin');
    });

    it('can be authenticated in both guards simultaneously', function () {
        $user = User::factory()->create();
        $admin = Admin::factory()->create();

        $this->actingAs($user, 'web');
        $this->actingAs($admin, 'admin');

        $this->assertAuthenticatedAs($user, 'web');
        $this->assertAuthenticatedAs($admin, 'admin');
    });
});
```

## Testando Middleware de Redirect

```php
<?php
// tests/Feature/Auth/RedirectMiddlewareTest.php

describe('RedirectIfAuthenticated Middleware', function () {
    it('redirects authenticated user from login to dashboard', function () {
        asUser();

        $this->get(route('login'))
            ->assertRedirect(route('dashboard'));
    });

    it('redirects authenticated admin from admin login to admin dashboard', function () {
        asAdmin();

        $this->get(route('admin.login'))
            ->assertRedirect(route('admin.dashboard'));
    });

    it('does not redirect admin from user login page', function () {
        asAdmin();

        $this->get(route('login'))
            ->assertOk();
    });
});
```

## Testando Filament Panels (Multi Guard)

```php
<?php
// tests/Feature/Filament/AdminPanelTest.php

use App\Models\Admin;
use Filament\Facades\Filament;
use function Pest\Livewire\livewire;

beforeEach(function () {
    $this->admin = Admin::factory()->create();
    $this->actingAs($this->admin, 'admin');
    Filament::setCurrentPanel(Filament::getPanel('admin'));
});

it('can render admin dashboard', function () {
    $this->get('/admin')
        ->assertSuccessful();
});

it('can render admin resource list', function () {
    $this->get(\App\Filament\Admin\Resources\UserResource::getUrl('index'))
        ->assertSuccessful();
});

it('redirects unauthenticated to admin login', function () {
    auth('admin')->logout();

    $this->get('/admin')
        ->assertRedirect('/admin/login');
});
```

## Testando API com Sanctum (Multi Guard)

```php
<?php
// tests/Feature/Auth/ApiAuthTest.php

use App\Models\Admin;
use App\Models\User;
use Laravel\Sanctum\Sanctum;

describe('API Multi Auth (Sanctum)', function () {
    it('user can access api with token', function () {
        Sanctum::actingAs(
            User::factory()->create(),
            ['*'],
            'sanctum'
        );

        $this->getJson('/api/user')
            ->assertOk();
    });

    it('admin can access admin api with token', function () {
        Sanctum::actingAs(
            Admin::factory()->create(),
            ['*'],
            'admin-api'
        );

        $this->getJson('/api/admin/profile')
            ->assertOk();
    });

    it('user token cannot access admin api', function () {
        Sanctum::actingAs(
            User::factory()->create(),
            ['*'],
            'sanctum'
        );

        $this->getJson('/api/admin/profile')
            ->assertForbidden();
    });
});
```

## Dataset para Multiplos Guards

```php
dataset('guards', [
    'web guard' => ['web', fn () => User::factory()->create()],
    'admin guard' => ['admin', fn () => Admin::factory()->create()],
]);

it('can authenticate via any guard', function (string $guard, $user) {
    $this->actingAs($user, $guard);
    $this->assertAuthenticated($guard);
})->with('guards');
```

## Reference Files

- `~/.claude/skills/laravel/pest-multi-auth-guard.md`
- `~/.claude/skills/laravel/pest-testing.md`
- `~/.claude/skills/laravel/multi-auth-guard.md`
