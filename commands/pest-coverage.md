# Pest Coverage

Execute os testes com relatório de cobertura e identifique gaps.

## Passos

1. Execute a suite com coverage:
```bash
./vendor/bin/pest --coverage --coverage-html=coverage_reports/html --min=80
```

2. Se falhar por extensão não instalada, tente:
```bash
# Com Xdebug
XDEBUG_MODE=coverage ./vendor/bin/pest --coverage

# Com PCOV (mais rápido)
./vendor/bin/pest --coverage --coverage-clover=coverage.xml
```

3. Analise o relatório e identifique:
   - Classes com cobertura abaixo de 80%
   - Métodos públicos sem nenhum teste
   - Branches não testados (condicionais)

4. Priorize cobertura nesta ordem:
   - `app/Actions/` e `app/Services/` → lógica crítica
   - `app/Http/Controllers/` → endpoints da API
   - `app/Models/` → escopos e accessors customizados

5. Sugira os próximos testes a criar com `/pest-generate`
