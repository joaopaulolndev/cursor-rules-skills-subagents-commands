---
description: Write and run tests with Pest PHP testing framework. Includes Laravel, Livewire, and Filament testing patterns.
---

# Pest Testing Assistant

You are a Pest testing specialist. Help write and run tests for PHP/Laravel projects.

## Quick Commands

```bash
# Run all tests
vendor/bin/pest

# Run specific file
vendor/bin/pest tests/Feature/UserTest.php

# Run with filter
vendor/bin/pest --filter="can create user"

# Run specific group
vendor/bin/pest --group=integration

# Parallel execution
vendor/bin/pest --parallel

# With coverage
vendor/bin/pest --coverage --min=80

# Stop on first failure
vendor/bin/pest --stop-on-failure
```

## Basic Test Syntax

```php
<?php

// Simple test
it('can do something', function () {
    expect(true)->toBeTrue();
});

// Alternative syntax
test('the app works', function () {
    $response = $this->get('/');
    $response->assertStatus(200);
});
```

## Common Expectations

```php
// Types
expect($value)->toBeString();
expect($value)->toBeInt();
expect($value)->toBeArray();
expect($value)->toBeNull();
expect($value)->toBeInstanceOf(User::class);

// Comparisons
expect($value)->toBe('exact');      // ===
expect($value)->toEqual('loose');   // ==
expect($value)->toBeTrue();
expect($value)->toBeFalse();
expect($value)->toBeEmpty();
expect($value)->toBeGreaterThan(5);

// Strings
expect($string)->toContain('needle');
expect($string)->toStartWith('Hello');
expect($string)->toMatch('/regex/');

// Arrays
expect($array)->toHaveKey('key');
expect($array)->toHaveCount(3);
expect($array)->toContain('value');

// Chaining
expect('hello')
    ->toBeString()
    ->not->toBeEmpty()
    ->toHaveLength(5);

// Multiple values
expect('hello')
    ->toBeString()
    ->and('world')
    ->toBeString();
```

## Laravel HTTP Tests

```php
it('shows welcome page', function () {
    $this->get('/')
        ->assertStatus(200)
        ->assertSee('Welcome');
});

it('requires authentication', function () {
    $this->get('/dashboard')
        ->assertRedirect('/login');
});

it('can create post', function () {
    $user = User::factory()->create();

    $this->actingAs($user)
        ->post('/posts', [
            'title' => 'My Post',
            'content' => 'Content here',
        ])
        ->assertStatus(201)
        ->assertJson(['title' => 'My Post']);
});
```

## Database Tests

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

it('creates user in database', function () {
    User::create([
        'name' => 'John',
        'email' => 'john@example.com',
        'password' => bcrypt('password'),
    ]);

    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com',
    ]);
});

it('deletes user', function () {
    $user = User::factory()->create();
    $user->delete();

    $this->assertDatabaseMissing('users', ['id' => $user->id]);
});
```

## Filament Tests

```php
use function Pest\Livewire\livewire;
use App\Filament\Resources\UserResource\Pages\ListUsers;
use App\Filament\Resources\UserResource\Pages\CreateUser;

it('can render list page', function () {
    $this->get(UserResource::getUrl('index'))
        ->assertSuccessful();
});

it('can list users', function () {
    $users = User::factory()->count(10)->create();

    livewire(ListUsers::class)
        ->assertCanSeeTableRecords($users);
});

it('can create user', function () {
    livewire(CreateUser::class)
        ->fillForm([
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ])
        ->call('create')
        ->assertHasNoFormErrors();

    $this->assertDatabaseHas('users', [
        'email' => 'john@example.com',
    ]);
});

it('validates required fields', function () {
    livewire(CreateUser::class)
        ->fillForm(['name' => ''])
        ->call('create')
        ->assertHasFormErrors(['name' => 'required']);
});
```

## Datasets (Data Providers)

```php
it('validates email formats', function (string $email, bool $valid) {
    $result = filter_var($email, FILTER_VALIDATE_EMAIL);
    expect((bool) $result)->toBe($valid);
})->with([
    ['valid@email.com', true],
    ['invalid-email', false],
    ['another@valid.org', true],
]);

// Named dataset
dataset('users', [
    'admin' => ['admin@test.com', 'admin'],
    'user' => ['user@test.com', 'user'],
]);

it('creates user with role', function (string $email, string $role) {
    // test logic
})->with('users');
```

## Hooks

```php
beforeEach(function () {
    $this->user = User::factory()->create();
});

afterEach(function () {
    // cleanup
});

it('can access user', function () {
    expect($this->user)->toBeInstanceOf(User::class);
});
```

## Test Modifiers

```php
// Skip test
it('is skipped')->skip();
it('is skipped with reason')->skip('Not ready');

// Mark as todo
it('needs implementation')->todo();

// Run only this test
it('runs only this')->only();

// Group tests
it('is slow')->group('slow', 'integration');

// Expect exception
it('throws exception', function () {
    throw new Exception('Error');
})->throws(Exception::class, 'Error');
```

## Setup for Packages

**tests/Pest.php:**
```php
<?php

use Vendor\Package\Tests\TestCase;

uses(TestCase::class)->in('Feature', 'Unit');

function createUser(array $attributes = []): User
{
    return User::factory()->create($attributes);
}
```

**tests/TestCase.php:**
```php
<?php

namespace Vendor\Package\Tests;

use Orchestra\Testbench\TestCase as Orchestra;
use Vendor\Package\PackageServiceProvider;

abstract class TestCase extends Orchestra
{
    protected function getPackageProviders($app): array
    {
        return [PackageServiceProvider::class];
    }

    protected function getEnvironmentSetUp($app): void
    {
        config()->set('database.default', 'testing');
        config()->set('database.connections.testing', [
            'driver' => 'sqlite',
            'database' => ':memory:',
        ]);
    }
}
```

## Reference Files

- `~/.claude/skills/laravel/pest-testing.md`
