# Database - Schemas

## Schema Core

### Users Table

```typescript
export const usersTable = pgTable(
  'users',
  {
    id: serial('id').primaryKey(),
    userId: varchar('user_id', { length: 255 }).notNull().unique(),
    username: varchar('username', { length: 255 }).notNull(),
    discriminator: varchar('discriminator', { length: 4 }),
    avatar: varchar('avatar', { length: 255 }),
    email: varchar('email', { length: 255 }).unique(),
    isBot: boolean('is_bot').default(false),
    createdAt: timestamp('created_at').defaultNow(),
    updatedAt: timestamp('updated_at').defaultNow(),
  },
  (table) => [
    unique('unique_user_discord_id').on(table.userId),
    index('idx_users_username').on(table.username),
  ]
);

export type SelectUser = typeof usersTable.$inferSelect;
export type InsertUser = typeof usersTable.$inferInsert;
```

### Guilds Table

```typescript
export const guildsTable = pgTable(
  'guilds',
  {
    id: serial('id').primaryKey(),
    guildId: varchar('guild_id', { length: 255 }).notNull().unique(),
    name: varchar('name', { length: 255 }).notNull(),
    icon: varchar('icon', { length: 255 }),
    ownerId: varchar('owner_id', { length: 255 }).notNull(),
    memberCount: integer('member_count').default(0),
    createdAt: timestamp('created_at').defaultNow(),
    updatedAt: timestamp('updated_at').defaultNow(),
  },
  (table) => [
    unique('unique_guild_discord_id').on(table.guildId),
    index('idx_guilds_name').on(table.name),
  ]
);
```

### Members Table

```typescript
export const membersTable = pgTable(
  'members',
  {
    id: serial('id').primaryKey(),
    guildId: varchar('guild_id', { length: 255 }).notNull(),
    userId: varchar('user_id', { length: 255 }).notNull(),
    nickname: varchar('nickname', { length: 255 }),
    roles: varchar('roles', { length: 1024 }), // JSON array
    joinedAt: timestamp('joined_at').notNull(),
    boostedAt: timestamp('boosted_at'),
    createdAt: timestamp('created_at').defaultNow(),
    updatedAt: timestamp('updated_at').defaultNow(),
  },
  (table) => [
    unique('unique_member').on(table.guildId, table.userId),
    index('idx_members_guild').on(table.guildId),
    index('idx_members_user').on(table.userId),
  ]
);
```

---

## Schema Moderation

### Bans Table

```typescript
export const bansTable = pgTable(
  'bans',
  {
    id: serial('id').primaryKey(),
    guildId: varchar('guild_id', { length: 255 }).notNull(),
    userId: varchar('user_id', { length: 255 }).notNull(),
    reason: text('reason'),
    moderatorId: varchar('moderator_id', { length: 255 }).notNull(),
    expiresAt: timestamp('expires_at'), // null = permanente
    isActive: boolean('is_active').default(true),
    createdAt: timestamp('created_at').defaultNow(),
    updatedAt: timestamp('updated_at').defaultNow(),
  },
  (table) => [
    index('idx_bans_guild').on(table.guildId),
    index('idx_bans_user').on(table.userId),
    index('idx_bans_active').on(table.isActive),
  ]
);
```

### Kicks Table

```typescript
export const kicksTable = pgTable(
  'kicks',
  {
    id: serial('id').primaryKey(),
    guildId: varchar('guild_id', { length: 255 }).notNull(),
    userId: varchar('user_id', { length: 255 }).notNull(),
    reason: text('reason'),
    moderatorId: varchar('moderator_id', { length: 255 }).notNull(),
    createdAt: timestamp('created_at').defaultNow(),
  },
  (table) => [
    index('idx_kicks_guild').on(table.guildId),
    index('idx_kicks_user').on(table.userId),
  ]
);
```

### Mutes Table

```typescript
export const mutesTable = pgTable(
  'mutes',
  {
    id: serial('id').primaryKey(),
    guildId: varchar('guild_id', { length: 255 }).notNull(),
    userId: varchar('user_id', { length: 255 }).notNull(),
    reason: text('reason'),
    moderatorId: varchar('moderator_id', { length: 255 }).notNull(),
    duration: integer('duration'), // minutos
    expiresAt: timestamp('expires_at'),
    isActive: boolean('is_active').default(true),
    createdAt: timestamp('created_at').defaultNow(),
  },
  (table) => [
    index('idx_mutes_guild').on(table.guildId),
    index('idx_mutes_active').on(table.isActive),
  ]
);
```

### Warnings Table

```typescript
export const warningsTable = pgTable(
  'warnings',
  {
    id: serial('id').primaryKey(),
    guildId: varchar('guild_id', { length: 255 }).notNull(),
    userId: varchar('user_id', { length: 255 }).notNull(),
    reason: text('reason'),
    moderatorId: varchar('moderator_id', { length: 255 }).notNull(),
    count: integer('count').default(1),
    createdAt: timestamp('created_at').defaultNow(),
  },
  (table) => [
    index('idx_warnings_guild').on(table.guildId),
  ]
);
```

### Whitelist + Blacklist

```typescript
export const whitelistTable = pgTable('whitelist', {
  id: serial('id').primaryKey(),
  guildId: varchar('guild_id', { length: 255 }).notNull(),
  userId: varchar('user_id', { length: 255 }).notNull(),
  reason: text('reason'),
  addedBy: varchar('added_by', { length: 255 }).notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});

export const blacklistTable = pgTable('blacklist', {
  id: serial('id').primaryKey(),
  guildId: varchar('guild_id', { length: 255 }).notNull(),
  userId: varchar('user_id', { length: 255 }).notNull(),
  reason: text('reason'),
  addedBy: varchar('added_by', { length: 255 }).notNull(),
  createdAt: timestamp('created_at').defaultNow(),
});
```

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Relacionamentos](./RELATIONSHIPS.md)
- [Queries](./QUERIES.md)
