# Git Flow

Guia do fluxo de trabalho Git utilizado no projeto LOCKDOWN.

---

## Visão Geral

Este projeto utiliza uma versão simplificada do Git Flow, otimizada para desenvolvimento ágil.

```
main (produção)
  │
  └── develop (staging/integração)
        │
        ├── feature/* (novas funcionalidades)
        ├── bugfix/* (correções de bugs)
        ├── hotfix/* (correções urgentes em produção)
        └── release/* (preparação de releases)
```

---

## Branches Principais

### `main`
- Branch de **produção**
- Sempre estável e deployável
- Recebe merges apenas de `release/*` ou `hotfix/*`
- Tags de versão são criadas aqui

### `develop`
- Branch de **integração/staging**
- Base para todas as features
- Recebe merges de `feature/*` e `bugfix/*`
- Deployada em ambiente de staging

---

## Branches de Suporte

### `feature/*`
**Propósito:** Desenvolvimento de novas funcionalidades

```bash
# Criar feature
git checkout develop
git pull origin develop
git checkout -b feature/nome-da-feature

# Desenvolver e finalizar
git checkout develop
git merge feature/nome-da-feature
git push origin develop
```

**Nomenclatura:**
- `feature/user-authentication`
- `feature/ban-synchronization`
- `feature/dashboard-charts`

### `bugfix/*`
**Propósito:** Correção de bugs em develop

```bash
git checkout develop
git checkout -b bugfix/descricao-do-bug
# Corrigir e merge em develop
```

### `hotfix/*`
**Propósito:** Correções urgentes em produção

```bash
# Criar hotfix (a partir de main)
git checkout main
git checkout -b hotfix/descricao-urgente
# Corrigir e merge em main E develop
```

### `release/*`
**Propósito:** Preparação de uma nova release

```bash
git checkout develop
git checkout -b release/1.0.0
# Preparar release e merge em main e develop
```

---

## Regras

### Nunca Faça
- Push direto para `main`
- Push direto para `develop`
- Rebase de branches públicas
- Force push em branches compartilhadas

### Sempre Faça
- Pull antes de criar branches
- Commits pequenos e atômicos
- Mensagens de commit descritivas
- Code review antes de merge
- Testes antes de merge

---

## Ambiente por Branch

| Branch | Ambiente | URL |
|--------|----------|-----|
| `main` | Produção | https://lockdown.app |
| `develop` | Staging | https://staging.lockdown.app |
| `feature/*` | Local | http://localhost:3000 |

---

**Mantenha o fluxo organizado!**
