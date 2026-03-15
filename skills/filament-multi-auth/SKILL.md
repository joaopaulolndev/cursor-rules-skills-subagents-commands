---
description: Configure multi auth guard in Filament with separate panels, tables, models, and guards for different user types (v3, v4, v5).
---

# Filament Multi Auth Guard Assistant

You are a Filament multi-authentication specialist. Help configure multiple panels with separate auth guards, tables, and models for different user types.

## Overview

No Filament, cada Panel pode ter seu proprio auth guard, permitindo autenticacao separada para diferentes tipos de usuarios (admin, member, vendor, etc.) com tabelas e login pages independentes.

## Quick Setup Checklist

1. Configurar guards e providers no Laravel (`config/auth.php`)
2. Criar migration e model para cada tipo de usuario
3. Criar Panel Provider para cada tipo
4. Configurar `authGuard()` no Panel
5. Implementar `FilamentUser` no model
6. Configurar password reset broker

---

## Step 1: Guards e Providers (`config/auth.php`)

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
    'member' => [
        'driver' => 'session',
        'provider' => 'members',
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
    'members' => [
        'driver' => 'eloquent',
        'model' => App\Models\Member::class,
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
    'members' => [
        'provider' => 'members',
        'table' => 'password_reset_tokens',
        'expire' => 60,
        'throttle' => 60,
    ],
],
```

## Step 2: Model

```php
<?php
// app/Models/Admin.php

namespace App\Models;

use Filament\Models\Contracts\FilamentUser;
use Filament\Panel;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class Admin extends Authenticatable implements FilamentUser
{
    use HasFactory, Notifiable;

    protected $fillable = ['name', 'email', 'password'];

    protected $hidden = ['password', 'remember_token'];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

    public function canAccessPanel(Panel $panel): bool
    {
        return true;
    }
}
```

## Step 3: Criar Panel

```bash
php artisan make:filament-panel admin
php artisan make:filament-panel member
```

---

## Configuracao por Versao do Filament

### Filament v3

```php
<?php
// app/Providers/Filament/AdminPanelProvider.php

namespace App\Providers\Filament;

use Filament\Panel;
use Filament\PanelProvider;
use Filament\Pages\Dashboard;
use Filament\Support\Colors\Color;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->id('admin')
            ->path('admin')
            ->colors(['primary' => Color::Amber])
            ->authGuard('admin')
            ->login()
            ->registration()
            ->passwordReset()
            ->authPasswordBroker('admins')
            ->emailVerification()
            ->profile()
            ->pages([Dashboard::class])
            ->discoverResources(
                in: app_path('Filament/Admin/Resources'),
                for: 'App\\Filament\\Admin\\Resources'
            )
            ->discoverPages(
                in: app_path('Filament/Admin/Pages'),
                for: 'App\\Filament\\Admin\\Pages'
            );
    }
}
```

### Filament v4

```php
<?php
// app/Providers/Filament/AdminPanelProvider.php

namespace App\Providers\Filament;

use Filament\Panel;
use Filament\PanelProvider;
use Filament\Pages\Dashboard;
use Filament\Support\Colors\Color;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->id('admin')
            ->path('admin')
            ->colors(['primary' => Color::Amber])
            ->authGuard('admin')
            ->login()
            ->registration()
            ->passwordReset()
            ->authPasswordBroker('admins')
            ->emailVerification()
            ->profile()
            ->pages([Dashboard::class])
            ->discoverResources(
                in: app_path('Filament/Admin/Resources'),
                for: 'App\\Filament\\Admin\\Resources'
            )
            ->discoverPages(
                in: app_path('Filament/Admin/Pages'),
                for: 'App\\Filament\\Admin\\Pages'
            );
    }
}
```

**Mudancas no v4:**
- `FilamentUser` contract continua obrigatorio em producao
- MFA disponivel via `->multiFactorAuthentication()`
- Mesma API de `authGuard()` e `authPasswordBroker()`

### Filament v5

```php
<?php
// app/Providers/Filament/AdminPanelProvider.php

namespace App\Providers\Filament;

use Filament\Panel;
use Filament\PanelProvider;
use Filament\Pages\Dashboard;
use Filament\Support\Colors\Color;

class AdminPanelProvider extends PanelProvider
{
    public function panel(Panel $panel): Panel
    {
        return $panel
            ->id('admin')
            ->path('admin')
            ->colors(['primary' => Color::Amber])
            ->authGuard('admin')
            ->login()
            ->registration()
            ->passwordReset()
            ->authPasswordBroker('admins')
            ->emailVerification()
            ->profile()
            ->pages([Dashboard::class])
            ->discoverResources(
                in: app_path('Filament/Admin/Resources'),
                for: 'App\\Filament\\Admin\\Resources'
            )
            ->discoverPages(
                in: app_path('Filament/Admin/Pages'),
                for: 'App\\Filament\\Admin\\Pages'
            );
    }
}
```

**Mudancas no v5:**
- Requer Livewire 4.x
- Mesma API de panel configuration
- MFA com `AppAuthentication` e `EmailAuthentication`

---

## Abordagem: Tabela Separada vs Tabela Unica

### Tabela Separada (Recomendado para isolamento total)

Cada tipo de usuario tem sua propria tabela e model:

```
admins table → Admin model → admin guard → Admin Panel
members table → Member model → member guard → Member Panel
users table → User model → web guard → App Panel
```

### Tabela Unica com Global Scope

Mesma tabela `users`, models com scope que filtram por role:

```php
class Admin extends User
{
    protected $table = 'users';

    protected static function booted(): void
    {
        static::addGlobalScope('admin', function (Builder $builder) {
            $builder->where('role', 'admin');
        });
    }
}
```

### Tabela Unica com Spatie Permission

```php
class Member extends User
{
    protected $table = 'users';

    protected static function booted(): void
    {
        static::addGlobalScope('is_member', function (Builder $builder) {
            $builder->whereHas('roles', fn ($q) => $q->where('name', 'member'));
        });
    }
}
```

**Importante:** Ao usar tabela unica, lembrar de atribuir a role durante o registro (listener ou extendendo a pagina Register).

---

## canAccessPanel por Panel ID

```php
public function canAccessPanel(Panel $panel): bool
{
    return match ($panel->getId()) {
        'admin' => $this->hasRole('admin'),
        'member' => $this->hasRole('member'),
        default => true,
    };
}
```

## Customizar Pagina de Login

```php
use Filament\Pages\Auth\Login;

class AdminLogin extends Login
{
    public function form(Form $form): Form
    {
        return $form->schema([
            $this->getEmailFormComponent(),
            $this->getPasswordFormComponent(),
            $this->getRememberFormComponent(),
        ]);
    }
}

// No PanelProvider
->login(AdminLogin::class)
```

## Customizar Registro com Role

```php
use Filament\Pages\Auth\Register;

class MemberRegister extends Register
{
    protected function afterRegister(): void
    {
        $this->getUser()->assignRole('member');
    }
}

// No PanelProvider
->registration(MemberRegister::class)
```

## Reference Files

- `~/.claude/skills/filament/multi-auth-guard.md`
- `~/.claude/skills/laravel/multi-auth-guard.md`
