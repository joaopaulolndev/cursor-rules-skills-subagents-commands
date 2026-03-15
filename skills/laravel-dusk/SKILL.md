---
description: laravel-dusk
---

# Laravel Dusk

> Browser automation and testing API using ChromeDriver. Provides expressive API for browser tests without JDK or Selenium.
> **Note:** For new projects, [Pest 4](https://pestphp.com/) now includes browser testing with better performance. Dusk remains fully supported.

## Installation

```bash
composer require laravel/dusk --dev
php artisan dusk:install
```

Creates `tests/Browser` directory, example test, and installs ChromeDriver.

> **Never** register Dusk's service provider in production.

Set `APP_URL` in `.env` to match browser-accessible URL.

### ChromeDriver Management

```bash
php artisan dusk:chrome-driver              # latest version
php artisan dusk:chrome-driver 86           # specific version
php artisan dusk:chrome-driver --all        # all supported OSs
php artisan dusk:chrome-driver --detect     # match installed Chrome
```

If issues: `chmod -R 0755 vendor/laravel/dusk/bin/`

### Using Other Browsers

In `tests/DuskTestCase.php`, comment out `startChromeDriver()` and customize `driver()`:

```php
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

## Getting Started

### Generate Tests

```bash
php artisan dusk:make LoginTest
```

Tests go in `tests/Browser/`.

### Database Reset

**Never use `RefreshDatabase`** — it uses transactions not available across HTTP requests.

#### DatabaseMigrations (slower, full migrate each test)

```php
// Pest
pest()->use(DatabaseMigrations::class);

// PHPUnit
use DatabaseMigrations;
```

#### DatabaseTruncation (faster, truncate after first migrate)

```php
// Pest
pest()->use(DatabaseTruncation::class);

// PHPUnit
use DatabaseTruncation;
```

Customization (on test class or `DuskTestCase`):

```php
protected $tablesToTruncate = ['users'];        // only truncate these
protected $exceptTables = ['users'];            // exclude from truncation
protected $connectionsToTruncate = ['mysql'];   // specific connections

protected function beforeTruncatingDatabase(): void {}
protected function afterTruncatingDatabase(): void {}
```

> SQLite in-memory databases cannot be used with Dusk.

### Running Tests

```bash
php artisan dusk                 # all tests
php artisan dusk:fails           # re-run failed tests
php artisan dusk --group=foo     # specific group
```

### Environment Handling

Create `.env.dusk.{environment}` (e.g., `.env.dusk.local`). Dusk swaps `.env` files automatically.

## Browser Basics

### Creating Browsers

```php
test('login', function () {
    $user = User::factory()->create(['email' => 'taylor@laravel.com']);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
    });
});
```

#### Multiple Browsers (e.g., chat testing)

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))->visit('/home')->waitForText('Message');
    $second->loginAs(User::find(2))->visit('/home')
        ->type('message', 'Hey Taylor')->press('Send');
    $first->waitForText('Hey Taylor')->assertSee('Jeffrey Way');
});
```

### Navigation

```php
$browser->visit('/login');
$browser->visitRoute($routeName, $parameters);
$browser->back();
$browser->forward();
$browser->refresh();
```

### Resizing

```php
$browser->resize(1920, 1080);
$browser->maximize();
$browser->fitContent();
$browser->disableFitOnFailure();
$browser->move($x = 100, $y = 100);
```

### Authentication

```php
$browser->loginAs(User::find(1))->visit('/home');
// Session maintained for all tests in file
```

### Cookies

```php
$browser->cookie('name');                    // get encrypted
$browser->cookie('name', 'Taylor');          // set encrypted
$browser->plainCookie('name');               // get plain
$browser->plainCookie('name', 'Taylor');     // set plain
$browser->deleteCookie('name');
```

### Executing JavaScript

```php
$browser->script('document.documentElement.scrollTop = 0');
$browser->script(['stmt1', 'stmt2']);
$output = $browser->script('return window.location.pathname');
```

### Screenshots

```php
$browser->screenshot('filename');                           // tests/Browser/screenshots/
$browser->responsiveScreenshots('filename');                 // multiple breakpoints
$browser->screenshotElement('#selector', 'filename');        // specific element
```

### Console & Source

```php
$browser->storeConsoleLog('filename');    // tests/Browser/console/
$browser->storeSource('filename');        // tests/Browser/source/
```

### Browser Macros

```php
// In DuskServiceProvider::boot()
Browser::macro('scrollToElement', function (string $element) {
    $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");
    return $this;
});

// Usage
$browser->scrollToElement('#credit-card-details');
```

## Interacting With Elements

### Dusk Selectors

```html
<button dusk="login-button">Login</button>
```

```php
$browser->click('@login-button');   // @ prefix for dusk attributes
```

Customize attribute:

```php
// AppServiceProvider::boot()
Dusk::selectorHtmlAttribute('data-dusk');
```

### Text, Values, Attributes

```php
$value = $browser->value('selector');
$browser->value('selector', 'value');
$value = $browser->inputValue('field');
$text = $browser->text('selector');
$attr = $browser->attribute('selector', 'value');
```

### Forms

```php
// Typing
$browser->type('email', 'taylor@laravel.com');
$browser->append('tags', ', bar, baz');
$browser->clear('email');
$browser->typeSlowly('mobile', '+1 (202) 555-5555');       // 100ms default
$browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);  // custom delay
$browser->appendSlowly('tags', ', bar, baz');

// Dropdowns
$browser->select('size', 'Large');       // by value
$browser->select('size');                // random
$browser->select('categories', ['Art', 'Music']);  // multiple

// Checkboxes
$browser->check('terms');
$browser->uncheck('terms');

// Radio
$browser->radio('size', 'large');
```

### Attaching Files

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

Requires `Zip` PHP extension.

### Buttons & Links

```php
$browser->press('Login');                          // by text or CSS/Dusk selector
$browser->pressAndWaitFor('Save');                 // wait for re-enable (5s default)
$browser->pressAndWaitFor('Save', 1);              // 1s timeout
$browser->clickLink($linkText);
$browser->seeLink($linkText);                     // returns bool
```

### Keyboard

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
$browser->keys('.app', ['{command}', 'j']);

// Fluent keyboard
$browser->withKeyboard(function (Keyboard $keyboard) {
    $keyboard->press('c')->pause(1000)->release('c')->type(['c', 'e', 'o']);
});
```

Modifier keys: `{shift}`, `{ctrl}`, `{alt}`, `{command}`, `{meta}` — from `WebDriverKeys` constants.

### Mouse

```php
$browser->click('.selector');
$browser->clickAtXPath('//div[@class = "selector"]');
$browser->clickAtPoint($x, $y);
$browser->doubleClick('.selector');
$browser->rightClick('.selector');
$browser->clickAndHold('.selector');
$browser->releaseMouse();
$browser->controlClick('.selector');
$browser->mouseover('.selector');

// Drag & Drop
$browser->drag('.from', '.to');
$browser->dragLeft('.selector', 10);
$browser->dragRight('.selector', 10);
$browser->dragUp('.selector', 10);
$browser->dragDown('.selector', 10);
$browser->dragOffset('.selector', $x, $y);
```

### JavaScript Dialogs

```php
$browser->waitForDialog($seconds);
$browser->assertDialogOpened('Dialog message');
$browser->typeInDialog('Hello World');
$browser->acceptDialog();
$browser->dismissDialog();
```

### Inline Frames

```php
$browser->withinFrame('#credit-card-details', function ($browser) {
    $browser->type('input[name="cardnumber"]', '4242424242424242');
});
```

### Scoping Selectors

```php
$browser->with('.table', function (Browser $table) {
    $table->assertSee('Hello World')->clickLink('Delete');
});

// Break out of scope
$browser->with('.table', function (Browser $table) {
    $browser->elsewhere('.page-title', function (Browser $title) {
        $title->assertSee('Hello World');
    });
    $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
        $title->assertSee('Hello World');
    });
});
```

### Waiting for Elements

```php
// Pause
$browser->pause(1000);
$browser->pauseIf(App::environment('production'), 1000);
$browser->pauseUnless(App::environment('testing'), 1000);

// Wait for selectors
$browser->waitFor('.selector');                    // 5s default
$browser->waitFor('.selector', 1);                 // 1s timeout
$browser->waitForTextIn('.selector', 'Hello');
$browser->waitUntilMissing('.selector');
$browser->waitUntilEnabled('.selector');
$browser->waitUntilDisabled('.selector');

// Scoped wait
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello')->press('OK');
});

// Wait for text
$browser->waitForText('Hello World');
$browser->waitUntilMissingText('Hello World');

// Wait for links/inputs
$browser->waitForLink('Create');
$browser->waitForInput($field);

// Wait for location
$browser->waitForLocation('/secret');
$browser->waitForLocation('https://example.com/path');
$browser->waitForRoute($routeName, $parameters);

// Wait for reload
$browser->waitForReload(fn (Browser $browser) => $browser->press('Submit'))
    ->assertSee('Success!');
$browser->clickAndWaitForReload('.selector');

// Wait for JS expression
$browser->waitUntil('App.data.servers.length > 0');

// Wait for Vue
$browser->waitUntilVue('user.name', 'Taylor', '@user');
$browser->waitUntilVueIsNot('user.name', null, '@user');

// Wait for events
$browser->waitForEvent('load');
$browser->waitForEvent('load', '.selector');
$browser->waitForEvent('scroll', 'document');
$browser->waitForEvent('resize', 'window', 5);

// Custom wait
$browser->waitUsing(10, 1, function () use ($something) {
    return $something->isReady();
}, "Something wasn't ready in time.");
```

### Scrolling

```php
$browser->scrollIntoView('.selector')->click('.selector');
```

## Assertions Reference

### URL/Path

```php
$browser->assertTitle($title);
$browser->assertTitleContains($title);
$browser->assertUrlIs($url);
$browser->assertSchemeIs($scheme);
$browser->assertHostIs($host);
$browser->assertPortIs($port);
$browser->assertPathIs('/home');
$browser->assertPathIsNot('/home');
$browser->assertPathBeginsWith('/home');
$browser->assertPathEndsWith('/home');
$browser->assertPathContains('/home');
$browser->assertRouteIs($name, $parameters);
$browser->assertQueryStringHas($name);
$browser->assertQueryStringHas($name, $value);
$browser->assertQueryStringMissing($name);
$browser->assertFragmentIs('anchor');
$browser->assertFragmentBeginsWith('anchor');
$browser->assertFragmentIsNot('anchor');
```

### Cookies

```php
$browser->assertHasCookie($name);
$browser->assertHasPlainCookie($name);
$browser->assertCookieMissing($name);
$browser->assertPlainCookieMissing($name);
$browser->assertCookieValue($name, $value);
$browser->assertPlainCookieValue($name, $value);
```

### Content/Visibility

```php
$browser->assertSee($text);
$browser->assertDontSee($text);
$browser->assertSeeIn($selector, $text);
$browser->assertDontSeeIn($selector, $text);
$browser->assertSeeAnythingIn($selector);
$browser->assertSeeNothingIn($selector);
$browser->assertCount($selector, $count);
$browser->assertSourceHas($code);
$browser->assertSourceMissing($code);
$browser->assertSeeLink($linkText);
$browser->assertDontSeeLink($linkText);
$browser->assertVisible($selector);
$browser->assertPresent($selector);
$browser->assertNotPresent($selector);
$browser->assertMissing($selector);
```

### Forms/Inputs

```php
$browser->assertInputValue($field, $value);
$browser->assertInputValueIsNot($field, $value);
$browser->assertChecked($field);
$browser->assertNotChecked($field);
$browser->assertIndeterminate($field);
$browser->assertRadioSelected($field, $value);
$browser->assertRadioNotSelected($field, $value);
$browser->assertSelected($field, $value);
$browser->assertNotSelected($field, $value);
$browser->assertSelectHasOptions($field, $values);
$browser->assertSelectMissingOptions($field, $values);
$browser->assertSelectHasOption($field, $value);
$browser->assertSelectMissingOption($field, $value);
$browser->assertInputPresent($name);
$browser->assertInputMissing($name);
```

### Elements/Attributes

```php
$browser->assertValue($selector, $value);
$browser->assertValueIsNot($selector, $value);
$browser->assertAttribute($selector, $attribute, $value);
$browser->assertAttributeMissing($selector, $attribute);
$browser->assertAttributeContains($selector, $attribute, $value);
$browser->assertAttributeDoesntContain($selector, $attribute, $value);
$browser->assertAriaAttribute($selector, $attribute, $value);
$browser->assertDataAttribute($selector, $attribute, $value);
$browser->assertEnabled($field);
$browser->assertDisabled($field);
$browser->assertButtonEnabled($button);
$browser->assertButtonDisabled($button);
$browser->assertFocused($field);
$browser->assertNotFocused($field);
```

### JavaScript

```php
$browser->assertScript('window.isLoaded');
$browser->assertScript('document.readyState', 'complete');
$browser->assertDialogOpened($message);
```

### Auth

```php
$browser->assertAuthenticated();
$browser->assertGuest();
$browser->assertAuthenticatedAs($user);
```

### Vue

```php
$browser->assertVue('user.name', 'Taylor', '@profile-component');
$browser->assertVueIsNot($property, $value, $componentSelector);
$browser->assertVueContains($property, $value, $componentSelector);
$browser->assertVueDoesntContain($property, $value, $componentSelector);
```

## Pages (Page Objects)

### Generate

```bash
php artisan dusk:page Login
```

### Structure

```php
class Login extends Page
{
    public function url(): string
    {
        return '/login';
    }

    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

    public function elements(): array
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

    // Custom methods
    public function submitLogin(Browser $browser, string $email, string $password): void
    {
        $browser->type('@email', $email)
            ->type('password', $password)
            ->press('Login');
    }
}
```

### Usage

```php
use Tests\Browser\Pages\Login;

$browser->visit(new Login)->submitLogin('taylor@laravel.com', 'password');

// Load page context without navigation
$browser->visit('/dashboard')
    ->clickLink('Create Playlist')
    ->on(new CreatePlaylist)
    ->assertSee('@create');
```

### Global Shorthand Selectors

In `tests/Browser/Pages/Page.php`:

```php
public static function siteElements(): array
{
    return ['@element' => '#selector'];
}
```

## Components

### Generate

```bash
php artisan dusk:component DatePicker
```

### Structure

```php
class DatePicker extends BaseComponent
{
    public function selector(): string { return '.date-picker'; }

    public function assert(Browser $browser): void
    {
        $browser->assertVisible($this->selector());
    }

    public function elements(): array
    {
        return [
            '@date-field' => 'input.datepicker-input',
            '@year-list' => 'div > div.datepicker-years',
        ];
    }

    public function selectDate(Browser $browser, int $year, int $month, int $day): void
    {
        $browser->click('@date-field')
            ->within('@year-list', fn ($b) => $b->click($year))
            ->within('@month-list', fn ($b) => $b->click($month))
            ->within('@day-list', fn ($b) => $b->click($day));
    }
}
```

### Usage

```php
$browser->visit('/')
    ->within(new DatePicker, fn (Browser $b) => $b->selectDate(2019, 1, 30))
    ->assertSee('January');

// Or via component()
$datePicker = $browser->component(new DatePickerComponent);
$datePicker->selectDate(2019, 1, 30);
```

## CI Configuration

### GitHub Actions

```yaml
name: CI
on: [push]
jobs:
  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
    steps:
      - uses: actions/checkout@v5
      - run: cp .env.example .env
      - run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - run: composer install --no-progress --prefer-dist --optimize-autoloader
      - run: php artisan key:generate
      - run: php artisan dusk:chrome-driver --detect
      - run: ./vendor/laravel/dusk/bin/chromedriver-linux --port=9515 &
      - run: php artisan serve --no-reload &
      - run: php artisan dusk
      - if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: console
          path: tests/Browser/console
```

### Key CI Notes

- Serve app on port 8000: `php artisan serve --no-reload &`
- Set `APP_URL=http://127.0.0.1:8000`
- Upload screenshots/console logs on failure for debugging
- Use `dusk:chrome-driver --detect` for CI Chrome version matching
