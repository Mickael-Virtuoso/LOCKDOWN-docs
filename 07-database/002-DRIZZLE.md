# Drizzle ORM - Overview

## O que é Drizzle ORM?

**Drizzle ORM** é um ORM TypeScript-first, type-safe e performático usado no LOCKDOWN para trabalhar com **PostgreSQL** e **SQLite**.

---

## Por Que Drizzle?

### Vantagens

```typescript
✅ Type-safety completo
   - Inferência automática de tipos
   - Erros de tipo em tempo de compilação
   - Autocomplete perfeito

✅ SQL-like API
   - Sintaxe próxima ao SQL nativo
   - Fácil de ler e entender
   - Sem magia, sem DSL complexa

✅ Performance
   - Queries otimizadas
   - Sem overhead
   - Suporta prepared statements

✅ Multi-database
   - PostgreSQL ✅
   - SQLite ✅
   - MySQL (se precisar)

✅ Migrations automáticas
   - Drizzle Kit gera migrations
   - A partir dos schemas TypeScript
   - Sem SQL manual
```

---

## Arquitetura do Drizzle no LOCKDOWN

```
Schema TypeScript (src/db/schema.ts)
    ↓
Drizzle Kit (gera migrations)
    ↓
SQL Migration Files (drizzle/*.sql)
    ↓
Aplicar no Database
    ↓
PostgreSQL (prod) / SQLite (dev)
```

---

## Stack Completa

```yaml
Development:
  Database: SQLite (better-sqlite3)
  ORM: Drizzle ORM
  Driver: drizzle-orm/better-sqlite3

Production:
  Database: PostgreSQL
  ORM: Drizzle ORM
  Driver: drizzle-orm/node-postgres (pg)
```

---

## Instalação

```bash
# Core
pnpm add drizzle-orm

# Drivers
pnpm add pg better-sqlite3

# Dev tools
pnpm add -D drizzle-kit @types/pg @types/better-sqlite3
```

---

## Configuração

### drizzle.config.ts

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  // Dialect: postgres ou sqlite
  dialect: process.env.DATABASE_PROVIDER === 'postgres'
    ? 'postgresql'
    : 'sqlite',

  // Schema source
  schema: './src/db/schema.ts',

  // Output migrations
  out: './drizzle',

  // Database credentials
  dbCredentials: process.env.DATABASE_PROVIDER === 'postgres'
    ? { url: process.env.DATABASE_URL! }
    : { url: 'file:./dev.db' }
});
```

### Database Connection

**SQLite (Development)**

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database('dev.db');
export const db = drizzle(sqlite);
```

**PostgreSQL (Production)**

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20
});

export const db = drizzle(pool);
```

---

## Definindo Schemas

### Schema Básico

```typescript
// src/db/schema.ts
import { pgTable, text, timestamp, integer } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').unique(),
  createdAt: timestamp('created_at').defaultNow()
});

// Type inference automático
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
```

### Com Relações

```typescript
export const guilds = pgTable('guilds', {
  id: text('id').primaryKey(),
  name: text('name').notNull()
});

export const members = pgTable('members', {
  id: text('id').primaryKey(),
  userId: text('user_id').references(() => users.id),
  guildId: text('guild_id').references(() => guilds.id),
  joinedAt: timestamp('joined_at').defaultNow()
});

// Define relations
export const usersRelations = relations(users, ({ many }) => ({
  members: many(members)
}));

export const membersRelations = relations(members, ({ one }) => ({
  user: one(users, {
    fields: [members.userId],
    references: [users.id]
  }),
  guild: one(guilds, {
    fields: [members.guildId],
    references: [guilds.id]
  })
}));
```

---

## CRUD Operations

### Select (Query)

```typescript
import { db } from './db';
import { users } from './schema';
import { eq, and, or, like, gt } from 'drizzle-orm';

// Select all
const allUsers = await db.select().from(users);

// Select specific columns
const userNames = await db
  .select({ name: users.name })
  .from(users);

// With WHERE
const user = await db
  .select()
  .from(users)
  .where(eq(users.id, '123'));

// Multiple conditions
const activeUsers = await db
  .select()
  .from(users)
  .where(
    and(
      eq(users.isActive, true),
      gt(users.createdAt, new Date('2024-01-01'))
    )
  );

// LIKE
const searchUsers = await db
  .select()
  .from(users)
  .where(like(users.name, '%john%'));

// Limit & Offset
const paginatedUsers = await db
  .select()
  .from(users)
  .limit(10)
  .offset(20);
```

### Insert

```typescript
// Single insert
await db.insert(users).values({
  id: '123',
  name: 'John Doe',
  email: 'john@example.com'
});

// Multiple insert
await db.insert(users).values([
  { id: '1', name: 'John' },
  { id: '2', name: 'Jane' }
]);

// Returning
const [newUser] = await db
  .insert(users)
  .values({ id: '123', name: 'John' })
  .returning();

console.log(newUser); // { id: '123', name: 'John', ... }
```

### Update

```typescript
// Update with where
await db
  .update(users)
  .set({ name: 'Jane Doe' })
  .where(eq(users.id, '123'));

// Update multiple fields
await db
  .update(users)
  .set({
    name: 'Jane',
    email: 'jane@example.com',
    updatedAt: new Date()
  })
  .where(eq(users.id, '123'));

// Returning updated row
const [updated] = await db
  .update(users)
  .set({ name: 'Jane' })
  .where(eq(users.id, '123'))
  .returning();
```

### Delete

```typescript
// Delete with where
await db
  .delete(users)
  .where(eq(users.id, '123'));

// Delete multiple
await db
  .delete(users)
  .where(
    and(
      eq(users.isActive, false),
      lt(users.createdAt, new Date('2020-01-01'))
    )
  );

// Returning deleted
const [deleted] = await db
  .delete(users)
  .where(eq(users.id, '123'))
  .returning();
```

---

## Joins

```typescript
// Inner Join
const result = await db
  .select({
    userName: users.name,
    guildName: guilds.name
  })
  .from(members)
  .innerJoin(users, eq(members.userId, users.id))
  .innerJoin(guilds, eq(members.guildId, guilds.id));

// Left Join
const usersWithGuilds = await db
  .select()
  .from(users)
  .leftJoin(members, eq(users.id, members.userId))
  .leftJoin(guilds, eq(members.guildId, guilds.id));

// With relations (easier)
const usersWithMembers = await db.query.users.findMany({
  with: {
    members: {
      with: {
        guild: true
      }
    }
  }
});
```

---

## Migrations

### Workflow

```bash
# 1. Definir schemas em TypeScript
# src/db/schema.ts

# 2. Gerar migration
pnpm drizzle-kit generate

# 3. Verificar migration gerada
# drizzle/0001_xxx.sql

# 4. Aplicar migration
pnpm drizzle-kit migrate

# Ou usar custom migrate script
pnpm db:migrate
```

### Custom Migrate Script

```typescript
// scripts/migrate.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { migrate } from 'drizzle-orm/node-postgres/migrator';
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL
});

const db = drizzle(pool);

async function main() {
  console.log('Running migrations...');

  await migrate(db, { migrationsFolder: './drizzle' });

  console.log('Migrations complete!');
  process.exit(0);
}

main().catch((err) => {
  console.error('Migration failed!', err);
  process.exit(1);
});
```

---

## Drizzle Studio

Interface visual para visualizar e editar dados:

```bash
pnpm drizzle-kit studio

# Abre em: https://local.drizzle.studio
```

Features:
- ✅ Visualizar tabelas
- ✅ Editar dados
- ✅ Ver relações
- ✅ Query builder visual

---

## Transactions

```typescript
await db.transaction(async (tx) => {
  // Insert user
  const [user] = await tx
    .insert(users)
    .values({ id: '123', name: 'John' })
    .returning();

  // Insert member
  await tx.insert(members).values({
    id: '456',
    userId: user.id,
    guildId: 'guild-1'
  });

  // Se qualquer operação falhar, tudo é revertido
});
```

---

## Prepared Statements

```typescript
// Definir prepared statement
const getUserById = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('get_user_by_id');

// Usar múltiplas vezes (cache query plan)
const user1 = await getUserById.execute({ id: '123' });
const user2 = await getUserById.execute({ id: '456' });
```

---

## Raw SQL (quando necessário)

```typescript
import { sql } from 'drizzle-orm';

// Execute raw SQL
const result = await db.execute(
  sql`SELECT * FROM users WHERE created_at > ${new Date('2024-01-01')}`
);

// Com tipagem
const users = await db.execute<{ id: string; name: string }>(
  sql`SELECT id, name FROM users`
);
```

---

## Best Practices

```typescript
// ✅ DO: Use type inference
const users = await db.select().from(usersTable);
// users é tipado automaticamente

// ✅ DO: Use transactions para operações relacionadas
await db.transaction(async (tx) => {
  await tx.insert(users).values(user);
  await tx.insert(members).values(member);
});

// ✅ DO: Use prepared statements para queries repetidas
const getUser = db.select()...prepare();

// ❌ DON'T: Construir SQL com string concatenation
// Use sql template tag

// ❌ DON'T: Queries N+1
// Use joins ou relational queries
```

---

## Drizzle vs Outros ORMs

| Feature | Drizzle | Prisma | TypeORM |
|---------|---------|--------|---------|
| **Type Safety** | ✅ Excelente | ✅ Excelente | ⚠️ Bom |
| **Performance** | ✅ Rápido | ⚠️ Overhead | ⚠️ Overhead |
| **Bundle Size** | ✅ Pequeno | ❌ Grande | ❌ Grande |
| **SQL-like API** | ✅ Sim | ❌ Não | ⚠️ Parcial |
| **Learning Curve** | ✅ Baixa | ⚠️ Média | ❌ Alta |
| **Migrations** | ✅ Automático | ✅ Automático | ⚠️ Manual |

---

## Recursos

- [Drizzle ORM Docs](https://orm.drizzle.team/)
- [Drizzle Kit Docs](https://orm.drizzle.team/kit-docs/overview)
- [Drizzle Examples](https://github.com/drizzle-team/drizzle-orm/tree/main/examples)

---

**Drizzle: Type-safe, performático e simples!** 🚀
