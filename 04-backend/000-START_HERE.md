# 🔧 Backend Development Guide

## Visão Geral

Documentação completa para desenvolvimento backend no LOCKDOWN. Tudo que você precisa para criar APIs, serviços e integrações.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - Visão geral do backend
   - Stack tecnológica (Fastify, Drizzle, Redis)
   - Arquitetura geral
   - Responsabilidades do backend

2. **[002-SETUP.md](./002-SETUP.md)**
   - Configuração do ambiente de desenvolvimento
   - Instalação de dependências
   - Configuração de banco de dados
   - Variáveis de ambiente específicas

3. **[003-STRUCTURE.md](./003-STRUCTURE.md)**
   - Estrutura de pastas detalhada
   - Organização de código
   - Convenções de nomenclatura
   - Onde colocar cada tipo de arquivo

4. **[004-DEVELOPMENT.md](./004-DEVELOPMENT.md)**
   - Como criar novos endpoints
   - Criação de routes, controllers, services
   - Validação de dados
   - Middlewares

5. **[005-PATTERNS.md](./005-PATTERNS.md)**
   - Padrões de código obrigatórios
   - Best practices
   - Code style
   - Design patterns utilizados

6. **[006-TESTING.md](./006-TESTING.md)**
   - Estratégia de testes
   - Testes unitários e de integração
   - Mocking e fixtures
   - Coverage

7. **[007-DEPLOYMENT.md](./007-DEPLOYMENT.md)**
   - Deploy em produção
   - Build process
   - Configurações de ambiente
   - Monitoramento

8. **[008-BACKEND.md](./008-BACKEND.md)**
   - Índice completo
   - Referência rápida
   - Links para recursos externos

---

## 🎯 Stack Tecnológica

```
├── Fastify      - Web framework
├── Drizzle ORM  - Database ORM
├── Redis        - Cache & Pub/Sub
├── SQLite/PG    - Database
├── Zod          - Validation
└── Pino         - Logging
```

---

## 🚀 Quick Start

```bash
# Setup backend
cd packages/backend
pnpm install

# Configure .env
cp .env.example .env

# Run migrations
pnpm db:migrate

# Start dev server
pnpm dev

# Backend running on http://localhost:3000
```

---

## 📁 Estrutura Resumida

```
packages/backend/
├── src/
│   ├── routes/       - API endpoints
│   ├── services/     - Business logic
│   ├── db/           - Database schemas
│   ├── middleware/   - Middlewares
│   └── utils/        - Utilities
└── tests/            - Tests
```

---

## 💡 Para Novos Backend Developers

1. Comece pelo [001-OVERVIEW.md](./001-OVERVIEW.md)
2. Configure seu ambiente com [002-SETUP.md](./002-SETUP.md)
3. Estude a estrutura em [003-STRUCTURE.md](./003-STRUCTURE.md)
4. Crie seu primeiro endpoint seguindo [004-DEVELOPMENT.md](./004-DEVELOPMENT.md)
5. Aplique os padrões de [005-PATTERNS.md](./005-PATTERNS.md)

---

## 🔗 Documentação Relacionada

- **[../07-database/](../07-database/)** - Schemas e queries
- **[../08-api/](../08-api/)** - Endpoints da API
- **[../09-security/](../09-security/)** - Autenticação e autorização

---

**Build APIs robustas e escaláveis!** 🔧
