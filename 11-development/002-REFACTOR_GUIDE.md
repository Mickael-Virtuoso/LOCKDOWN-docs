# LOCKDOWN Refactor Guide

## 📋 Overview

Este guide detalha **EXATAMENTE** como refatorar LOCKDOWN de VANGUARD (monolith) para arquitetura limpa (monorepo com separação de concerns).

**Timeline:** 4 semanas (Fevereiro 5 - Março 4, 2026)

```
ANTES (VANGUARD - Monolith):
├─ Discord.js acoplado ao core
├─ Backend misturado com bot
├─ 396 diretórios de spaghetti
└─ Refatorar = pesadelo

DEPOIS (LOCKDOWN - Clean):
├─ Backend API puro (sem Discord.js)
├─ Bot consome API (cliente REST)
├─ ~50 diretórios bem organizados
└─ Escalável, testável, profissional
```

---

## 🎯 Objetivos Finais

```
✅ Backend completamente isolado de Discord.js
✅ API REST funcional com tipos TypeScript
✅ Bot consumindo API como cliente externo
✅ Database (PostgreSQL) centralizado
✅ Cache (Redis) implementado
✅ Logging padronizado com Pino
✅ Testes automatizados
✅ Deploy independente
✅ Documentação completa
```

---

# PHASE 1: Backend Isolado (Semana 1)

**Data:** Fevereiro 5-11  
**Objetivo:** Separar lógica de business do Discord.js

## 📌 Backlog

```
☐ Task 1.1: Setup PostgreSQL + Drizzle ORM
☐ Task 1.2: Extract database models
☐ Task 1.3: Create repository layer
☐ Task 1.4: Setup Redis connection
☐ Task 1.5: Create service layer (business logic)
☐ Task 1.6: Setup logging (Pino)
☐ Task 1.7: Create types/interfaces
```

## 1.1 Setup PostgreSQL + Drizzle ORM

### Objetivo
Ter banco de dados configurado com Drizzle ORM e migrations prontas.

### Tarefas

```bash
# 1. Instalar Drizzle + PostgreSQL driver
cd packages/backend
pnpm add drizzle-orm pg dotenv

# 2. Criar arquivo de configuração
# backend/src/config/database.ts
```

### Código

**backend/src/config/database.ts:**
```typescript
/**
 * - @file database.ts
 * 
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 * @editor Mickael Virtuoso
 *
 * =====================================================================
 * ====================== CODE LIFE CYCLE ==============================
 * =====================================================================
 *
 * @version 1.0.0
 * @since 2026-02-05
 * @creator Mickael Virtuoso
 *
 * =====================================================================
 * ====================== QUALITY CONTROL ==============================
 * =====================================================================
 *
 * @audit Mickael Virtuoso
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * Database connection configuration using Drizzle ORM and PostgreSQL
 *
 * =====================================================================
 * ====================== DEPENDENCIES ================================
 * =====================================================================
 *
 * @requires
 * - drizzle-orm
 * - pg
 * - dotenv
 *
 * =====================================================================
 */

import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL || 'postgresql://lockdown:lockdown@localhost:5432/lockdown',
});

export const db = drizzle(pool);

export default db;
```

**backend/.env.example:**
```env
DATABASE_URL=postgresql://lockdown:lockdown@localhost:5432/lockdown
REDIS_URL=redis://localhost:6379
NODE_ENV=development
LOG_LEVEL=debug
```

### Docker Compose

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: lockdown
      POSTGRES_PASSWORD: lockdown
      POSTGRES_DB: lockdown
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U lockdown"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
  redis_data:
```

### Validação

```bash
# 1. Starta Docker
docker-compose up -d

# 2. Testa conexão
psql -U lockdown -d lockdown -h localhost

# 3. Verifica Redis
redis-cli ping
# Resultado: PONG

# Commits
git add packages/backend/src/config/database.ts .env.example docker-compose.yml
git commit -m "chore(backend): setup PostgreSQL and Drizzle ORM configuration"
```

---

## 1.2 Extract Database Models

### Objetivo
Converter Mongoose schemas em Drizzle schema definitions.

### Análise VANGUARD

De `backend/src/core/database/mongodb/models/moderation/ban.ts`:
```typescript
// OLD (Mongoose)
const banSchema = new Schema({
  userId: String,
  guildId: String,
  reason: String,
  timestamp: Date,
  moderatorId: String,
  evidenceHash: String,
});
```

### Novo com Drizzle

**backend/src/core/database/schema.ts:**
```typescript
/**
 * - @file schema.ts
 *
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 * @editor Mickael Virtuoso
 *
 * =====================================================================
 * ====================== CODE LIFE CYCLE ==============================
 * =====================================================================
 *
 * @version 1.0.0
 * @since 2026-02-05
 * @creator Mickael Virtuoso
 *
 * =====================================================================
 * ====================== QUALITY CONTROL ==============================
 * =====================================================================
 *
 * @audit Mickael Virtuoso
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * Drizzle ORM schema definitions for all database tables
 *
 * =====================================================================
 */

import { pgTable, text, timestamp, integer, boolean, jsonb } from 'drizzle-orm/pg-core';

// ============= USERS TABLE =============
export const users = pgTable('users', {
  id: text('id').primaryKey(), // Discord user ID
  username: text('username').notNull(),
  email: text('email'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  isVerified: boolean('is_verified').default(false),
});

// ============= GUILDS TABLE =============
export const guilds = pgTable('guilds', {
  id: text('id').primaryKey(), // Discord guild ID
  name: text('name').notNull(),
  icon: text('icon'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// ============= BANS TABLE =============
export const bans = pgTable('bans', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  guildId: text('guild_id').notNull(),
  moderatorId: text('moderator_id').notNull(),
  reason: text('reason').notNull(), // REQUIRED (min 10 chars in app)
  evidenceHash: text('evidence_hash'),
  isCrossServer: boolean('is_cross_server').default(false),
  bannedAt: timestamp('banned_at').defaultNow().notNull(),
  expireAt: timestamp('expire_at'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// ============= WARNINGS TABLE =============
export const warnings = pgTable('warnings', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  guildId: text('guild_id').notNull(),
  moderatorId: text('moderator_id').notNull(),
  reason: text('reason').notNull(),
  count: integer('count').default(1),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// ============= MUTES TABLE =============
export const mutes = pgTable('mutes', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  guildId: text('guild_id').notNull(),
  moderatorId: text('moderator_id').notNull(),
  reason: text('reason').notNull(),
  durationMs: integer('duration_ms'),
  mutedAt: timestamp('muted_at').defaultNow().notNull(),
  expireAt: timestamp('expire_at'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// ============= AUDIT LOGS TABLE =============
export const auditLogs = pgTable('audit_logs', {
  id: text('id').primaryKey(),
  guildId: text('guild_id').notNull(),
  userId: text('user_id'),
  moderatorId: text('moderator_id'),
  action: text('action').notNull(), // ban, kick, warn, mute, etc
  targetId: text('target_id'), // User or channel ID
  reason: text('reason'),
  metadata: jsonb('metadata'), // Extra data
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// ============= WHITELISTS TABLE =============
export const whitelists = pgTable('whitelists', {
  id: text('id').primaryKey(),
  guildId: text('guild_id').notNull(),
  userId: text('user_id').notNull(),
  reason: text('reason'),
  addedBy: text('added_by').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// ============= BLACKLISTS TABLE =============
export const blacklists = pgTable('blacklists', {
  id: text('id').primaryKey(),
  guildId: text('guild_id').notNull(),
  userId: text('user_id').notNull(),
  reason: text('reason'),
  addedBy: text('added_by').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// ============= SNAPSHOTS TABLE =============
export const snapshots = pgTable('snapshots', {
  id: text('id').primaryKey(),
  guildId: text('guild_id').notNull(),
  type: text('type').notNull(), // auto, manual
  data: jsonb('data').notNull(), // Channels, roles, permissions, etc
  createdBy: text('created_by'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

### Commits

```bash
git add packages/backend/src/core/database/schema.ts
git commit -m "feat(database): create Drizzle ORM schema definitions"
```

---

## 1.3 Create Repository Layer

### Objetivo
CRUD abstraction layer entre business logic e database.

**backend/src/core/database/repositories/BanRepository.ts:**
```typescript
/**
 * - @file BanRepository.ts
 *
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 * @editor Mickael Virtuoso
 *
 * =====================================================================
 * ====================== CODE LIFE CYCLE ==============================
 * =====================================================================
 *
 * @version 1.0.0
 * @since 2026-02-05
 * @creator Mickael Virtuoso
 *
 * =====================================================================
 * ====================== QUALITY CONTROL ==============================
 * =====================================================================
 *
 * @audit Mickael Virtuoso
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * Repository for ban data access operations
 *
 * =====================================================================
 * ====================== DEPENDENCIES ================================
 * =====================================================================
 *
 * @requires
 * - drizzle-orm
 * - database schema
 *
 * =====================================================================
 */

import { eq, and } from 'drizzle-orm';
import db from '../../config/database';
import { bans } from '../../database/schema';
import type { Ban, BanCreateInput } from '@lockdown/shared';

export default class BanRepository {
  /**
   * Create a new ban
   * @param ban - Ban data to create
   * @returns Created ban record
   */
  async create(ban: BanCreateInput): Promise<Ban> {
    const [result] = await db
      .insert(bans)
      .values({
        id: crypto.randomUUID(),
        ...ban,
      })
      .returning();

    logger.info(`[${this.constructor.name}] "✅ Ban created for user ${ban.userId}"`);
    return result as Ban;
  }

  /**
   * Find ban by ID
   * @param id - Ban ID
   * @returns Ban record or null
   */
  async findById(id: string): Promise<Ban | null> {
    const [result] = await db
      .select()
      .from(bans)
      .where(eq(bans.id, id));

    return (result as Ban) || null;
  }

  /**
   * Find bans by user and guild
   * @param userId - Discord user ID
   * @param guildId - Discord guild ID
   * @returns Array of bans
   */
  async findByUserAndGuild(userId: string, guildId: string): Promise<Ban[]> {
    const results = await db
      .select()
      .from(bans)
      .where(
        and(
          eq(bans.userId, userId),
          eq(bans.guildId, guildId)
        )
      );

    return results as Ban[];
  }

  /**
   * Remove a ban
   * @param id - Ban ID
   * @returns Success boolean
   */
  async remove(id: string): Promise<boolean> {
    const result = await db
      .delete(bans)
      .where(eq(bans.id, id));

    logger.info(`[${this.constructor.name}] "🗑️ Ban removed: ${id}"`);
    return !!result;
  }
}
```

### Commits

```bash
git add packages/backend/src/core/database/repositories/
git commit -m "feat(database): create repository layer for CRUD operations"
```

---

## 1.4 Setup Redis Connection

**backend/src/config/redis.ts:**
```typescript
/**
 * - @file redis.ts
 *
 * @description Redis cache connection configuration
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 */

import { createClient } from 'redis';

const redis = createClient({
  url: process.env.REDIS_URL || 'redis://localhost:6379',
});

redis.on('error', (err) => logger.error(`[Redis] "❌ Connection error: ${err}"`));
redis.on('connect', () => logger.info(`[Redis] "✅ Connected to Redis"`));

await redis.connect();

export default redis;
```

### Commits

```bash
git add packages/backend/src/config/redis.ts
git commit -m "chore(backend): setup Redis cache configuration"
```

---

## 1.5 Create Service Layer

**backend/src/core/services/BanService.ts:**
```typescript
/**
 * - @file BanService.ts
 *
 * @description Business logic for ban operations
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 */

import BanRepository from '../database/repositories/BanRepository';
import type { Ban, BanCreateInput } from '@lockdown/shared';

export default class BanService {
  private banRepository: BanRepository;

  constructor() {
    this.banRepository = new BanRepository();
  }

  /**
   * Execute a ban with validation
   * @param banData - Ban information
   * @returns Created ban
   * @throws Error if validation fails
   */
  async executeBan(banData: BanCreateInput): Promise<Ban> {
    if (!banData.reason || banData.reason.length < 10) {
      throw new Error('Ban reason must be at least 10 characters');
    }

    const ban = await this.banRepository.create(banData);

    logger.info(`[${this.constructor.name}] "🔴 Ban executed: ${ban.id}"`);
    return ban;
  }

  /**
   * Get user's ban history
   * @param userId - Discord user ID
   * @param guildId - Discord guild ID
   * @returns Array of bans
   */
  async getUserBanHistory(userId: string, guildId: string): Promise<Ban[]> {
    const bans = await this.banRepository.findByUserAndGuild(userId, guildId);
    logger.info(`[${this.constructor.name}] "📋 Retrieved ${bans.length} bans for user"`);
    return bans;
  }
}
```

### Commits

```bash
git add packages/backend/src/core/services/
git commit -m "feat(backend): create service layer with business logic"
```

---

## 1.6 Setup Logging

**backend/src/config/logger.ts:**
```typescript
/**
 * - @file logger.ts
 *
 * @description Pino logger configuration
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 */

import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true,
      singleLine: false,
      ignore: 'pid,hostname',
    },
  },
});

export default logger;
```

### Commits

```bash
git add packages/backend/src/config/logger.ts
git commit -m "chore(backend): setup Pino logger configuration"
```

---

## 1.7 Create Shared Types

**packages/shared/src/types/moderation.ts:**
```typescript
/**
 * - @file moderation.ts
 *
 * @description Type definitions for moderation operations
 *
 * @author Mickael Virtuoso
 * @since 2026-02-05
 */

export interface Ban {
  id: string;
  userId: string;
  guildId: string;
  moderatorId: string;
  reason: string;
  evidenceHash?: string;
  isCrossServer: boolean;
  bannedAt: Date;
  expireAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}

export interface BanCreateInput {
  userId: string;
  guildId: string;
  moderatorId: string;
  reason: string;
  evidenceHash?: string;
  isCrossServer?: boolean;
  expireAt?: Date;
}

export interface BanResponse {
  success: boolean;
  ban?: Ban;
  error?: string;
}

export interface Warning {
  id: string;
  userId: string;
  guildId: string;
  moderatorId: string;
  reason: string;
  count: number;
  createdAt: Date;
}

export interface Mute {
  id: string;
  userId: string;
  guildId: string;
  moderatorId: string;
  reason: string;
  durationMs?: number;
  mutedAt: Date;
  expireAt?: Date;
  createdAt: Date;
}
```

**packages/shared/src/types/index.ts:**
```typescript
export * from './moderation';
export * from './auth';
export * from './snapshots';
```

### Commits

```bash
git add packages/shared/src/types/
git commit -m "feat(shared): create type definitions for moderation"
```

---

## ✅ Phase 1 Validation Checklist

```
☑️ PostgreSQL rodando (docker ps)
☑️ Redis rodando (redis-cli ping)
☑️ Database schema criado
☑️ Repositories implementados
☑️ Services implementados
☑️ Shared types prontos
☑️ Logger funcionando
☑️ Sem imports de Discord.js em core/
☑️ TypeScript strict mode: tudo compilando
☑️ Todos os commits feitos
```

### Teste

```bash
# Compilar TypeScript
pnpm type-check

# Build
pnpm build

# Se passar: Phase 1 ✅
```

---

# PHASE 2: API Skeleton (Semana 2)

**Data:** Fevereiro 12-18  
**Objetivo:** Criar endpoints REST que consumem o backend isolado

## 📌 Backlog

```
☐ Task 2.1: Setup Express/Fastify
☐ Task 2.2: Create authentication middleware
☐ Task 2.3: Create error handling middleware
☐ Task 2.4: Create API routes skeleton
☐ Task 2.5: Implement request/response validation
☐ Task 2.6: Setup API documentation (Swagger)
☐ Task 2.7: Create health check endpoint
```

## 2.1 Setup Express

**backend/src/config/express.ts:**
```typescript
/**
 * - @file express.ts
 *
 * @description Express application setup
 *
 * @author Mickael Virtuoso
 * @since 2026-02-12
 */

import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import logger from './logger';

const app = express();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());

// Logging middleware
app.use((req, res, next) => {
  logger.info(`[Express] "${req.method} ${req.path}"`);
  next();
});

export default app;
```

**backend/src/main.ts:**
```typescript
/**
 * - @file main.ts
 *
 * @description Application entry point
 *
 * @author Mickael Virtuoso
 * @since 2026-02-12
 */

import app from './config/express';
import logger from './config/logger';

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  logger.info(`[Server] "✅ Listening on port ${PORT}"`);
});
```

### Commits

```bash
git add packages/backend/src/config/express.ts packages/backend/src/main.ts
git commit -m "feat(api): setup Express server configuration"
```

---

## 2.2 Create API Routes

**backend/src/api/routes/bans.ts:**
```typescript
/**
 * - @file bans.ts
 *
 * @description Ban management API routes
 *
 * @author Mickael Virtuoso
 * @since 2026-02-12
 */

import { Router } from 'express';
import BanService from '../../core/services/BanService';
import type { BanCreateInput, BanResponse } from '@lockdown/shared';

const router = Router();
const banService = new BanService();

/**
 * POST /api/v1/bans
 * Create a new ban
 */
router.post('/', async (req, res) => {
  try {
    const banData: BanCreateInput = req.body;
    const ban = await banService.executeBan(banData);

    const response: BanResponse = {
      success: true,
      ban,
    };

    res.status(201).json(response);
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
    });
  }
});

/**
 * GET /api/v1/bans/:userId/:guildId
 * Get user's ban history
 */
router.get('/:userId/:guildId', async (req, res) => {
  try {
    const { userId, guildId } = req.params;
    const bans = await banService.getUserBanHistory(userId, guildId);

    res.json({
      success: true,
      bans,
    });
  } catch (error) {
    res.status(400).json({
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
    });
  }
});

export default router;
```

**backend/src/api/index.ts:**
```typescript
/**
 * - @file index.ts
 *
 * @description API routes aggregator
 *
 * @author Mickael Virtuoso
 * @since 2026-02-12
 */

import { Router } from 'express';
import bansRouter from './routes/bans';

const router = Router();

router.use('/bans', bansRouter);

export default router;
```

**backend/src/config/express.ts (updated):**
```typescript
import app from 'express';
import apiRouter from '../api';

app.use('/api/v1', apiRouter);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});
```

### Commits

```bash
git add packages/backend/src/api/
git commit -m "feat(api): create ban management API routes"
```

---

## ✅ Phase 2 Validation

```
☑️ Express server inicia sem erros
☑️ Health check: GET /health → 200 OK
☑️ Ban endpoint: POST /api/v1/bans → 201 Created
☑️ Ban history: GET /api/v1/bans/:userId/:guildId → 200 OK
☑️ Error handling funciona
☑️ Tipos TypeScript para requests/responses
```

### Teste

```bash
pnpm dev

# Em outro terminal
curl http://localhost:3000/health
# {"status": "ok"}
```

---

# PHASE 3: Core Endpoints (Semana 3)

**Data:** Fevereiro 19-25  
**Objetivo:** Implementar todos os endpoints principais

## 📌 Endpoints

```
POST   /api/v1/bans                 - Create ban
GET    /api/v1/bans/:userId/:guildId - Get user bans
DELETE /api/v1/bans/:banId          - Remove ban

POST   /api/v1/warns                - Create warning
GET    /api/v1/warns/:userId/:guildId - Get user warnings
DELETE /api/v1/warns/:warnId        - Remove warning

POST   /api/v1/mutes                - Create mute
GET    /api/v1/mutes/:userId/:guildId - Get user mutes
DELETE /api/v1/mutes/:muteId        - Remove mute

GET    /api/v1/audit-logs           - Get audit logs
GET    /api/v1/audit-logs/:guildId  - Get guild audit logs

POST   /api/v1/snapshots            - Create snapshot
GET    /api/v1/snapshots/:guildId   - Get snapshots
POST   /api/v1/snapshots/:id/restore - Restore snapshot
```

### Implementação

Seguir mesmo padrão de bans.ts para:
- warnings.ts
- mutes.ts
- audit-logs.ts
- snapshots.ts

### Commits

```bash
git add packages/backend/src/api/routes/
git commit -m "feat(api): implement all core endpoints (warns, mutes, audit-logs, snapshots)"
```

---

# PHASE 4: Bot Consumer (Semana 4)

**Data:** Fevereiro 26 - Março 4  
**Objetivo:** Discord bot consumindo API

## 📌 Backlog

```
☐ Task 4.1: Setup Discord.js client
☐ Task 4.2: Create API HTTP client
☐ Task 4.3: Implement ban command
☐ Task 4.4: Implement kick command
☐ Task 4.5: Implement warn command
☐ Task 4.6: Implement mute command
☐ Task 4.7: Add event listeners (ban/kick from Discord)
```

## 4.1 Bot API Client

**lockdown-bot/src/core/ApiClient.ts:**
```typescript
/**
 * - @file ApiClient.ts
 *
 * @description HTTP client for backend API communication
 *
 * @author Mickael Virtuoso
 * @since 2026-02-26
 */

import type { Ban, BanResponse, BanCreateInput } from '@lockdown/shared';

export default class ApiClient {
  private baseUrl: string;

  constructor(baseUrl: string = process.env.API_URL || 'http://localhost:3000') {
    this.baseUrl = baseUrl;
  }

  /**
   * Create a ban via API
   * @param banData - Ban information
   * @returns Ban response
   */
  async createBan(banData: BanCreateInput): Promise<BanResponse> {
    const response = await fetch(`${this.baseUrl}/api/v1/bans`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(banData),
    });

    return response.json();
  }

  /**
   * Get user's ban history
   * @param userId - Discord user ID
   * @param guildId - Discord guild ID
   * @returns Array of bans
   */
  async getUserBans(userId: string, guildId: string): Promise<Ban[]> {
    const response = await fetch(`${this.baseUrl}/api/v1/bans/${userId}/${guildId}`);
    const data = await response.json();
    return data.bans || [];
  }
}
```

## 4.2 Ban Command

**lockdown-bot/src/commands/ban.ts:**
```typescript
/**
 * - @file ban.ts
 *
 * @description Ban command for Discord
 *
 * @author Mickael Virtuoso
 * @since 2026-02-26
 */

import {
  SlashCommandBuilder,
  type CommandInteraction,
  PermissionFlagsBits,
} from 'discord.js';
import ApiClient from '../core/ApiClient';

const apiClient = new ApiClient();

export default {
  data: new SlashCommandBuilder()
    .setName('ban')
    .setDescription('Ban a user from the server')
    .addUserOption((option) =>
      option
        .setName('user')
        .setDescription('User to ban')
        .setRequired(true)
    )
    .addStringOption((option) =>
      option
        .setName('reason')
        .setDescription('Reason for ban (min 10 characters)')
        .setRequired(true)
    )
    .setDefaultMemberPermissions(PermissionFlagsBits.BanMembers),

  async execute(interaction: CommandInteraction): Promise<void> {
    const targetUser = interaction.options.getUser('user', true);
    const reason = interaction.options.getString('reason', true);

    // Validate reason
    if (reason.length < 10) {
      await interaction.reply({
        content: '❌ Ban reason must be at least 10 characters.',
        ephemeral: true,
      });
      return;
    }

    try {
      // Call API
      const response = await apiClient.createBan({
        userId: targetUser.id,
        guildId: interaction.guildId!,
        moderatorId: interaction.user.id,
        reason,
      });

      if (!response.success) {
        throw new Error(response.error || 'Failed to create ban');
      }

      // Ban on Discord
      await interaction.guild!.members.ban(targetUser, { reason });

      await interaction.reply({
        content: `✅ **${targetUser.username}** has been banned.\n**Reason:** ${reason}`,
      });

      logger.info(`[BanCommand] "🔴 User ${targetUser.id} banned by ${interaction.user.id}"`);
    } catch (error) {
      logger.error(`[BanCommand] "❌ Error: ${error}"`);
      await interaction.reply({
        content: '❌ Failed to ban user.',
        ephemeral: true,
      });
    }
  },
};
```

### Commits

```bash
git add lockdown-bot/src/core/ApiClient.ts
git commit -m "feat(bot): create API HTTP client for backend communication"

git add lockdown-bot/src/commands/ban.ts
git commit -m "feat(bot): implement ban command using backend API"
```

---

## ✅ Phase 4 Validation

```
☑️ Bot inicia sem erros
☑️ /ban command funciona
☑️ API é chamada corretamente
☑️ Ban aparece no database
☑️ User é banido no Discord
☑️ Reason é armazenado
☑️ Logging funciona
☑️ Error handling funciona
```

---

# 📊 Timeline Summary

```
SEMANA 1 (5-11 Fev):  Database + Repositories + Services
SEMANA 2 (12-18 Fev): Express + API routes skeleton
SEMANA 3 (19-25 Fev): Core endpoints (bans, kicks, warns, mutes, audit, snapshots)
SEMANA 4 (26 Fev-4 Mar): Bot consumer + Commands

MARÇO-ABRIL:
├─ Testing + Debugging
├─ Truth Confiability implementation
├─ Snapshots advanced features
├─ Dashboard integration
└─ Legal validation

MAIO-JUNHO:
├─ Security review
├─ Performance optimization
├─ Documentation
└─ Pre-launch

OUTUBRO: LAUNCH! 🚀
```

---

# 🐛 Troubleshooting

## PostgreSQL connection fails

```bash
# Check if running
docker ps

# Check logs
docker logs <postgres-container-id>

# Reconnect
docker-compose restart postgres
```

## Redis connection fails

```bash
# Test Redis
redis-cli ping

# Restart
docker-compose restart redis
```

## TypeScript compilation errors

```bash
# Check strict mode issues
pnpm type-check

# Fix imports
# ✅ import type { Ban } from '@lockdown/shared'
# ❌ import { Ban } from '@lockdown/shared'
```

## API endpoint returns 404

```bash
# Check Express setup
# Verify route registration in backend/src/config/express.ts
# Check if /api/v1 prefix is correct
```

---

# 📚 Resources

- [Drizzle ORM Docs](https://orm.drizzle.team/)
- [Express Guide](https://expressjs.com/)
- [Discord.js Guide](https://discordjs.guide/)
- [TypeScript Strict Mode](https://www.typescriptlang.org/tsconfig#strict)
- [Pino Logger](https://getpino.io/)

---

**Você consegue! Essa refatoração vai ser TRANSFORMADORA pra LOCKDOWN!** 🚀

Dúvidas em qualquer phase? Me chama! 💪
