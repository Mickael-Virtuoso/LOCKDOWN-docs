# 💻 Development Workflow

## Visão Geral

Documentação de workflows de desenvolvimento, git flow, testes e boas práticas para colaboração em equipe.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-TESTING.md](./001-TESTING.md)** ⭐ **COMECE AQUI!**
   - Estratégia de testes
   - Unit, Integration, E2E tests
   - Test coverage
   - Testing best practices

2. **[002-REFACTOR_GUIDE.md](./002-REFACTOR_GUIDE.md)**
   - Guia de refatoração
   - Quando refatorar
   - Como refatorar com segurança
   - Code smells

3. **[003-workflow/](./003-workflow/)** - Git Workflow
   - **[000-FLOW.md](./003-workflow/000-FLOW.md)** - Git flow geral
   - **[001-BRANCH.md](./003-workflow/001-BRANCH.md)** - Branch strategy
   - **[002-COMMIT.md](./003-workflow/002-COMMIT.md)** - Commit messages
   - **[003-PUSH.md](./003-workflow/003-PUSH.md)** - Push guidelines
   - **[004-PULL.md](./003-workflow/004-PULL.md)** - Pull requests

---

## 🎯 Workflow Resumido

```
1. Criar branch       → git checkout -b feature/nome
2. Desenvolver        → escrever código + testes
3. Commit             → git commit -m "feat: algo"
4. Push               → git push origin feature/nome
5. Pull Request       → criar PR no GitHub
6. Code Review        → revisar com equipe
7. Merge              → merge para main
```

---

## 🌿 Branch Strategy

```
main              - Produção (sempre estável)
├── develop       - Development (integração)
├── feature/*     - Novas features
├── fix/*         - Bug fixes
├── hotfix/*      - Fixes urgentes
└── release/*     - Release candidates
```

---

## 💬 Commit Messages

Seguimos [Conventional Commits](https://www.conventionalcommits.org/):

```bash
feat: adiciona novo comando de moderação
fix: corrige bug no sistema de logs
docs: atualiza documentação da API
style: formata código com prettier
refactor: refatora serviço de autenticação
test: adiciona testes para PolicyEngine
chore: atualiza dependências
```

---

## 🧪 Testes

### Pirâmide de Testes

```
       /\
      /E2E\       (Poucos, slow)
     /------\
    /  INT   \    (Moderados, medium)
   /----------\
  /   UNIT     \  (Muitos, fast)
 /--------------\
```

### Comandos

```bash
pnpm test              # Todos os testes
pnpm test:unit         # Unit tests
pnpm test:integration  # Integration tests
pnpm test:e2e          # E2E tests
pnpm test:watch        # Watch mode
pnpm test:coverage     # Coverage report
```

---

## 🔍 Code Review Checklist

- [ ] Código segue padrões do projeto
- [ ] Testes foram adicionados/atualizados
- [ ] Documentação foi atualizada
- [ ] Sem warnings de lint/type-check
- [ ] Coverage mantido ou melhorado
- [ ] Sem código comentado (dead code)
- [ ] Performance não degradou
- [ ] Segurança foi considerada

---

## 📝 Pull Request Template

```markdown
## Descrição
[Descreva as mudanças]

## Tipo de mudança
- [ ] Bug fix
- [ ] Nova feature
- [ ] Breaking change
- [ ] Documentação

## Checklist
- [ ] Testes passando
- [ ] Lint/Type-check ok
- [ ] Documentação atualizada
- [ ] Breaking changes documentadas

## Screenshots (se aplicável)
[Adicione screenshots]
```

---

## ⚙️ Pre-commit Hooks

Validações automáticas antes de commit:

```bash
# Instalado via husky
.husky/pre-commit
  ├── lint-staged
  ├── type-check
  └── test:unit
```

---

## 💡 Best Practices

1. **Commits pequenos e focados**
2. **Escreva testes primeiro (TDD)**
3. **Code review obrigatório**
4. **Nunca push direto para main**
5. **Mantenha PRs pequenos (< 400 linhas)**
6. **Documente decisões importantes**

---

## 🔗 Documentação Relacionada

- **[../04-backend/005-PATTERNS.md](../04-backend/005-PATTERNS.md)** - Code patterns
- **[../12-release/](../12-release/)** - Release process

---

## 📚 Recursos

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [Testing Best Practices](https://testingjavascript.com/)

---

**Código de qualidade através de processos!** 💻
