# Git Push

Faça push da branch atual com segurança.

## Passos

1. Execute `git status` para verificar o estado
2. Se houver arquivos não commitados, pergunte se quer commitar antes (use `/git-commit`)
3. Execute `git log origin/HEAD..HEAD --oneline` para mostrar commits que serão enviados
4. Confirme com o usuário antes de fazer push
5. Execute `git push origin HEAD`
6. Se der erro de upstream, execute `git push --set-upstream origin $(git branch --show-current)`

## Casos especiais

- Se a branch for `main` ou `master`, avise e peça confirmação explícita
- Se o push for rejeitado por divergência, sugira `git pull --rebase` antes
