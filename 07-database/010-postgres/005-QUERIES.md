# Database - Queries

## Select

### Selecionar Usuário

```typescript
import { eq } from 'drizzle-orm';
import { usersTable } from '@database/schemas/core';

async findUserByDiscordId(userId: string) {
  const [user] = await db
    .select()
    .from(usersTable)
    .where(eq(usersTable.userId, userId));

  return user ?? null;
}
```

### Listar Bans Ativos

```typescript
import { and, eq, desc } from 'drizzle-orm';

async getActiveBans(guildId: string) {
  return db
    .select()
    .from(bansTable)
    .where(
      and(
        eq(bansTable.guildId, guildId),
        eq(bansTable.isActive, true)
      )
    )
    .orderBy(desc(bansTable.createdAt));
}
```

---

## Insert

### Inserir Ban

```typescript
async createBan(data: {
  guildId: string;
  userId: string;
  reason: string;
  moderatorId: string;
  expiresAt?: Date;
}) {
  const [ban] = await db
    .insert(bansTable)
    .values({
      ...data,
      isActive: true,
      createdAt: new Date(),
    })
    .returning();

  return ban;
}
```

---

## Update

### Atualizar Ban

```typescript
async removeBan(banId: number) {
  await db
    .update(bansTable)
    .set({
      isActive: false,
      updatedAt: new Date(),
    })
    .where(eq(bansTable.id, banId));
}
```

---

## Delete

### Deletar Ban

```typescript
async deleteBan(banId: number) {
  const result = await db
    .delete(bansTable)
    .where(eq(bansTable.id, banId));

  return result.rowsAffected > 0;
}
```

---

## Joins

### Guild com Bans

```typescript
import { leftJoin, eq } from 'drizzle-orm';

async getGuildWithBans(guildId: string) {
  return db
    .select({
      guild: guildsTable,
      bans: bansTable,
    })
    .from(guildsTable)
    .leftJoin(
      bansTable,
      eq(guildsTable.guildId, bansTable.guildId)
    )
    .where(eq(guildsTable.guildId, guildId));
}
```

### Ban com User

```typescript
async getBansWithUsers(guildId: string) {
  return db
    .select({
      ban: bansTable,
      user: usersTable,
    })
    .from(bansTable)
    .innerJoin(usersTable, eq(bansTable.userId, usersTable.userId))
    .where(eq(bansTable.guildId, guildId));
}
```

---

## Aggregations

### Contar Bans

```typescript
import { count } from 'drizzle-orm';

async countActiveBans(guildId: string): Promise<number> {
  const [result] = await db
    .select({ total: count() })
    .from(bansTable)
    .where(
      and(
        eq(bansTable.guildId, guildId),
        eq(bansTable.isActive, true)
      )
    );

  return result?.total ?? 0;
}
```

---

## Pagination

### Listar com Paginação

```typescript
async listBans(guildId: string, page: number = 1, pageSize: number = 50) {
  const offset = (page - 1) * pageSize;

  const bans = await db
    .select()
    .from(bansTable)
    .where(eq(bansTable.guildId, guildId))
    .limit(pageSize)
    .offset(offset)
    .orderBy(desc(bansTable.createdAt));

  return {
    data: bans,
    page,
    pageSize,
  };
}
```

---

## Transactions

### Operação Atômica

```typescript
async banAndAudit(banData: InsertBan, auditData: InsertAudit) {
  return db.transaction(async (tx) => {
    const [ban] = await tx.insert(bansTable).values(banData).returning();
    const [audit] = await tx.insert(auditLogsTable).values(auditData).returning();
    return { ban, audit };
  });
}
```

---

## Links Relacionados

- [Schemas](./SCHEMAS.md)
- [Performance](./PERFORMANCE.md)
- [Backend Development](../04-backend/DEVELOPMENT.md)
