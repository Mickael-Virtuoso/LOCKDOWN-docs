# Database - LOCKDOWN

Documentação completa do banco de dados da plataforma LOCKDOWN.

---

## Índice

| Documento | Descrição |
|-----------|-----------|
| [Visão Geral](./OVERVIEW.md) | Estrutura e providers |
| [Schemas](./SCHEMAS.md) | Definições de tabelas |
| [Relacionamentos](./RELATIONSHIPS.md) | Foreign keys e ERD |
| [Migrations](./MIGRATIONS.md) | Versionamento de schema |
| [Queries](./QUERIES.md) | Exemplos de queries |
| [Performance](./PERFORMANCE.md) | Índices e otimização |
| [Backup](./BACKUP.md) | Backup e restore |

---

## Quick Start

```bash
# Aplicar migrations
pnpm db:migrate

# Abrir Drizzle Studio
pnpm drizzle-kit studio
```

---

## Stack

| Ambiente | Driver | ORM |
|----------|--------|-----|
| Development | SQLite | Drizzle |
| Production | PostgreSQL | Drizzle |

---

## Schemas

```
📦 Banco de Dados
├─ core        → Users, Members, Guilds
├─ moderation  → Bans, Kicks, Mutes, Warnings
├─ config      → Auto-roles, Settings
├─ sharing     → Ban Lists compartilhados
└─ audit       → Audit Logs
```

---

## Referências

- [Backend](../04-backend/BACKEND.md)
- [API](../08-api/API.md)
