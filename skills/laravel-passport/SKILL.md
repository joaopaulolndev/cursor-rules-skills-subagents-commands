---
description: laravel-passport
---

--..."
PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----..."
```

## Configuration

### Token Lifetimes

```php
// AppServiceProvider::boot()
Passport::tokensExpireIn(CarbonInterval::days(15));
Passport::refreshTokensExpireIn(CarbonInterval::days(30));
Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
```

> `expires_at` columns are read-only/display. To invalidate, revoke the token.

### Custom Models

```php
// AppServiceProvider::boot()
Passport::useTokenModel(Token::class);
Passport::useRefreshTokenModel(RefreshToken::class);
Passport::useAuthCodeModel(AuthCode::class);
Passport::useClientModel(Client::class);
Passport::useDeviceCodeModel(DeviceCode::class);
```

### Custom Routes

```php
// AppServiceProvider::register()
Passport::ignoreRoutes();

// Then copy routes from Passport's routes/web.php to your routes/web.php
```

## Authorization Code Grant

### Authorization View

```php
// AppServiceProvider::boot()
Passport::authorizationView('auth.oauth.authorize');

// Or with Inertia
Passport::authorizationView(fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
    'request' => $parameters['request'],
    'authToken' => $parameters['authToken'],
    'client' => $parameters['client'],
    'user' => $parameters['user'],
    'scopes' => $parameters['scopes'],
]));
```

View must POST to `passport.authorizations.approve` and DELETE to `passport.authorizations.deny` with `state`, `client_id`, `auth_token`.

### Managing Clients

```bash
php artisan passport:client
```

Programmatic (third-party):

```php
use Laravel\Passport\ClientRepository;

$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// $client->id, $client->plainSecret
$clients = $user->oauthApps()->get();
```

### Requesting Tokens

#### 1. Redirect for Authorization

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

#### 2. Exchange Code for Token

```php
Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');
    throw_unless(strlen($state) > 0 && $state === $request->state, InvalidArgumentException::class);

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json(); // access_token, refresh_token, expires_in
});
```

### Skip Authorization (First-Party)

```php
class Client extends BaseClient
{
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

### Managing Tokens

```php
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();
```

### Refreshing Tokens

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // confidential clients only
    'scope' => 'user:read orders:create',
]);
```

### Revoking Tokens

```php
$token->revoke();
$token->refreshToken?->revoke();

// Revoke all user tokens
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

### Purging Tokens

```bash
php artisan passport:purge                # revoked + expired
php artisan passport:purge --hours=6      # expired > 6h
php artisan passport:purge --revoked      # only revoked
php artisan passport:purge --expired      # only expired
```

```php
// Scheduled
Schedule::command('passport:purge')->hourly();
```

## Authorization Code Grant With PKCE

For SPAs and mobile apps where client secret can't be stored securely.

```bash
php artisan passport:client --public
```

### Redirect with PKCE

```php
$request->session()->put('code_verifier', $codeVerifier = Str::random(128));

$codeChallenge = strtr(rtrim(
    base64_encode(hash('sha256', $codeVerifier, true))
, '='), '+/', '-_');

$query = http_build_query([
    'client_id' => 'your-client-id',
    'redirect_uri' => 'https://third-party-app.com/callback',
    'response_type' => 'code',
    'scope' => 'user:read orders:create',
    'state' => $state,
    'code_challenge' => $codeChallenge,
    'code_challenge_method' => 'S256',
]);
```

### Exchange with Code Verifier

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'authorization_code',
    'client_id' => 'your-client-id',
    'redirect_uri' => 'https://third-party-app.com/callback',
    'code_verifier' => $codeVerifier,
    'code' => $request->code,
]);
```

## Device Authorization Grant

For browserless/limited-input devices (TVs, game consoles).

```bash
php artisan passport:client --device
```

### Views

```php
Passport::deviceUserCodeView('auth.oauth.device.user-code');
Passport::deviceAuthorizationView('auth.oauth.device.authorize');
```

### Request Device Code

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);
// Returns: device_code, user_code, verification_uri, interval, expires_in
```

### Poll for Token

```php
$interval = 5;
do {
    Sleep::for($interval)->seconds();
    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // confidential only
        'device_code' => 'the-device-code',
    ]);
    if ($response->json('error') === 'slow_down') $interval += 5;
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));
```

## Password Grant

> **Not recommended.** Use authorization code grant instead.

```php
// AppServiceProvider::boot()
Passport::enablePasswordGrant();
```

```bash
php artisan passport:client --password
```

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => 'user:read orders:create', // or '*' for all scopes
]);
```

### Customize Username/Password

```php
// On User model
public function findForPassport(string $username, Client $client): User
{
    return $this->where('username', $username)->first();
}

public function validateForPassportPasswordGrant(string $password): bool
{
    return Hash::check($password, $this->password);
}
```

## Implicit Grant

> **Not recommended.**

```php
Passport::enableImplicitGrant();
```

```bash
php artisan passport:client --implicit
```

Redirect with `response_type=token`.

## Client Credentials Grant

Machine-to-machine authentication.

```bash
php artisan passport:client --client
```

```php
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // ...
})->middleware(EnsureClientIsResourceOwner::class);

// With scopes
Route::get('/orders', function () {
    // ...
})->middleware(EnsureClientIsResourceOwner::using('servers:read', 'servers:create'));
```

```php
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read servers:create',
]);
```

## Personal Access Tokens

```bash
php artisan passport:client --personal
```

```php
$token = $user->createToken('My Token')->accessToken;
$token = $user->createToken('My Token', ['user:read', 'orders:create'])->accessToken;
$token = $user->createToken('My Token', ['*'])->accessToken;

// List valid PATs
$tokens = $user->tokens()
    ->with('client')
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get()
    ->filter(fn (Token $token) => $token->client->hasGrantType('personal_access'));
```

## Protecting Routes

### Via Middleware

```php
Route::get('/user', function () {
    // ...
})->middleware('auth:api');
```

### Multiple Guards

```php
// config/auth.php
'guards' => [
    'api' => ['driver' => 'passport', 'provider' => 'users'],
    'api-customers' => ['driver' => 'passport', 'provider' => 'customers'],
],

Route::get('/customer', fn () => ...)->middleware('auth:api-customers');
```

### Passing Access Token

```php
$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');
```

## Token Scopes

### Defining Scopes

```php
// AppServiceProvider::boot()
Passport::tokensCan([
    'user:read' => 'Retrieve the user info',
    'orders:create' => 'Place orders',
    'orders:read:status' => 'Check order status',
]);

Passport::defaultScopes(['user:read', 'orders:create']);
```

### Checking Scopes (Middleware)

```php
use Laravel\Passport\Http\Middleware\CheckToken;
use Laravel\Passport\Http\Middleware\CheckTokenForAnyScope;

// All scopes required
Route::get('/orders', fn () => ...)
    ->middleware(['auth:api', CheckToken::using('orders:read', 'orders:create')]);

// Any scope matches
Route::get('/orders', fn () => ...)
    ->middleware(['auth:api', CheckTokenForAnyScope::using('orders:read', 'orders:create')]);
```

### Checking on Token Instance

```php
if ($request->user()->tokenCan('orders:create')) {
    // ...
}
```

### Scope Helpers

```php
Passport::scopeIds();                              // array of IDs
Passport::scopes();                                // array of Scope instances
Passport::scopesFor(['user:read', 'orders:create']); // specific scopes
Passport::hasScope('orders:create');                // bool
```

## SPA Authentication

Cookie-based API consumption from your own JavaScript app.

```php
// bootstrap/app.php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        CreateFreshApiToken::class,  // must be last
    ]);
})
```

Attaches `laravel_token` cookie (encrypted JWT, lifetime = `session.lifetime`).

```js
// No Bearer token needed — cookie is sent automatically
axios.get('/api/user').then(response => console.log(response.data));
```

Customize cookie name:

```php
Passport::cookie('custom_name');
```

> Requires CSRF token (`X-XSRF-TOKEN` header) on requests.

## Events

| Event |
|-------|
| `Laravel\Passport\Events\AccessTokenCreated` |
| `Laravel\Passport\Events\AccessTokenRevoked` |
| `Laravel\Passport\Events\RefreshTokenCreated` |

## Testing

```php
use Laravel\Passport\Passport;

// Act as user with scopes
Passport::actingAs(
    User::factory()->create(),
    ['orders:create']
);
$response = $this->post('/api/orders');
$response->assertStatus(201);

// Act as client with scopes
Passport::actingAsClient(
    Client::factory()->create(),
    ['servers:read']
);
$response = $this->get('/api/servers');
$response->assertStatus(200);
```

## Client Types Summary

| Command | Grant Type | Use Case |
|---------|-----------|----------|
| `passport:client` | Authorization Code | Third-party apps |
| `passport:client --public` | Auth Code + PKCE | SPAs, mobile (no secret) |
| `passport:client --device` | Device Authorization | TVs, IoT, game consoles |
| `passport:client --password` | Password (deprecated) | First-party apps |
| `passport:client --implicit` | Implicit (deprecated) | JS/mobile apps |
| `passport:client --client` | Client Credentials | Machine-to-machine |
| `passport:client --personal` | Personal Access | User-issued API tokens |

## Passport Routes (auto-registered)

| Route | Method | Purpose |
|-------|--------|---------|
| `/oauth/authorize` | GET | Authorization screen |
| `/oauth/token` | POST | Exchange code/credentials for token |
| `/oauth/device/code` | POST | Request device code |
