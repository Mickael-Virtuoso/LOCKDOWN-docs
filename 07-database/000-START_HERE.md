# 🗄️ Database Documentation

## Visão Geral

Documentação completa do sistema de banco de dados do LOCKDOWN. O projeto utiliza **3 bancos de dados**:

- **Redis** - Cache e Pub/Sub (dev + prod)
- **PostgreSQL** - Database principal (production)
- **SQLite** - Database principal (development)

Todos gerenciados através do **Drizzle ORM** para type-safety completo.

---

## 📖 Ordem de Leitura Recomendada

### Documentação Geral

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - Visão geral dos 3 bancos de dados
   - Redis vs PostgreSQL vs SQLite
   - Quando usar cada um
   - Arquitetura geral

2. **[002-DRIZZLE.md](./002-DRIZZLE.md)**
   - O que é Drizzle ORM
   - Configuração e setup
   - CRUD operations
   - Migrations automáticas
   - Type-safety

### Documentação Específica por Banco

3. **[009-redis/](./009-redis/)** 🔴 **Redis (Cache & Pub/Sub)**
   - Overview e casos de uso
   - Setup (Docker + ioredis)
   - Pub/Sub para eventos
   - Caching strategies
   - Production deployment

4. **[010-postgres/](./010-postgres/)** 🐘 **PostgreSQL (Production)**
   - Overview e features avançadas
   - Setup e configuração
   - Schemas e relationships
   - Migrations e queries
   - Performance e backup
   - Production deployment

5. **[011-sqlite/](./011-sqlite/)** 📁 **SQLite (Development)**
   - Overview e por que em dev
   - Setup instantâneo
   - Limitações vs PostgreSQL
   - Workflow de desenvolvimento

---

## 🎯 Stack de Banco de Dados

```
Application Layer
    ↓
┌─────────────────┬──────────────────┬────────────────────┐
│  Redis          │  PostgreSQL      │  SQLite            │
│  (ioredis)      │  (pg + drizzle)  │  (better-sqlite3)  │
├─────────────────┼──────────────────┼────────────────────┤
│ Cache           │ Main DB (Prod)   │ Main DB (Dev)      │
│ Pub/Sub         │ Full SQL         │ Local File         │
│ Rate Limiting   │ Scalable         │ Zero Config        │
│ Sessions        │ JSONB, Arrays    │ Simple & Fast      │
└─────────────────┴──────────────────┴────────────────────┘
         ↓                  ↓                    ↓
   Development      Production Only      Development Only
   Production
```

### Quando Usar Cada Banco

#### 🔴 Redis
```yaml
Uso:
  - ✅ Cache de queries (reduz latência)
  - ✅ Pub/Sub de eventos (comunicação real-time)
  - ✅ Rate limiting (controle de abuso)
  - ✅ Session storage (JWT, autenticação)
  - ✅ Contadores e rankings

Ambientes: Development + Production

Não use para:
  - ❌ Dados permanentes críticos
  - ❌ Queries complexas (SQL)
  - ❌ Dados > 1GB por key
```

#### 🐘 PostgreSQL
```yaml
Uso:
  - ✅ Dados principais (users, guilds, moderation)
  - ✅ Features SQL avançadas (JSONB, CTEs, window functions)
  - ✅ Escalabilidade (milhões de registros)
  - ✅ Integridade referencial (foreign keys)
  - ✅ Full-text search
  - ✅ Transações complexas

Ambientes: Production ONLY

Não use para:
  - ❌ Development local (use SQLite)
  - ❌ Cache temporário (use Redis)
```

#### 📁 SQLite
```yaml
Uso:
  - ✅ Desenvolvimento local (zero setup)
  - ✅ Testes rápidos (reset fácil)
  - ✅ Protótipos e demos
  - ✅ Dados portáteis (arquivo único)

Ambientes: Development ONLY

Não use para:
  - ❌ Produção (sem escalabilidade)
  - ❌ Múltiplos writers concorrentes
  - ❌ Features avançadas SQL
```

---

## 🚀 Quick Start

### Development (SQLite + Redis)

```bash
# 1. Install dependencies
pnpm install

# 2. Start Redis (Docker)
docker run -d -p 6379:6379 redis:alpine

# 3. Run migrations (cria dev.db automaticamente)
pnpm db:migrate

# 4. Seed data (opcional)
pnpm db:seed

# 5. Start development
pnpm dev

# Database criado: packages/backend/dev.db
# Redis: localhost:6379
```

### Production (PostgreSQL + Redis)

```bash
# 1. Configure environment variables
DATABASE_URL=postgresql://user:pass@host:5432/db
DATABASE_PROVIDER=postgres
REDIS_HOST=redis.production.com
REDIS_PASSWORD=xxx

# 2. Run migrations
pnpm db:migrate

# 3. Start production
pnpm start
```

---

## 📊 Principais Tabelas (PostgreSQL/SQLite)

```
Core Tables:
├── users              - Usuários do sistema
├── guilds             - Servidores Discord
├── members            - Relação user ↔ guild

Moderation:
├── bans               - Banimentos
├── kicks              - Kicks
├── mutes              - Mutes
├── warnings           - Avisos
└── moderation_logs    - Logs de ações

Configuration:
├── guild_configs      - Configurações de guilds
├── policies           - Políticas de moderação
└── features           - Feature flags

Audit:
└── audit_logs         - Auditoria completa
```

---

## 💡 Trabalhando com os Bancos

### Drizzle ORM (PostgreSQL/SQLite)

```typescript
// Query
const users = await db.select().from(usersTable);

// Insert
await db.insert(usersTable).values({
  id: '123',
  name: 'John Doe'
});

// Update
await db.update(usersTable)
  .set({ name: 'Jane' })
  .where(eq(usersTable.id, '123'));
```

### Redis (Cache & Pub/Sub)

```typescript
// Cache
await redis.set('user:123', JSON.stringify(user), 'EX', 300);
const cached = await redis.get('user:123');

// Pub/Sub
await redis.publish('events', JSON.stringify(event));
redis.subscribe('events', (err, count) => {
  console.log(`Subscribed to ${count} channels`);
});
```

### Migrations

```bash
# Generate migration from schema changes
pnpm db:generate

# Apply migrations
pnpm db:migrate

# Reset database (development only)
pnpm db:reset
```

### Drizzle Studio (Visual Database UI)

```bash
pnpm db:studio
# Opens: https://local.drizzle.studio

# View:
# - All tables
# - Data
# - Relationships
# - Execute queries
```

---

## 🔍 Exemplo Completo

```typescript
import { db } from './db';
import { redis } from './redis';
import { users } from './schema';
import { eq } from 'drizzle-orm';

async function getUser(userId: string) {
  // 1. Try cache first (Redis)
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }

  // 2. Query database (PostgreSQL/SQLite via Drizzle)
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId),
    with: {
      guilds: true,
      members: true
    }
  });

  // 3. Cache for 5 minutes
  if (user) {
    await redis.setex(`user:${userId}`, 300, JSON.stringify(user));
  }

  return user;
}
```

---

## 🏗️ Arquitetura

```
┌────────────────────────────────────────┐
│         Application Layer              │
│  (Fastify, Discord Bot, Workers)       │
└───────────┬────────────────────────────┘
            │
    ┌───────┴────────┐
    │                │
    ↓                ↓
┌─────────┐    ┌──────────────┐
│  Redis  │    │ Drizzle ORM  │
│         │    │              │
│ ioredis │    └──────┬───────┘
│         │           │
└─────────┘      ┌────┴─────┐
                 │          │
                 ↓          ↓
           ┌──────────┐ ┌────────┐
           │PostgreSQL│ │ SQLite │
           │  (prod)  │ │ (dev)  │
           └──────────┘ └────────┘
```

---

## 🔗 Documentação Relacionada

- **[../04-backend/](../04-backend/)** - Backend integration
- **[../02-platforms/002-EVENTS.md](../02-platforms/002-EVENTS.md)** - Event system (Redis Pub/Sub)
- **[../08-api/](../08-api/)** - API endpoints
- **[../10-operations/](../10-operations/)** - Monitoring

---

## 📚 Recursos Externos

- [Drizzle ORM](https://orm.drizzle.team/)
- [PostgreSQL](https://www.postgresql.org/)
- [Redis](https://redis.io/)
- [better-sqlite3](https://github.com/WiseLibs/better-sqlite3)

---

**Três bancos, um sistema robusto!** 🗄️
