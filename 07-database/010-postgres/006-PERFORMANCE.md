# Database - Performance

## Índices

### Core

```sql
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_guilds_name ON guilds(name);
CREATE INDEX idx_members_guild ON members(guild_id);
CREATE INDEX idx_members_user ON members(user_id);
```

### Moderation

```sql
CREATE INDEX idx_bans_guild ON bans(guild_id);
CREATE INDEX idx_bans_user ON bans(user_id);
CREATE INDEX idx_bans_active ON bans(is_active);
CREATE INDEX idx_kicks_guild ON kicks(guild_id);
CREATE INDEX idx_mutes_active ON mutes(is_active);
```

### Audit

```sql
CREATE INDEX idx_audit_guild ON audit_logs(guild_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

---

## Evitar N+1 Queries

### Problema

```typescript
// ❌ RUIM: 10 queries para 10 bans
const bans = await db.select().from(bansTable);

for (const ban of bans) {
  const user = await db
    .select()
    .from(usersTable)
    .where(eq(usersTable.userId, ban.userId));
}
```

### Solução

```typescript
// ✅ BOM: 1 query com JOIN
const bans = await db
  .select({
    ban: bansTable,
    user: usersTable,
  })
  .from(bansTable)
  .innerJoin(usersTable, eq(bansTable.userId, usersTable.userId));
```

---

## Paginação Eficiente

### Offset/Limit

```typescript
// Para datasets pequenos (< 10k rows)
const bans = await db
  .select()
  .from(bansTable)
  .limit(50)
  .offset(100);
```

### Cursor-based (Recomendado)

```typescript
// Para datasets grandes
const bans = await db
  .select()
  .from(bansTable)
  .where(gt(bansTable.id, lastId)) // Cursor
  .limit(50)
  .orderBy(asc(bansTable.id));
```

---

## Query Optimization

### Select apenas colunas necessárias

```typescript
// ❌ RUIM: Seleciona tudo
const bans = await db.select().from(bansTable);

// ✅ BOM: Apenas o necessário
const bans = await db
  .select({
    id: bansTable.id,
    reason: bansTable.reason,
    isActive: bansTable.isActive,
  })
  .from(bansTable);
```

### Usar WHERE antes de JOIN

```typescript
// ✅ Filtrar cedo
const bans = await db
  .select()
  .from(bansTable)
  .where(eq(bansTable.guildId, guildId))
  .innerJoin(usersTable, eq(bansTable.userId, usersTable.userId));
```

---

## Connection Pooling

### PostgreSQL

```typescript
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // Máximo de conexões
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

---

## Monitoring

### Query Timing

```typescript
const start = performance.now();
const result = await db.select().from(bansTable);
const duration = performance.now() - start;

logger.debug(`Query took ${duration.toFixed(2)}ms`);
```

### Slow Query Log

```typescript
// Log queries > 100ms
if (duration > 100) {
  logger.warn(`Slow query detected: ${duration.toFixed(2)}ms`);
}
```

---

## Links Relacionados

- [Queries](./QUERIES.md)
- [Schemas](./SCHEMAS.md)
- [Logging](../03-core/LOGGING.md)
