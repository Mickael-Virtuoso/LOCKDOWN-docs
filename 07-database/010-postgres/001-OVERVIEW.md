# PostgreSQL - Overview

PostgreSQL é o banco de dados principal do LOCKDOWN em **produção**. Oferece features SQL completas que não estão disponíveis no SQLite.

## Por Que PostgreSQL em Produção?

### Features Avançadas

```typescript
// 1. JSONB com queries
await db.select().from(users).where(
  sql`data->>'role' = 'admin'`
);

// 2. Array types
create table posts (
  tags text[]
);

// 3. Full-text search
create index posts_search_idx on posts using gin(to_tsvector('english', content));

// 4. Window functions
select
  username,
  score,
  rank() over (order by score desc) as ranking
from users;

// 5. CTEs (Common Table Expressions)
with active_users as (
  select * from users where last_seen > now() - interval '30 days'
)
select * from active_users;
```

## Driver Stack

```
Application
    ↓
Drizzle ORM (type-safe queries)
    ↓
pg (PostgreSQL driver)
    ↓
PostgreSQL Database
```

### Instalação

```bash
pnpm add drizzle-orm pg
pnpm add -D drizzle-kit @types/pg
```

## Drizzle com PostgreSQL

```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  dialect: 'postgresql',
  schema: './src/db/schema.ts',
  out: './drizzle',
  dbCredentials: {
    url: process.env.DATABASE_URL!
  }
});
```

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // connection pool size
});

export const db = drizzle(pool);
```

## PostgreSQL vs SQLite

| Feature | SQLite | PostgreSQL |
|---------|--------|------------|
| **Concurrent Writes** | ❌ Single writer | ✅ Multiple writers |
| **Data Types** | 5 basic types | 40+ types |
| **JSON** | Limited | ✅ JSONB native |
| **Full-text Search** | Basic | ✅ Advanced |
| **Foreign Keys** | Optional | ✅ Enforced |
| **Triggers** | Limited | ✅ Full support |
| **Views** | Basic | ✅ Materialized views |
| **Replication** | ❌ No | ✅ Built-in |
| **User Management** | ❌ File-based | ✅ Role-based |
| **Performance** | Great for reads | Scales to millions |

## Production Hosting Options

```yaml
Neon (Recomendado):
  - Serverless Postgres
  - Auto-scaling
  - Free tier generoso
  - Branching (dev environments)
  - https://neon.tech

Supabase:
  - Postgres + Auth + Storage
  - Real-time subscriptions
  - Free tier
  - https://supabase.com

Railway:
  - Simples e rápido
  - $5/mo
  - Backups automáticos
  - https://railway.app

AWS RDS:
  - Enterprise-grade
  - Multi-AZ
  - Mais caro
```

## Schema Migration (SQLite → PostgreSQL)

Drizzle gerencia automaticamente:

```typescript
// Same schema works for both!
import { pgTable, text, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow()
});
```

Deploy:

```bash
# Development (SQLite)
DATABASE_PROVIDER=sqlite pnpm db:migrate

# Production (PostgreSQL)
DATABASE_PROVIDER=postgres pnpm db:migrate
```

---

**PostgreSQL: poder e escala para produção!** 🐘
