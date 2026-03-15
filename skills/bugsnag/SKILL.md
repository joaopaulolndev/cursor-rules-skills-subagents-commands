---
description: Configure Bugsnag error monitoring for any project type. Supports PHP/Laravel, JavaScript/React/Vue/Angular/Node.js, Python/Django/Flask, Ruby/Rails, Go, Java/Spring, .NET, Flutter, React Native/Expo, and mobile platforms.
---

# Bugsnag Integration Assistant

You are a Bugsnag specialist. Help configure error monitoring and performance tracking for any project type.

## Quick Start — Detect Project & Install

```bash
# Detect project type
if [ -f "composer.json" ]; then echo "PHP project detected"; fi
if [ -f "package.json" ]; then echo "Node/JS project detected"; fi
if [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then echo "Python project detected"; fi
if [ -f "Gemfile" ]; then echo "Ruby project detected"; fi
if [ -f "go.mod" ]; then echo "Go project detected"; fi
if [ -f "pubspec.yaml" ]; then echo "Flutter/Dart project detected"; fi
if [ -f "pom.xml" ] || [ -f "build.gradle" ]; then echo "Java project detected"; fi
```

## Supported Platforms (50+)

| Category | Platforms |
|----------|-----------|
| **PHP** | Laravel, Lumen, Symfony, WordPress, plain PHP |
| **JavaScript** | Browser, React, Vue, Angular, Next.js, Node.js, Express, Koa, Restify |
| **Python** | Django, Flask, ASGI, Bottle, Celery, Tornado, WSGI, AWS Lambda |
| **Ruby** | Rails, Rack, Sinatra, Sidekiq, Rake |
| **Go** | net/http, Gin, Negroni, Revel |
| **Java** | Spring, plain Java |
| **.NET** | ASP.NET Core, ASP.NET MVC, Web API, WPF |
| **Mobile** | Android, iOS, Flutter, React Native, Expo |
| **Game Engines** | Unity, Unreal Engine, Cocos2d-x |
| **Desktop** | Electron, macOS, Windows (WPF) |
| **Other** | Kotlin Multiplatform, tvOS, watchOS, C/C++ |

## Workflow

When asked to configure Bugsnag:

1. **Detect** project type from files (composer.json, package.json, etc.)
2. **Install** the appropriate SDK package
3. **Configure** initialization with API key
4. **Set up** exception handler / error boundary / middleware
5. **Add** environment variables
6. **Configure** source maps (JS/mobile) if applicable
7. **Test** with a manual error notification
8. **Verify** on dashboard

See platform-specific guides:
- PHP/Laravel: `~/.claude/skills/bugsnag/laravel.md`
- JavaScript/React/Vue/Node: `~/.claude/skills/bugsnag/javascript.md`
- Python/Django/Flask: `~/.claude/skills/bugsnag/python.md`
- Ruby/Rails: `~/.claude/skills/bugsnag/ruby.md`
- Go: `~/.claude/skills/bugsnag/go.md`
- Flutter: `~/.claude/skills/bugsnag/flutter.md`
- React Native/Expo: `~/.claude/skills/bugsnag/react-native.md`
- Java/Spring: `~/.claude/skills/bugsnag/java.md`
- .NET: `~/.claude/skills/bugsnag/dotnet.md`

## Universal Environment Variables

All platforms share these concepts:

```env
BUGSNAG_API_KEY=your-project-api-key
BUGSNAG_RELEASE_STAGE=production          # production, staging, development
BUGSNAG_APP_VERSION=1.0.0                 # Your app version
BUGSNAG_NOTIFY_RELEASE_STAGES=production,staging  # Which stages report errors
```

## Common Configuration Patterns

### Filter Sensitive Data
All SDKs support redacting keys from metadata:
- PHP: `'redacted_keys' => ['password', 'secret', 'token', 'authorization']`
- JS: `redactedKeys: ['password', 'secret', 'token', 'authorization']`
- Python: `bugsnag.configure(params_filters=['password', 'secret', 'token'])`

### User Identification
Attach user info to correlate errors with users:
- PHP: `Bugsnag::registerCallback(fn ($report) => $report->setUser([...]))`
- JS: `Bugsnag.setUser('id', 'email', 'name')`
- Python: `bugsnag.configure(user={'id': '1', 'email': '...', 'name': '...'})`

### Custom Metadata
Add extra context to error reports:
- PHP: `Bugsnag::notifyException($e, function ($report) { $report->addMetadata('order', [...]) })`
- JS: `Bugsnag.notify(err, event => { event.addMetadata('order', {...}) })`
- Python: `bugsnag.notify(e, metadata={'order': {...}})`

### Feature Flags
Track feature rollouts:
- PHP: `Bugsnag::addFeatureFlag('new-checkout', 'variant-a')`
- JS: `Bugsnag.addFeatureFlag('new-checkout', 'variant-a')`

### Breadcrumbs
Leave manual breadcrumbs for debugging:
- PHP: `Bugsnag::leaveBreadcrumb('User clicked checkout')`
- JS: `Bugsnag.leaveBreadcrumb('User clicked checkout')`
- Python: `bugsnag.leave_breadcrumb('User clicked checkout')`

## Release Tracking & Source Maps

### Build Integration (CI/CD)
```bash
# Upload source maps (JavaScript)
npx @bugsnag/source-maps upload-browser \
  --api-key YOUR_API_KEY \
  --app-version 1.0.0 \
  --directory ./dist

# Upload Node.js source maps
npx @bugsnag/source-maps upload-node \
  --api-key YOUR_API_KEY \
  --app-version 1.0.0 \
  --directory ./dist

# Notify of deploy (any platform)
curl https://build.bugsnag.com/ \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "apiKey": "YOUR_API_KEY",
    "appVersion": "1.0.0",
    "releaseStage": "production",
    "sourceControl": {
      "provider": "github",
      "repository": "https://github.com/org/repo",
      "revision": "'$(git rev-parse HEAD)'"
    }
  }'
```

### Laravel Artisan Deploy Tracking
```bash
php artisan bugsnag:deploy \
  --repository "https://github.com/org/repo" \
  --revision "$(git rev-parse HEAD)" \
  --builder "CI"
```

## Testing the Integration

### Manual Error Test
```php
# PHP/Laravel
Bugsnag::notifyException(new \RuntimeException('Test error from Bugsnag setup'));
```

```javascript
// JavaScript
Bugsnag.notify(new Error('Test error from Bugsnag setup'))
```

```python
# Python
bugsnag.notify(Exception('Test error from Bugsnag setup'))
```

```ruby
# Ruby
Bugsnag.notify(RuntimeError.new('Test error from Bugsnag setup'))
```

## Dashboard Setup Tips

1. **Create separate projects** for frontend and backend
2. **Set up release stages** (production, staging, development)
3. **Configure integrations**: Slack, GitHub, Jira, etc.
4. **Set up alerts** for new error types and spikes
5. **Create saved searches** for common error patterns
6. **Enable session tracking** for stability scores
