# Database - Relacionamentos

## ERD Simplificado

```
Users ──────────┐
                ├──── Members ──── Guilds
                │
                ├──── Bans
                ├──── Kicks
                ├──── Mutes
                ├──── Warnings
                ├──── Whitelist
                └──── Blacklist

Guilds ──── AutoRoles
       ──── RestrictRoles
       ──── GuildConfig
       ──── AuditLogs
```

---

## Foreign Keys

### Members → Users + Guilds

```typescript
foreignKey({
  columns: [table.guildId],
  foreignColumns: [guildsTable.guildId],
  onDelete: 'cascade',
}),
foreignKey({
  columns: [table.userId],
  foreignColumns: [usersTable.userId],
  onDelete: 'cascade',
}),
```

### Moderation → Users + Guilds

```typescript
// Todas as tabelas de moderação seguem o padrão
foreignKey({
  columns: [table.guildId],
  foreignColumns: [guildsTable.guildId],
  onDelete: 'cascade',
}),
foreignKey({
  columns: [table.userId],
  foreignColumns: [usersTable.userId],
  onDelete: 'cascade',
}),
```

---

## Cascade Delete

Quando deleta uma Guild, deleta automaticamente:
- Todos os Members
- Todos os Bans
- Todos os Kicks
- Todos os Mutes
- Todas as Warnings
- Whitelist/Blacklist
- Configurações
- Audit Logs

```typescript
foreignKey({
  columns: [table.guildId],
  foreignColumns: [guildsTable.guildId],
  onDelete: 'cascade', // ← Automático
});
```

---

## Unique Constraints

### Member Único por Guild

```typescript
unique('unique_member').on(table.guildId, table.userId)
```

Um usuário só pode ser membro de uma guild uma vez.

### Discord IDs Únicos

```typescript
unique('unique_user_discord_id').on(table.userId)
unique('unique_guild_discord_id').on(table.guildId)
```

---

## Joins Comuns

### Guild com Bans

```typescript
db.select({
  guild: guildsTable,
  bans: bansTable,
})
.from(guildsTable)
.leftJoin(bansTable, eq(guildsTable.guildId, bansTable.guildId))
```

### Ban com User

```typescript
db.select({
  ban: bansTable,
  user: usersTable,
})
.from(bansTable)
.innerJoin(usersTable, eq(bansTable.userId, usersTable.userId))
```

---

## Links Relacionados

- [Schemas](./SCHEMAS.md)
- [Queries](./QUERIES.md)
- [Performance](./PERFORMANCE.md)
