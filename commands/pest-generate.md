# Pest Generate

Gere testes Pest completos para o arquivo atualmente aberto ou para o arquivo informado.

## Passos

1. Identifique o arquivo alvo (controller, model, service, action)
2. Leia o arquivo para entender os métodos e comportamentos
3. Gere testes cobrindo obrigatoriamente:

| Cenário | Assertion |
|---|---|
| Caminho feliz (sucesso) | `assertOk()`, `assertCreated()`, `assertDatabaseHas()` |
| Não autenticado | `assertUnauthorized()` |
| Sem permissão | `assertForbidden()` |
| Validação falhou | `assertUnprocessable()`, `assertJsonValidationErrors()` |
| Recurso não encontrado | `assertNotFound()` |

4. Use este template base:

```php
<?php

use App\Models\User;
use function Pest\Laravel\{actingAs, getJson, postJson, putJson, deleteJson};

uses(Illuminate\Foundation\Testing\RefreshDatabase::class);

beforeEach(function () {
    $this->user = User::factory()->create();
});

it('caminho feliz', function () {
    // Arrange + Act + Assert
});

it('requer autenticação', function () {
    postJson('/api/v1/rota', [])->assertUnauthorized();
});
```

5. Salve o arquivo em `tests/Feature/[Contexto]/[NomeTest].php`
6. Execute os testes gerados com `/pest-run`
