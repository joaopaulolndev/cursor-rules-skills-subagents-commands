# Laravel Tinker

Prepare e execute código no Tinker para inspecionar dados ou testar lógica.

## Passos

1. Entenda o que o usuário quer inspecionar ou testar
2. Escreva o código PHP adequado
3. Exiba o código antes de executar — confirme se necessário
4. Execute com `php artisan tinker --execute="código"`

## Exemplos de uso

**Inspecionar dados:**
```php
// Buscar usuário
User::find(1)->toArray()

// Contar registros com condição
Post::where('status', 'published')->count()

// Ver relacionamentos
User::with('posts')->first()->posts->pluck('title')
```

**Testar lógica:**
```php
// Testar service/action
app(App\Actions\CreatePostAction::class)->execute(['title' => 'Test', 'body' => 'Body'], User::first())

// Verificar config
config('services.stripe.key')

// Testar helper
Str::slug('Meu Título Aqui')
```

**Gerar dados:**
```php
// Criar factory
User::factory()->create(['email' => 'test@test.com'])

// Criar múltiplos
Post::factory()->count(10)->published()->create()
```
