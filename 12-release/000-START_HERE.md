# 🚀 Release Management

## Visão Geral

Documentação do processo de release, versionamento e changelog do LOCKDOWN.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-VERSIONING.md](./001-VERSIONING.md)** ⭐ **COMECE AQUI!**
   - Semantic Versioning (SemVer)
   - Quando incrementar versões
   - Breaking changes
   - Version strategy

2. **[002-CHANGELOG.md](./002-CHANGELOG.md)**
   - Como manter o CHANGELOG
   - Formato de changelog
   - Categorias de mudanças
   - Automação de changelog

3. **[003-RELEASE.md](./003-RELEASE.md)**
   - Processo completo de release
   - Release checklist
   - Deployment steps
   - Rollback procedures

---

## 🎯 Semantic Versioning

```
MAJOR.MINOR.PATCH

1.2.3
│ │ └─ Patch: Bug fixes
│ └─── Minor: New features (backward compatible)
└───── Major: Breaking changes
```

---

## 📝 Release Checklist

Antes de cada release:
- [ ] Todos os testes passando
- [ ] CHANGELOG atualizado
- [ ] Version bumped em package.json
- [ ] Documentação atualizada
- [ ] Migration scripts testados
- [ ] Backup do database feito
- [ ] Rollback plan preparado

---

## 🔄 Release Flow

```
1. Criar release branch
   → git checkout -b release/v1.2.0

2. Bump version
   → pnpm version 1.2.0

3. Atualizar CHANGELOG
   → Adicionar novas features/fixes

4. Merge para main
   → PR: release/v1.2.0 → main

5. Tag release
   → git tag v1.2.0

6. Deploy
   → CI/CD automático ou manual
```

---

## 📊 Release Types

### Major Release (1.0.0 → 2.0.0)
- Breaking changes
- Mudanças significativas na API
- Requer migration guide

### Minor Release (1.0.0 → 1.1.0)
- Novas features
- Backward compatible
- Pode incluir deprecations

### Patch Release (1.0.0 → 1.0.1)
- Bug fixes apenas
- Nenhuma nova feature
- Urgent security fixes

---

## 🔗 Documentação Relacionada

- **[../11-development/](../11-development/)** - Development workflow
- **[../01-architecture/005-DEPLOYMENT.md](../01-architecture/005-DEPLOYMENT.md)** - Deployment

---

## 📚 Recursos

- [Semantic Versioning](https://semver.org/)
- [Keep a Changelog](https://keepachangelog.com/)

---

**Releases organizadas e previsíveis!** 🚀
