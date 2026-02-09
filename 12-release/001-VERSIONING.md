# Versionamento

Guia de versionamento do projeto LOCKDOWN.

---

## Versionamento Semântico

Este projeto segue o [Versionamento Semântico 2.0.0](https://semver.org/lang/pt-BR/).

### Formato

```
MAJOR.MINOR.PATCH[-PRERELEASE]

Exemplos:
- 1.0.0
- 1.2.3
- 2.0.0-alpha.1
- 1.5.0-beta.2
```

---

## Componentes da Versão

### MAJOR (X.0.0)
Mudanças **incompatíveis** com versões anteriores:
- Breaking changes na API REST
- Mudanças no schema do banco de dados
- Remoção de endpoints ou comandos

### MINOR (0.X.0)
**Novas funcionalidades** compatíveis:
- Novos endpoints na API
- Novos comandos do bot
- Novas features no dashboard

### PATCH (0.0.X)
**Correções** compatíveis:
- Bug fixes
- Correções de segurança
- Melhorias de performance

---

## Pré-Releases

```
1.0.0-alpha.1   # Desenvolvimento inicial
1.0.0-beta.1    # Feature-complete, em teste
1.0.0-rc.1      # Release Candidate
```

---

## Versionamento dos Packages

Todos os packages mantêm versões sincronizadas:

| Package | Versão |
|---------|--------|
| `lockdown-monorepo` | 0.0.1 |
| `@lockdown/backend` | 0.0.1 |
| `@lockdown/shared` | 0.0.1 |
| `@lockdown/bot-framework` | 0.0.1 |
| `@lockdown/lockdown-bot` | 0.0.1 |
| `@lockdown/frontend` | 0.0.1 |
| `@lockdown/utility-bot` | 0.0.1 |

---

## Comandos

```bash
npm version patch    # 1.0.0 → 1.0.1
npm version minor    # 1.0.0 → 1.1.0
npm version major    # 1.0.0 → 2.0.0
```

---

## Tags Git

```bash
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0
```

---

**Versione com responsabilidade!**
