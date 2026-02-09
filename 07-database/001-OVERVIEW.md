# Database - Visão Geral

## Estrutura Lógica

```
📦 Banco de Dados LOCKDOWN
├─ Schema: core              (Usuários, Membros, Guilds)
├─ Schema: moderation        (Bans, Kicks, Mutes, Warnings)
├─ Schema: config            (Auto-roles, Restrict-roles)
├─ Schema: sharing           (Ban Lists compartilhados)
└─ Schema: audit             (Audit Logs)
```

---

## Providers

| Ambiente | Driver | Setup | Performance |
|----------|--------|-------|-------------|
| **Development** | SQLite | Zero config | Rápido local |
| **Production** | PostgreSQL | Servidor | Escalável |

---

## ORM

Usamos **Drizzle ORM** para:
- Type-safe queries
- Schema definitions em TypeScript
- Migrações automáticas
- Suporte multi-database

---

## Configuração

### Development (SQLite)

```typescript
// config/database.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database('./data/lockdown.db');
export const db = drizzle(sqlite);
```

### Production (PostgreSQL)

```typescript
// config/database.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});
export const db = drizzle(pool);
```

---

## Links Relacionados

- [Schemas](./SCHEMAS.md)
- [Queries](./QUERIES.md)
- [Migrations](./MIGRATIONS.md)
- [Performance](./PERFORMANCE.md)
