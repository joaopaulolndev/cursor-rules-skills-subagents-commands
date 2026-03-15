# Filament Make Resource

Gere um Resource Filament completo para um model existente.

## Passos

1. Pergunte qual model precisa de um Resource (se não informado)
2. Verifique se o model existe em `app/Models/`
3. Execute o comando de geração:

```bash
# Resource completo com pages geradas automaticamente
php artisan make:filament-resource NomeModel --generate

# Com suporte a soft deletes
php artisan make:filament-resource NomeModel --generate --soft-deletes
```

4. Após gerar, faça as seguintes melhorias automáticas no Resource:

**No `form()`:**
- Adicione `->label()` em português em todos os campos
- Use `Section::make()` para agrupar campos relacionados
- Adicione `->required()` nos campos obrigatórios conforme migration

**No `table()`:**
- Adicione `->searchable()` em colunas de texto principais
- Adicione `->sortable()` em colunas de data e status
- Adicione `->badge()->color()` em colunas de status
- Adicione `TrashedFilter::make()` se usar SoftDeletes

5. Mostre os arquivos criados e sugira o próximo passo
