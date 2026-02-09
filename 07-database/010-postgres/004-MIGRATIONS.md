# Database - Migrations

## Estrutura de Arquivos

```
database/migrations/
├─ 0001_initial_schema.sql
├─ 0002_moderation_tables.sql
├─ 0003_config_tables.sql
├─ 0004_sharing_tables.sql
└─ 0005_audit_tables.sql
```

---

## Exemplo: Create Bans Table

```sql
-- 0002_moderation_tables.sql

CREATE TABLE IF NOT EXISTS bans (
  id SERIAL PRIMARY KEY,
  guild_id VARCHAR(255) NOT NULL,
  user_id VARCHAR(255) NOT NULL,
  reason TEXT,
  moderator_id VARCHAR(255) NOT NULL,
  expires_at TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (guild_id) REFERENCES guilds(guild_id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

CREATE INDEX idx_bans_guild ON bans(guild_id);
CREATE INDEX idx_bans_user ON bans(user_id);
CREATE INDEX idx_bans_active ON bans(is_active);
```

---

## Comandos

### Drizzle Kit

```bash
# Gerar migrations a partir dos schemas
pnpm drizzle-kit generate:pg
pnpm drizzle-kit generate:sqlite

# Aplicar migrations
pnpm drizzle-kit push:pg
pnpm drizzle-kit push:sqlite

# Ver status
pnpm drizzle-kit check
```

### Script Customizado

```bash
# Rodar todas as migrations pendentes
pnpm db:migrate

# Reverter última migration
pnpm db:rollback

# Reset completo
pnpm db:reset
```

---

## Boas Práticas

### 1. Migrations são imutáveis

```sql
-- ❌ NUNCA edite uma migration já aplicada
-- ✅ Crie uma nova migration para alterações
```

### 2. Nomes descritivos

```
0001_initial_schema.sql
0002_add_bans_table.sql
0003_add_expires_at_to_bans.sql
0004_create_audit_logs.sql
```

### 3. Sempre com rollback

```sql
-- UP
CREATE TABLE bans (...);

-- DOWN (em arquivo separado ou comentado)
DROP TABLE IF EXISTS bans;
```

### 4. Testar antes de produção

```bash
# Testar em ambiente local
pnpm db:migrate

# Verificar schema
pnpm drizzle-kit studio
```

---

## Links Relacionados

- [Schemas](./SCHEMAS.md)
- [Visão Geral](./OVERVIEW.md)
- [Backend Setup](../04-backend/SETUP.md)
