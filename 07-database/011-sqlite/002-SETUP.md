# SQLite - Setup e Configuração

## Instalação

### Driver: better-sqlite3

```bash
cd packages/backend
pnpm add better-sqlite3
pnpm add -D @types/better-sqlite3
```

**Por que better-sqlite3?**
```
✅ Síncrono (mais simples, sem async/await)
✅ Mais rápido que alternativas
✅ API simples e direta
✅ Bem mantido e popular
```

---

## Configuração do Drizzle

### drizzle.config.ts

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  // SQLite dialect
  dialect: 'sqlite',

  // Schema source
  schema: './src/db/schema.ts',

  // Output migrations
  out: './drizzle',

  // SQLite database file
  dbCredentials: {
    url: 'file:./dev.db'
  },

  // Optional: verbose logging
  verbose: true,

  // Optional: strict mode
  strict: true
});
```

### Database Connection

**Singleton Pattern (Recomendado)**

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

// Create singleton
let db: ReturnType<typeof drizzle> | null = null;

export function getDb() {
  if (!db) {
    const sqlite = new Database('dev.db', {
      // Enable verbose mode for debugging
      verbose: process.env.NODE_ENV === 'development'
        ? console.log
        : undefined
    });

    // Enable WAL mode for better concurrency
    sqlite.pragma('journal_mode = WAL');

    // Enable foreign keys
    sqlite.pragma('foreign_keys = ON');

    db = drizzle(sqlite);
  }

  return db;
}

// Export for use
export const db = getDb();
```

**Simples (para scripts)**

```typescript
// scripts/seed.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database('dev.db');
const db = drizzle(sqlite);

// Use db...
```

---

## Variáveis de Ambiente

**`packages/backend/.env`**

```ini
# Database Provider
DATABASE_PROVIDER=sqlite

# SQLite Database File
DATABASE_URL=file:./dev.db

# Optional: Enable query logging
DATABASE_LOGGING=true
```

### Ambiente Condicional

```typescript
// src/db/index.ts
const dbPath = process.env.NODE_ENV === 'test'
  ? ':memory:'  // In-memory for tests
  : 'dev.db';   // File for development

const sqlite = new Database(dbPath);
```

---

## Schemas Drizzle para SQLite

### Tabela Básica

```typescript
// src/db/schema.ts
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';
import { sql } from 'drizzle-orm';

export const users = sqliteTable('users', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').unique(),
  createdAt: integer('created_at', { mode: 'timestamp' })
    .default(sql`(unixepoch())`)
});

// Type inference
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
```

### Com Foreign Keys

```typescript
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';

export const guilds = sqliteTable('guilds', {
  id: text('id').primaryKey(),
  name: text('name').notNull()
});

export const members = sqliteTable('members', {
  id: text('id').primaryKey(),
  userId: text('user_id')
    .notNull()
    .references(() => users.id, { onDelete: 'cascade' }),
  guildId: text('guild_id')
    .notNull()
    .references(() => guilds.id, { onDelete: 'cascade' }),
  joinedAt: integer('joined_at', { mode: 'timestamp' })
    .default(sql`(unixepoch())`)
});
```

---

## Migrations

### Gerar Migration

```bash
# Gera migration a partir do schema
pnpm drizzle-kit generate

# Output:
# drizzle/0001_xxx.sql
```

### Aplicar Migrations

**Opção 1: Drizzle Kit (desenvolvimento)**

```bash
pnpm drizzle-kit migrate
```

**Opção 2: Custom Script (mais controle)**

```typescript
// scripts/migrate.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import { migrate } from 'drizzle-orm/better-sqlite3/migrator';
import Database from 'better-sqlite3';

const sqlite = new Database('dev.db');
const db = drizzle(sqlite);

async function main() {
  console.log('Running migrations...');

  await migrate(db, { migrationsFolder: './drizzle' });

  console.log('Migrations complete!');
  sqlite.close();
}

main();
```

**package.json**

```json
{
  "scripts": {
    "db:migrate": "tsx scripts/migrate.ts",
    "db:generate": "drizzle-kit generate",
    "db:studio": "drizzle-kit studio"
  }
}
```

---

## Drizzle Studio

Interface visual para visualizar e editar dados:

```bash
pnpm drizzle-kit studio

# Abre automaticamente no navegador
# https://local.drizzle.studio
```

**Features:**
- ✅ Visualizar todas as tabelas
- ✅ Ver dados em tempo real
- ✅ Editar dados (CRUD visual)
- ✅ Ver relacionamentos
- ✅ Executar queries customizadas
- ✅ Ver estrutura do schema

---

## SQLite CLI

### Instalação

```bash
# Ubuntu/Debian
sudo apt install sqlite3

# macOS
brew install sqlite3

# Verificar instalação
sqlite3 --version
```

### Comandos Úteis

```bash
# Abrir database
sqlite3 dev.db

# Listar tabelas
.tables

# Ver schema de uma tabela
.schema users

# Ver dados
SELECT * FROM users;

# Ver schema completo
.fullschema

# Dump database
.dump > backup.sql

# Import SQL
.read backup.sql

# Enable column headers
.headers on

# Enable column mode
.mode column

# Sair
.quit
```

---

## Pragmas SQLite

Otimizações importantes:

```typescript
// src/db/index.ts
const sqlite = new Database('dev.db');

// WAL mode (Write-Ahead Logging) - melhor concorrência
sqlite.pragma('journal_mode = WAL');

// Enable foreign keys (IMPORTANTE!)
sqlite.pragma('foreign_keys = ON');

// Sync mode (0=OFF, 1=NORMAL, 2=FULL)
sqlite.pragma('synchronous = NORMAL');

// Cache size (em páginas, -KB)
sqlite.pragma('cache_size = -64000'); // 64MB

// Temp store (memory)
sqlite.pragma('temp_store = MEMORY');

// MMM (memory-mapped I/O)
sqlite.pragma('mmap_size = 30000000000'); // 30GB
```

### Verificar Pragmas Ativos

```typescript
console.log(sqlite.pragma('journal_mode'));
console.log(sqlite.pragma('foreign_keys'));
```

---

## Performance Tips

### 1. Batch Inserts com Transaction

```typescript
// ❌ LENTO: Individual inserts
for (const user of users) {
  await db.insert(usersTable).values(user);
}

// ✅ RÁPIDO: Batch insert
await db.insert(usersTable).values(users);

// ✅ RÁPIDO: Transaction manual
const sqlite = new Database('dev.db');
const insertStmt = sqlite.prepare(
  'INSERT INTO users (id, name) VALUES (?, ?)'
);

const insertMany = sqlite.transaction((users) => {
  for (const user of users) {
    insertStmt.run(user.id, user.name);
  }
});

insertMany(users); // Muito mais rápido!
```

### 2. Prepared Statements

```typescript
// Drizzle prepared statement
const getUserById = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare();

// Use múltiplas vezes (reutiliza query plan)
const user1 = getUserById.execute({ id: '1' });
const user2 = getUserById.execute({ id: '2' });
```

### 3. Índices

```typescript
import { index } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: text('id').primaryKey(),
  email: text('email').unique(),
  name: text('name').notNull()
}, (table) => ({
  // Índice para busca por nome
  nameIdx: index('name_idx').on(table.name),

  // Índice composto
  emailNameIdx: index('email_name_idx').on(table.email, table.name)
}));
```

---

## Database em Memória (Testes)

```typescript
// tests/setup.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

// In-memory database (perfeito para testes)
const sqlite = new Database(':memory:');
const db = drizzle(sqlite);

// Run migrations
await migrate(db, { migrationsFolder: './drizzle' });

// Agora use db para testes
export { db };
```

**Vantagens:**
- ✅ Extremamente rápido
- ✅ Isolado (cada teste tem DB limpo)
- ✅ Sem arquivos no filesystem
- ✅ Perfeito para CI/CD

---

## Backup e Restore

### Backup Manual

```bash
# SQLite CLI
sqlite3 dev.db .dump > backup.sql

# Compactar
gzip backup.sql
```

### Backup via Script

```typescript
// scripts/backup.ts
import Database from 'better-sqlite3';
import fs from 'fs';
import { execSync } from 'child_process';

const timestamp = new Date().toISOString().replace(/:/g, '-');
const backupFile = `backups/dev-${timestamp}.db`;

// Copiar arquivo
fs.copyFileSync('dev.db', backupFile);

// Comprimir
execSync(`gzip ${backupFile}`);

console.log(`Backup created: ${backupFile}.gz`);
```

### Restore

```bash
# Restaurar de SQL dump
sqlite3 dev.db < backup.sql

# Ou copiar arquivo direto
gunzip backup.db.gz
cp backup.db dev.db
```

---

## Reset Database

Script útil para desenvolvimento:

```typescript
// scripts/reset-db.ts
import fs from 'fs';
import { execSync } from 'child_process';

// Delete database file
if (fs.existsSync('dev.db')) {
  fs.unlinkSync('dev.db');
  console.log('✅ Database deleted');
}

// Run migrations
execSync('pnpm db:migrate', { stdio: 'inherit' });

// Optional: seed
execSync('pnpm db:seed', { stdio: 'inherit' });

console.log('✅ Database reset complete!');
```

**package.json**

```json
{
  "scripts": {
    "db:reset": "tsx scripts/reset-db.ts"
  }
}
```

---

## Troubleshooting

### Database is locked

```typescript
// Aumentar timeout
const sqlite = new Database('dev.db', {
  timeout: 5000 // 5 segundos
});

// Ou enable WAL mode
sqlite.pragma('journal_mode = WAL');
```

### Foreign keys não funcionam

```typescript
// SEMPRE enable foreign keys!
sqlite.pragma('foreign_keys = ON');

// Verificar
console.log(sqlite.pragma('foreign_keys')); // deve ser 1
```

### Database corrompido

```bash
# Verificar integridade
sqlite3 dev.db "PRAGMA integrity_check;"

# Se corrompido, recover de backup
cp backup.db dev.db
```

### Performance lenta

```bash
# Vacuum (compacta DB)
sqlite3 dev.db "VACUUM;"

# Analyze (atualiza stats)
sqlite3 dev.db "ANALYZE;"

# Reindex
sqlite3 dev.db "REINDEX;"
```

---

## Ferramentas Visuais

### DB Browser for SQLite

```bash
# Ubuntu/Debian
sudo apt install sqlitebrowser

# macOS
brew install --cask db-browser-for-sqlite

# Abrir
sqlitebrowser dev.db
```

### VSCode Extension

```
Name: SQLite Viewer
ID: qwtel.sqlite-viewer
```

Permite visualizar `.db` files diretamente no VSCode!

---

## package.json Scripts Completo

```json
{
  "scripts": {
    "db:generate": "drizzle-kit generate",
    "db:migrate": "tsx scripts/migrate.ts",
    "db:studio": "drizzle-kit studio",
    "db:seed": "tsx scripts/seed.ts",
    "db:reset": "tsx scripts/reset-db.ts",
    "db:backup": "tsx scripts/backup.ts",
    "db:cli": "sqlite3 dev.db"
  }
}
```

---

## Próximos Passos

- [003-LIMITATIONS.md](./003-LIMITATIONS.md) - Limitações do SQLite
- [004-DEVELOPMENT.md](./004-DEVELOPMENT.md) - Workflow de desenvolvimento
- [005-DRIZZLE-SQLITE.md](./005-DRIZZLE-SQLITE.md) - Drizzle com SQLite

---

**SQLite: configuração simples, desenvolvimento rápido!** 📁
