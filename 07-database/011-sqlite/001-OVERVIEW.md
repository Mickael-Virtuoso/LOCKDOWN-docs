# SQLite - Overview

SQLite é usado no LOCKDOWN exclusivamente para **desenvolvimento local**. É um banco de dados em arquivo único, sem servidor.

## Por Que SQLite em Development?

### Vantagens

```
✅ Zero configuração
   - Não precisa instalar nada
   - Não precisa rodar servidor
   - Apenas um arquivo: dev.db

✅ Setup instantâneo
   - Clone repo → pnpm install → pnpm dev
   - Pronto!

✅ Fácil de resetar
   - rm dev.db
   - pnpm db:migrate
   - Banco zerado

✅ Portável
   - Commit dev.db no git (com seed data)
   - Toda equipe tem mesmos dados

✅ Performance suficiente
   - Para desenvolvimento local, é rápido
```

### Limitações (vs PostgreSQL)

```
❌ Concurrent writes limitados
   - Uma escrita por vez
   - Desenvolvimento local: OK
   - Produção: ❌ Não escala

❌ Tipos de dados limitados
   - INTEGER, TEXT, REAL, BLOB
   - Sem JSONB, ARRAY, etc.

❌ Features SQL limitadas
   - Sem window functions robustas
   - Sem full-text search avançado
   - Foreign keys opcionais

❌ Sem user management
   - Sem roles, sem permissions
   - Arquivo é o "user"
```

## Driver: better-sqlite3

```bash
pnpm add better-sqlite3
pnpm add -D @types/better-sqlite3
```

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database('dev.db');
export const db = drizzle(sqlite);
```

## Drizzle com SQLite

```typescript
// drizzle.config.ts
export default defineConfig({
  dialect: 'sqlite',
  schema: './src/db/schema.ts',
  out: './drizzle',
  dbCredentials: {
    url: 'file:./dev.db'
  }
});
```

## Workflow de Desenvolvimento

```bash
# 1. Clone repo
git clone ...
cd LOCKDOWN

# 2. Install
pnpm install

# 3. Migrate (cria dev.db)
pnpm db:migrate

# 4. Seed (opcional)
pnpm db:seed

# 5. Develop
pnpm dev

# Backend: http://localhost:3000
# Database: ./packages/backend/dev.db
```

## Visualizar Dados

```bash
# Drizzle Studio (recomendado)
pnpm db:studio
# https://local.drizzle.studio

# SQLite CLI
sqlite3 dev.db
.tables
.schema users
SELECT * FROM users;
.quit

# VSCode Extension
# SQLite Viewer
# https://marketplace.visualstudio.com/items?itemName=qwtel.sqlite-viewer
```

## Quando Usar SQLite vs PostgreSQL

### Use SQLite (Development)

```
✅ Desenvolvimento local
✅ Testes unitários/integração
✅ Protótipos rápidos
✅ Demos offline
```

### Use PostgreSQL (Production)

```
✅ Produção
✅ Múltiplos usuários concorrentes
✅ Features avançadas (JSONB, full-text search)
✅ Escalabilidade
✅ Alta disponibilidade
```

## Schema Compartilhado

Drizzle permite usar **mesmo schema** em ambos:

```typescript
// src/db/schema.ts
// Funciona em SQLite E PostgreSQL!

import { sqliteTable, text } from 'drizzle-orm/sqlite-core';
// ou
import { pgTable, text } from 'drizzle-orm/pg-core';

export const users = sqliteTable('users', { // dev
// export const users = pgTable('users', { // prod
  id: text('id').primaryKey(),
  name: text('name').notNull()
});
```

### Melhor: Conditional Schema

```typescript
// src/db/schema.ts
const isProduction = process.env.DATABASE_PROVIDER === 'postgres';

const tableFactory = isProduction ? pgTable : sqliteTable;
const textType = isProduction ? pgText : sqliteText;

export const users = tableFactory('users', {
  id: textType('id').primaryKey(),
  name: textType('name').notNull()
});
```

## Resetar Database

```bash
# Development
rm dev.db
pnpm db:migrate
pnpm db:seed

# Ou script
pnpm db:reset
```

---

**SQLite: simplicidade para desenvolvimento!** 📁
