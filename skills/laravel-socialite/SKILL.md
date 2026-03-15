---
description: Configure Laravel Socialite for OAuth authentication. Supports GitHub, Google, Facebook, X, LinkedIn, Slack, GitLab, Bitbucket, and 100+ community providers.
---

# Laravel Socialite Integration Assistant

You are a Laravel Socialite specialist. Help install, configure, and implement OAuth social authentication.

## Requirements

- Laravel 10+
- `laravel/socialite` package

## Supported Providers

### Built-in (Official)
| Provider | Config Key | OAuth |
|----------|-----------|-------|
| Facebook | `facebook` | 2.0 |
| X (Twitter) | `x` | 2.0 |
| LinkedIn | `linkedin-openid` | 2.0 |
| Google | `google` | 2.0 |
| GitHub | `github` | 2.0 |
| GitLab | `gitlab` | 2.0 |
| Bitbucket | `bitbucket` | 2.0 |
| Slack | `slack` / `slack-openid` | 2.0 |

### Community Providers (100+)
Available via [Socialite Providers](https://socialiteproviders.com/):
Apple, Discord, Twitch, Spotify, Microsoft, Azure, Auth0, Okta, Keycloak, Amazon, Stripe, etc.

---

## Installation

```bash
composer require laravel/socialite
```

### Windows / Laravel Herd
```bash
"$HOME/.config/herd/bin/composer.bat" require laravel/socialite
```

---

## Configuration

### 1. Create OAuth App on Provider

Each provider requires creating a "developer application" in their dashboard to get `client_id` and `client_secret`. Common URLs:

| Provider | Developer Console |
|----------|------------------|
| GitHub | https://github.com/settings/developers |
| Google | https://console.cloud.google.com/apis/credentials |
| Facebook | https://developers.facebook.com/apps |
| X (Twitter) | https://developer.x.com/en/portal |
| LinkedIn | https://www.linkedin.com/developers/apps |
| GitLab | https://gitlab.com/-/user_settings/applications |
| Bitbucket | https://bitbucket.org/account/settings/app-authorizations/ |
| Slack | https://api.slack.com/apps |

### 2. Environment Variables (`.env`)

```env
# GitHub
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret
GITHUB_REDIRECT_URL=http://localhost:8000/auth/github/callback

# Google
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_REDIRECT_URL=http://localhost:8000/auth/google/callback

# Facebook
FACEBOOK_CLIENT_ID=your-facebook-app-id
FACEBOOK_CLIENT_SECRET=your-facebook-app-secret
FACEBOOK_REDIRECT_URL=http://localhost:8000/auth/facebook/callback

# X (Twitter)
X_CLIENT_ID=your-x-client-id
X_CLIENT_SECRET=your-x-client-secret
X_REDIRECT_URL=http://localhost:8000/auth/x/callback

# LinkedIn
LINKEDIN_CLIENT_ID=your-linkedin-client-id
LINKEDIN_CLIENT_SECRET=your-linkedin-client-secret
LINKEDIN_REDIRECT_URL=http://localhost:8000/auth/linkedin/callback

# Slack
SLACK_CLIENT_ID=your-slack-client-id
SLACK_CLIENT_SECRET=your-slack-client-secret
SLACK_REDIRECT_URL=http://localhost:8000/auth/slack/callback
```

### 3. Services Config (`config/services.php`)

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => env('GITHUB_REDIRECT_URL'),
],

'google' => [
    'client_id' => env('GOOGLE_CLIENT_ID'),
    'client_secret' => env('GOOGLE_CLIENT_SECRET'),
    'redirect' => env('GOOGLE_REDIRECT_URL'),
],

'facebook' => [
    'client_id' => env('FACEBOOK_CLIENT_ID'),
    'client_secret' => env('FACEBOOK_CLIENT_SECRET'),
    'redirect' => env('FACEBOOK_REDIRECT_URL'),
],

'x' => [
    'client_id' => env('X_CLIENT_ID'),
    'client_secret' => env('X_CLIENT_SECRET'),
    'redirect' => env('X_REDIRECT_URL'),
],

'linkedin-openid' => [
    'client_id' => env('LINKEDIN_CLIENT_ID'),
    'client_secret' => env('LINKEDIN_CLIENT_SECRET'),
    'redirect' => env('LINKEDIN_REDIRECT_URL'),
],

'slack' => [
    'client_id' => env('SLACK_CLIENT_ID'),
    'client_secret' => env('SLACK_CLIENT_SECRET'),
    'redirect' => env('SLACK_REDIRECT_URL'),
],
```

> **Note:** If `redirect` contains a relative path, it is automatically resolved to a fully qualified URL.

---

## Database Migration

Add social provider columns to the users table:

```bash
php artisan make:migration add_social_columns_to_users_table
```

### Single Provider
```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('github_id')->nullable()->unique();
        $table->string('github_token')->nullable();
        $table->string('github_refresh_token')->nullable();
        $table->string('password')->nullable()->change(); // Allow null for social-only users
    });
}
```

### Multiple Providers
```php
public function up(): void
{
    Schema::table('users', function (Blueprint $table) {
        $table->string('provider')->nullable();       // 'github', 'google', etc.
        $table->string('provider_id')->nullable();
        $table->string('provider_token')->nullable();
        $table->string('provider_refresh_token')->nullable();
        $table->string('avatar')->nullable();
        $table->string('password')->nullable()->change();

        $table->unique(['provider', 'provider_id']);
    });
}
```

### Update User Model (`app/Models/User.php`)

```php
class User extends Authenticatable
{
    protected $fillable = [
        'name',
        'email',
        'password',
        'provider',
        'provider_id',
        'provider_token',
        'provider_refresh_token',
        'avatar',
    ];

    protected $hidden = [
        'password',
        'remember_token',
        'provider_token',
        'provider_refresh_token',
    ];
}
```

---

## Routing

### Basic (Single Provider)

```php
use Laravel\Socialite\Facades\Socialite;

Route::get('/auth/github/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/github/callback', function () {
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate([
        'github_id' => $githubUser->id,
    ], [
        'name' => $githubUser->name,
        'email' => $githubUser->email,
        'github_token' => $githubUser->token,
        'github_refresh_token' => $githubUser->refreshToken,
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

### Dynamic Multi-Provider (Recommended)

```php
use App\Http\Controllers\Auth\SocialiteController;

Route::get('/auth/{provider}/redirect', [SocialiteController::class, 'redirect'])
    ->name('socialite.redirect');
Route::get('/auth/{provider}/callback', [SocialiteController::class, 'callback'])
    ->name('socialite.callback');
```

**Controller** (`app/Http/Controllers/Auth/SocialiteController.php`):

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Facades\Socialite;

class SocialiteController extends Controller
{
    /**
     * Supported OAuth providers.
     */
    protected array $providers = [
        'github',
        'google',
        'facebook',
        'x',
        'linkedin-openid',
        'gitlab',
        'bitbucket',
        'slack',
    ];

    /**
     * Redirect to OAuth provider.
     */
    public function redirect(string $provider): RedirectResponse
    {
        abort_unless(in_array($provider, $this->providers), 404);

        return Socialite::driver($provider)->redirect();
    }

    /**
     * Handle OAuth callback.
     */
    public function callback(string $provider): RedirectResponse
    {
        abort_unless(in_array($provider, $this->providers), 404);

        try {
            $socialUser = Socialite::driver($provider)->user();
        } catch (\Exception $e) {
            return redirect()->route('login')
                ->with('error', 'Authentication failed. Please try again.');
        }

        $user = User::updateOrCreate(
            [
                'provider' => $provider,
                'provider_id' => $socialUser->getId(),
            ],
            [
                'name' => $socialUser->getName() ?? $socialUser->getNickname(),
                'email' => $socialUser->getEmail(),
                'avatar' => $socialUser->getAvatar(),
                'provider_token' => $socialUser->token,
                'provider_refresh_token' => $socialUser->refreshToken ?? null,
            ],
        );

        Auth::login($user, remember: true);

        return redirect()->intended('/dashboard');
    }
}
```

### Login Links (Blade)

```blade
<div class="social-login">
    <a href="{{ route('socialite.redirect', 'github') }}" class="btn btn-github">
        Login with GitHub
    </a>
    <a href="{{ route('socialite.redirect', 'google') }}" class="btn btn-google">
        Login with Google
    </a>
    <a href="{{ route('socialite.redirect', 'facebook') }}" class="btn btn-facebook">
        Login with Facebook
    </a>
</div>
```

---

## Access Scopes

### Add Scopes (merge with defaults)
```php
return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();
```

### Replace All Scopes
```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

### Common Provider Scopes

| Provider | Common Scopes |
|----------|--------------|
| GitHub | `read:user`, `user:email`, `public_repo`, `repo` |
| Google | `openid`, `profile`, `email`, `https://www.googleapis.com/auth/calendar.readonly` |
| Facebook | `email`, `public_profile`, `user_friends`, `pages_manage_posts` |
| LinkedIn | `openid`, `profile`, `email`, `w_member_social` |
| Slack | `identity.basic`, `identity.email`, `identity.team`, `identity.avatar` |

---

## Slack Bot Scopes

Slack supports two token types:
- **User tokens** (`xoxp-`) — default behavior
- **Bot tokens** (`xoxb-`) — for sending notifications to workspaces

### Generate Bot Token
```php
// Redirect
return Socialite::driver('slack')
    ->asBotUser()
    ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
    ->redirect();

// Callback — MUST also call asBotUser()
$user = Socialite::driver('slack')->asBotUser()->user();
// Only $user->token is populated (bot token for sending notifications)
```

---

## Optional Parameters

```php
// Google: restrict to specific domain
return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();

// Google: force account selection
return Socialite::driver('google')
    ->with(['prompt' => 'select_account'])
    ->redirect();

// GitHub: allow signup
return Socialite::driver('github')
    ->with(['allow_signup' => 'true'])
    ->redirect();

// Facebook: re-request declined permissions
return Socialite::driver('facebook')
    ->with(['auth_type' => 'rerequest'])
    ->redirect();
```

> **Warning:** Do not pass reserved keywords like `state` or `response_type` via `with()`.

---

## Retrieving User Details

### User Object Properties & Methods

```php
$user = Socialite::driver('github')->user();

// All providers
$user->getId();        // Provider's unique user ID
$user->getNickname();  // Username/handle
$user->getName();      // Full name
$user->getEmail();     // Email address
$user->getAvatar();    // Avatar URL

// OAuth 2.0 providers
$user->token;          // Access token
$user->refreshToken;   // Refresh token (if available)
$user->expiresIn;      // Token expiration (seconds)
$user->approvedScopes; // Scopes granted by user

// OAuth 1.0 providers
$user->token;          // Access token
$user->tokenSecret;    // Token secret
```

### From Existing Token
```php
// If you already have a valid access token
$user = Socialite::driver('github')->userFromToken($token);
```

### Stateless Authentication (APIs)
```php
// Disable session state verification (useful for SPAs / stateless APIs)
return Socialite::driver('google')->stateless()->user();
```

---

## Linking Social Accounts to Existing Users

Allow authenticated users to link additional social accounts:

### Migration
```bash
php artisan make:migration create_social_accounts_table
```

```php
public function up(): void
{
    Schema::create('social_accounts', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->string('provider');
        $table->string('provider_id');
        $table->string('provider_token')->nullable();
        $table->string('provider_refresh_token')->nullable();
        $table->timestamps();

        $table->unique(['provider', 'provider_id']);
    });
}
```

### Model
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class SocialAccount extends Model
{
    protected $fillable = [
        'user_id',
        'provider',
        'provider_id',
        'provider_token',
        'provider_refresh_token',
    ];

    protected $hidden = [
        'provider_token',
        'provider_refresh_token',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

### User Model Relationship
```php
public function socialAccounts(): HasMany
{
    return $this->hasMany(SocialAccount::class);
}
```

### Controller with Linking Support
```php
public function callback(string $provider): RedirectResponse
{
    abort_unless(in_array($provider, $this->providers), 404);

    try {
        $socialUser = Socialite::driver($provider)->user();
    } catch (\Exception $e) {
        return redirect()->route('login')
            ->with('error', 'Authentication failed.');
    }

    // Check if social account already linked
    $socialAccount = SocialAccount::where('provider', $provider)
        ->where('provider_id', $socialUser->getId())
        ->first();

    if ($socialAccount) {
        // Login existing user
        Auth::login($socialAccount->user, remember: true);
        return redirect()->intended('/dashboard');
    }

    if (Auth::check()) {
        // Link to currently authenticated user
        Auth::user()->socialAccounts()->create([
            'provider' => $provider,
            'provider_id' => $socialUser->getId(),
            'provider_token' => $socialUser->token,
            'provider_refresh_token' => $socialUser->refreshToken,
        ]);

        return redirect()->route('profile')
            ->with('success', ucfirst($provider) . ' account linked.');
    }

    // Try to find user by email
    $user = User::where('email', $socialUser->getEmail())->first();

    if (! $user) {
        // Create new user
        $user = User::create([
            'name' => $socialUser->getName() ?? $socialUser->getNickname(),
            'email' => $socialUser->getEmail(),
            'avatar' => $socialUser->getAvatar(),
        ]);
    }

    // Create social account link
    $user->socialAccounts()->create([
        'provider' => $provider,
        'provider_id' => $socialUser->getId(),
        'provider_token' => $socialUser->token,
        'provider_refresh_token' => $socialUser->refreshToken,
    ]);

    Auth::login($user, remember: true);

    return redirect()->intended('/dashboard');
}
```

---

## Community Providers (Socialite Providers)

### Installation
```bash
composer require socialiteproviders/apple
# or any provider from socialiteproviders.com
```

### Register Event Listener

**Laravel 11+** (`bootstrap/app.php` or `AppServiceProvider`):
```php
use SocialiteProviders\Manager\SocialiteWasCalled;
use SocialiteProviders\Apple\AppleExtendSocialite;

public function boot(): void
{
    Event::listen(SocialiteWasCalled::class, AppleExtendSocialite::class . '@handle');
}
```

### Config (`config/services.php`)
```php
'apple' => [
    'client_id' => env('APPLE_CLIENT_ID'),
    'client_secret' => env('APPLE_CLIENT_SECRET'),
    'redirect' => env('APPLE_REDIRECT_URL'),
],
```

### Popular Community Providers
| Provider | Package | Config Key |
|----------|---------|-----------|
| Apple | `socialiteproviders/apple` | `apple` |
| Discord | `socialiteproviders/discord` | `discord` |
| Twitch | `socialiteproviders/twitch` | `twitch` |
| Spotify | `socialiteproviders/spotify` | `spotify` |
| Microsoft | `socialiteproviders/microsoft` | `microsoft` |
| Azure | `socialiteproviders/microsoft-azure` | `microsoft` |
| Auth0 | `socialiteproviders/auth0` | `auth0` |
| Keycloak | `socialiteproviders/keycloak` | `keycloak` |
| Amazon | `socialiteproviders/amazon` | `amazon` |
| Stripe | `socialiteproviders/stripe` | `stripe` |

---

## Testing

### Faking the Redirect
```php
use Laravel\Socialite\Facades\Socialite;

test('user is redirected to github', function () {
    Socialite::fake('github');

    $response = $this->get('/auth/github/redirect');

    $response->assertRedirect();
});
```

### Faking the Callback
```php
use Laravel\Socialite\Facades\Socialite;
use Laravel\Socialite\Two\User;

test('user can login with github', function () {
    Socialite::fake('github', (new User)->map([
        'id' => 'github-123',
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]));

    $response = $this->get('/auth/github/callback');

    $response->assertRedirect('/dashboard');

    $this->assertDatabaseHas('users', [
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]);

    $this->assertAuthenticated();
});
```

### Fake User with Full Properties
```php
$fakeUser = (new User)->map([
    'id' => 'github-123',
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'nickname' => 'johndoe',
    'avatar' => 'https://example.com/avatar.jpg',
])->setToken('fake-token')
  ->setRefreshToken('fake-refresh-token')
  ->setExpiresIn(3600)
  ->setApprovedScopes(['read', 'write']);
```

### Test Multiple Providers
```php
test('user can login with any provider', function (string $provider) {
    Socialite::fake($provider, (new User)->map([
        'id' => $provider . '-123',
        'name' => 'John Doe',
        'email' => 'john@example.com',
    ]));

    $this->get("/auth/{$provider}/callback")
        ->assertRedirect('/dashboard');

    $this->assertAuthenticated();
})->with(['github', 'google', 'facebook']);
```

### Test Failed Authentication
```php
test('failed oauth shows error', function () {
    // Don't fake — the callback will throw an exception
    $response = $this->get('/auth/github/callback');

    $response->assertRedirect('/login');
    $response->assertSessionHas('error');
});
```

---

## Filament Integration

For Filament admin panels, use Socialite in a custom login page:

```php
// In your Filament PanelProvider
use App\Filament\Pages\Auth\Login;

->login(Login::class)
```

Or use the community package `dutchcodingcompany/filament-socialite` for built-in Filament support.

---

## Security Best Practices

1. **Always validate email**: Some providers don't verify emails — check `email_verified` if available
2. **Use HTTPS** in production for redirect URLs
3. **Store tokens encrypted**: Use Laravel's `encrypted` cast for token columns
4. **Handle missing email**: Some providers (Apple, GitHub) may not return email — have a fallback
5. **Validate provider**: Always check the provider parameter against an allowlist
6. **Use CSRF protection**: Socialite handles `state` parameter automatically (don't use `stateless` unless needed)
7. **Rate limit**: Add rate limiting to redirect routes to prevent abuse

```php
// Encrypted tokens in User model
protected function casts(): array
{
    return [
        'provider_token' => 'encrypted',
        'provider_refresh_token' => 'encrypted',
    ];
}
```

---

## Workflow

When asked to configure Socialite:

1. **Install**: `composer require laravel/socialite`
2. **Create OAuth app** on the provider's developer console
3. **Add env vars**: client_id, client_secret, redirect URL
4. **Configure** `config/services.php`
5. **Create migration** for social columns / social_accounts table
6. **Create controller** with redirect + callback (single or multi-provider)
7. **Add routes** for redirect and callback
8. **Add login buttons** to the login view
9. **Write tests** using `Socialite::fake()`
10. **Test manually** — verify login flow end-to-end
