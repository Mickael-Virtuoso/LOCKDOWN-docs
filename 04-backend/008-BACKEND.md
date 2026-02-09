# Backend - LOCKDOWN

Documentação completa do backend da plataforma LOCKDOWN.

---

## Índice

| Documento | Descrição |
|-----------|-----------|
| [Visão Geral](./OVERVIEW.md) | Stack, arquitetura e princípios |
| [Setup](./SETUP.md) | Instalação e configuração |
| [Estrutura](./STRUCTURE.md) | Organização de pastas |
| [Desenvolvimento](./DEVELOPMENT.md) | Criação de endpoints |
| [Padrões](./PATTERNS.md) | Convenções de código |
| [Testing](./TESTING.md) | Testes unitários e integração |
| [Deployment](./DEPLOYMENT.md) | Build e deploy |

---

## Quick Start

```bash
# Instalar dependências
pnpm install

# Iniciar em desenvolvimento
cd packages/backend
pnpm dev

# Verificar saúde
curl http://localhost:3000/api/v1/health
```

---

## Stack

| Componente | Versão | Propósito |
|------------|--------|-----------|
| Express.js | ^4.18.2 | Web framework |
| TypeScript | latest | Tipagem |
| Drizzle ORM | ^0.30.0 | Database |
| Redis | 5.3.2 | Cache/Pub-Sub |
| Pino | ^8.17.0 | Logging |

---

## Arquitetura

```
Request → Routes → Controllers → Services → Repositories → Database
                       ↓
                   Middlewares
```

---

## Referências

- [Arquitetura Geral](../01-architecture/ARCHITECTURE.md)
- [Database](../07-database/DATABASE.md)
- [API](../08-api/API.md)
