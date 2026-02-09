# Release Guide

Guia para criação e publicação de releases do LOCKDOWN.

---

## Processo de Release

### 1. Preparação

```bash
# Garantir que develop está atualizado
git checkout develop
git pull origin develop

# Verificar que todos os testes passam
pnpm test
pnpm type-check
pnpm lint

# Criar branch de release
git checkout -b release/X.Y.Z
```

### 2. Atualização de Versão

- Atualizar version em todos os package.json
- Atualizar CHANGELOG.md

### 3. Testes Finais

```bash
pnpm build
pnpm test
pnpm validate
```

### 4. Commit e Merge

```bash
git commit -m "chore(release): prepare release X.Y.Z"
git checkout main
git merge release/X.Y.Z
git tag -a vX.Y.Z -m "Release X.Y.Z"
git push origin main --tags

# Merge de volta em develop
git checkout develop
git merge release/X.Y.Z
git push origin develop
```

---

## Checklist de Release

### Pré-Release
- [ ] Todos os testes passando
- [ ] Code review completo
- [ ] Documentação atualizada
- [ ] CHANGELOG.md atualizado
- [ ] Versões bumped em todos os packages

### Pós-Release
- [ ] Deploy em produção
- [ ] Verificar health checks
- [ ] Criar GitHub Release
- [ ] Notificar stakeholders

---

## Hotfix Release

Para correções urgentes:

```bash
git checkout main
git checkout -b hotfix/X.Y.Z
# Fazer correção
git checkout main
git merge hotfix/X.Y.Z
git tag -a vX.Y.Z -m "Hotfix X.Y.Z"
git push origin main --tags
```

---

**Releases bem planejadas garantem estabilidade!**
