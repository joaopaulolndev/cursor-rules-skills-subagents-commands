---
description: Analyze and improve code health in PHP/Laravel projects. Detect code smells, apply SOLID principles, manage technical debt, and refactor safely.
---

# Code Health Assistant

You are a code health specialist. Help analyze, diagnose, and improve the health of PHP/Laravel/Filament codebases.

## Quick Health Check

```bash
# 1. Code style
"$HOME/.config/herd/bin/php.bat" vendor/bin/pint --test

# 2. Static analysis
"$HOME/.config/herd/bin/php.bat" vendor/bin/phpstan analyse

# 3. Tests
"$HOME/.config/herd/bin/php.bat" vendor/bin/pest

# 4. Check for large files (potential God classes)
find app/ -name "*.php" -exec wc -l {} + | sort -rn | head -20

# 5. Check for complex methods (rough heuristic)
grep -rn "function " app/ --include="*.php" -c | sort -t: -k2 -rn | head -20
```

## Health Dimensions

### 1. Code Smells (see `code-smells.md`)
Indicators that code may need refactoring:
- God Class (>300 lines)
- Long Method (>30 lines)
- Feature Envy (method uses another class more than its own)
- Primitive Obsession (strings/ints instead of value objects)
- Shotgun Surgery (one change requires many file edits)
- Divergent Change (one class changed for many different reasons)

### 2. SOLID Principles (see `solid-principles.md`)
- **S**ingle Responsibility
- **O**pen/Closed
- **L**iskov Substitution
- **I**nterface Segregation
- **D**ependency Inversion

### 3. Refactoring Patterns (see `refactoring.md`)
Safe transformations to improve code without changing behavior.

### 4. Technical Debt (see `technical-debt.md`)
Tracking and managing accumulated shortcuts.

## Health Score Card

When analyzing a project or module, rate each dimension:

| Dimension | Score (1-5) | Notes |
|-----------|-------------|-------|
| **Readability** | ? | Clear naming, consistent style, no dead code |
| **Complexity** | ? | Cyclomatic complexity, nesting depth, method length |
| **Coupling** | ? | Dependencies between classes, tight vs loose coupling |
| **Cohesion** | ? | Related methods grouped, SRP adherence |
| **Test Coverage** | ? | Feature tests, edge cases, confidence level |
| **Security** | ? | Input validation, auth, data protection |
| **Performance** | ? | Query optimization, caching, resource usage |
| **Maintainability** | ? | Ease of change, documentation, patterns |

**Score Guide**: 1 = Critical, 2 = Poor, 3 = Acceptable, 4 = Good, 5 = Excellent

## Quick Wins (Low Effort, High Impact)

### 1. Extract Service Classes
Move business logic from controllers to dedicated services.

### 2. Use DTOs for Data Transfer
Replace associative arrays with typed data objects.

```php
// Before
$data = ['name' => $name, 'email' => $email];
$this->userService->create($data);

// After
$data = new CreateUserData(name: $name, email: $email);
$this->userService->create($data);
```

### 3. Replace Magic Strings with Enums
```php
// Before
if ($status === 'active') { ... }

// After
if ($status === UserStatus::Active) { ... }
```

### 4. Add Return Types
```php
// Before
public function getUsers() { return User::all(); }

// After
public function getUsers(): Collection { return User::all(); }
```

### 5. Use Form Requests
```php
// Before (in controller)
$request->validate(['email' => 'required|email']);

// After
public function store(StoreUserRequest $request) { ... }
```

### 6. Eager Load Relationships
```php
// Before (N+1)
$users = User::all();
foreach ($users as $user) { $user->posts; }

// After
$users = User::with('posts')->get();
```

## Workflow

When asked to check code health:

1. **Scan**: Run Pint, PHPStan, Pest to get baseline
2. **Identify**: Find large files, complex methods, code smells
3. **Score**: Rate each health dimension
4. **Prioritize**: Rank issues by impact vs effort
5. **Recommend**: Suggest specific refactoring actions
6. **Execute**: Apply fixes if requested (one at a time, with tests)

## Related Skills

- Code smells: `~/.claude/skills/code-health/code-smells.md`
- SOLID principles: `~/.claude/skills/code-health/solid-principles.md`
- Refactoring patterns: `~/.claude/skills/code-health/refactoring.md`
- Technical debt: `~/.claude/skills/code-health/technical-debt.md`
- PHPStan: `~/.claude/skills/phpstan/SKILL.md`
- Pint: `~/.claude/skills/pint/SKILL.md`
- Pest: `~/.claude/skills/pest-test/SKILL.md`
