# 🔀 Git Workflow Guide

## Visão Geral

Guia completo do workflow Git usado no LOCKDOWN. Leia estes documentos para entender como trabalhar colaborativamente com a equipe.

---

## 📖 Ordem de Leitura Recomendada

1. **[000-FLOW.md](./000-FLOW.md)** ⭐ **COMECE AQUI!**
   - Git flow geral
   - Visão geral do processo
   - Branch strategy overview
   - Workflow completo

2. **[001-BRANCH.md](./001-BRANCH.md)**
   - Estratégia de branches
   - Nomenclatura de branches
   - Quando criar branches
   - Branch protection rules

3. **[002-COMMIT.md](./002-COMMIT.md)**
   - Padrão de commit messages
   - Conventional Commits
   - Boas práticas de commit
   - Quando fazer commit

4. **[003-PUSH.md](./003-PUSH.md)**
   - Guidelines de push
   - Pre-push validations
   - Force push (quando evitar)
   - Conflict resolution

5. **[004-PULL.md](./004-PULL.md)**
   - Pull requests
   - PR template
   - Code review process
   - Merge strategies

---

## 🎯 Quick Reference

### Criar feature branch

```bash
git checkout -b feature/nova-funcionalidade
```

### Commit com mensagem padronizada

```bash
git add .
git commit -m "feat: adiciona nova funcionalidade"
```

### Push para remote

```bash
git push origin feature/nova-funcionalidade
```

### Criar Pull Request

```bash
# No GitHub UI ou via gh CLI
gh pr create --title "feat: nova funcionalidade" --body "Descrição"
```

---

## 💡 Regras de Ouro

1. **Nunca** commit direto em `main`
2. **Sempre** criar branch para mudanças
3. **Sempre** usar Conventional Commits
4. **Sempre** fazer code review
5. **Nunca** force push em branches compartilhadas

---

**Colaboração eficiente através do Git!** 🔀
