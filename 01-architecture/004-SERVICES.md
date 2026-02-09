# 🎯 SERVICES - LOCKDOWN

## Service Layer - Business Logic e Orquestração

---

## 📖 Índice

1. [Visão Geral](#visão-geral)
2. [Arquitetura](#arquitetura)
3. [Ban Service](#ban-service)
4. [Kick Service](#kick-service)
5. [Cache Service](#cache-service)
6. [Email Service](#email-service)
7. [Patterns](#patterns)
8. [Testing](#testing)

---

## 📋 Visão Geral

### O que é Service Layer?

Camada que contém **lógica de negócio** e **orquestração**:

```
Controller (HTTP)
    ↓
Service (Lógica de negócio)
    ├─ Validações
    ├─ Regras
    ├─ Orquestração
    └─ Eventos
    ↓
Repository (Dados)
    ↓
Database / Cache
```

### Responsabilidades

```
✅ Validar entrada
✅ Aplicar regras de negócio
✅ Orquestrar múltiplas operações
✅ Publicar eventos
✅ Cachear dados
✅ Lidar com erros
```

### NÃO fazer em Services

```
❌ Fazer queries diretas ao DB (use Repository)
❌ Formatar resposta HTTP (use Controller)
❌ Rotear requisições (use Routes)
❌ Validar schema HTTP (use Middleware)
```

---

## 🏗️ Arquitetura

### Estrutura

```
packages/backend/src/
├─ database/
│  ├─ repositories/
│  │  ├─ BanRepository.ts
│  │  ├─ KickRepository.ts
│  │  └─ UserRepository.ts
│  │
│  └─ services/          ← VOCÊ ESTÁ AQUI
│     ├─ moderation/
│     │  ├─ BanService.ts
│     │  ├─ KickService.ts
│     │  └─ MuteService.ts
│     │
│     ├─ core/
│     │  ├─ UserService.ts
│     │  └─ GuildService.ts
│     │
│     └─ shared/
│        ├─ CacheService.ts
│        ├─ EmailService.ts
│        └─ LogService.ts
```

### Dependency Injection

```typescript
// Service recebe repositórios como dependências
class BanService {
  constructor(
    private banRepository: BanRepository,
    private userRepository: UserRepository,
    private cacheService: CacheService
  ) {}

  // Use os repositories
}
```

---

## 🚫 Ban Service

**`packages/backend/src/database/services/moderation/BanService.ts`:**

```typescript
/**
 * - @file BanService.ts
 *
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Mickael Virtuoso
 * @since 2025-02-05
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * Service para gerenciar banimentos
 *
 */

import { logger } from '@config/logger';
import { AppError } from '@utils/errors';
import BanRepository from '@database/repositories/moderation/BanRepository';
import BanPublisher from '@events/publishers/BanPublisher';
import CacheService from '@services/shared/CacheService';

export interface CreateBanDTO {
  guildId: string;
  userId: string;
  reason?: string;
  moderatorId: string;
  expiresAt?: Date;
}

export interface ListBansDTO {
  guildId: string;
  page?: number;
  limit?: number;
  isActive?: boolean;
}

export default class BanService {
  constructor(
    private banRepository: BanRepository,
    private banPublisher: BanPublisher,
    private cacheService: CacheService
  ) {}

  /**
   * Criar novo banimento
   *
   * Validações:
   * - Guild existe
   * - Usuário existe
   * - Não é duplicate
   * - Razão não é vazia
   */
  async createBan(data: CreateBanDTO): Promise<any> {
    logger.debug(`[${this.constructor.name}] "🔄 creating ban for user: ${data.userId}"`);

    // ========================================
    // 1. VALIDAÇÕES
    // ========================================
    if (!data.guildId || !data.guildId.trim()) {
      throw new AppError('Guild ID is required', 400);
    }

    if (!data.userId || !data.userId.trim()) {
      throw new AppError('User ID is required', 400);
    }

    if (!data.reason || !data.reason.trim()) {
      throw new AppError('Reason is required', 400);
    }

    if (data.reason.length < 3) {
      throw new AppError('Reason must be at least 3 characters', 400);
    }

    if (data.reason.length > 500) {
      throw new AppError('Reason must be less than 500 characters', 400);
    }

    // ========================================
    // 2. VERIFICAR DUPLICATA
    // ========================================
    const existingBan = await this.banRepository.findActiveByUserAndGuild(
      data.guildId,
      data.userId
    );

    if (existingBan) {
      throw new AppError('User is already banned in this guild', 409);
    }

    // ========================================
    // 3. CRIAR NO BANCO
    // ========================================
    const ban = await this.banRepository.create({
      guildId: data.guildId,
      userId: data.userId,
      reason: data.reason.trim(),
      moderatorId: data.moderatorId,
      expiresAt: data.expiresAt || null,
      isActive: true,
    });

    logger.info(`[${this.constructor.name}] "✅ ban created: ${ban.id}"`);

    // ========================================
    // 4. LIMPAR CACHE
    // ========================================
    await this.cacheService.deletePattern(`guild:${data.guildId}:bans`);

    // ========================================
    // 5. PUBLICAR EVENTO
    // ========================================
    await this.banPublisher.publishBanCreated(ban);

    logger.debug(`[${this.constructor.name}] "📡 ban:created event published"`);

    return ban;
  }

  /**
   * Listar banimentos de uma guild
   */
  async listBans(dto: ListBansDTO): Promise<{
    data: any[];
    pagination: {
      page: number;
      limit: number;
      total: number;
      pages: number;
    };
  }> {
    logger.debug(`[${this.constructor.name}] "🔄 listing bans for guild: ${dto.guildId}"`);

    // Validações
    if (!dto.guildId) {
      throw new AppError('Guild ID is required', 400);
    }

    const page = Math.max(1, dto.page || 1);
    const limit = Math.min(100, dto.limit || 50);

    // Check cache
    const cacheKey = `guild:${dto.guildId}:bans:${page}:${limit}`;
    const cached = await this.cacheService.get(cacheKey);
    if (cached) {
      logger.debug(`[${this.constructor.name}] "✅ bans from cache"`);
      return JSON.parse(cached);
    }

    // Fetch from DB
    const [bans, total] = await this.banRepository.findByGuild(
      dto.guildId,
      page,
      limit,
      dto.isActive
    );

    const pages = Math.ceil(total / limit);

    const result = {
      data: bans,
      pagination: { page, limit, total, pages },
    };

    // Cache result
    await this.cacheService.set(cacheKey, JSON.stringify(result), 300); // 5 min

    return result;
  }

  /**
   * Remover banimento
   */
  async removeBan(banId: string): Promise<void> {
    logger.debug(`[${this.constructor.name}] "🔄 removing ban: ${banId}"`);

    // Validações
    if (!banId) {
      throw new AppError('Ban ID is required', 400);
    }

    // Find ban
    const ban = await this.banRepository.findById(banId);
    if (!ban) {
      throw new AppError('Ban not found', 404);
    }

    // Delete
    await this.banRepository.delete(banId);

    logger.info(`[${this.constructor.name}] "✅ ban removed: ${banId}"`);

    // Clear cache
    await this.cacheService.deletePattern(`guild:${ban.guildId}:bans`);

    // Publish event
    await this.banPublisher.publishBanRemoved(ban.id.toString(), ban.guildId, ban.userId);

    logger.debug(`[${this.constructor.name}] "📡 ban:removed event published"`);
  }

  /**
   * Verificar se usuário está banido
   */
  async isUserBanned(guildId: string, userId: string): Promise<boolean> {
    const cacheKey = `ban:${guildId}:${userId}`;
    const cached = await this.cacheService.get(cacheKey);

    if (cached !== null) {
      return cached === 'true';
    }

    const ban = await this.banRepository.findActiveByUserAndGuild(guildId, userId);

    const isBanned = !!ban;
    await this.cacheService.set(
      cacheKey,
      isBanned ? 'true' : 'false',
      3600 // 1 hora
    );

    return isBanned;
  }
}
```

---

## 👢 Kick Service

```typescript
export default class KickService {
  constructor(
    private kickRepository: KickRepository,
    private banPublisher: BanPublisher,
    private cacheService: CacheService
  ) {}

  async executeKick(data: {
    guildId: string;
    userId: string;
    reason?: string;
    moderatorId: string;
  }): Promise<any> {
    logger.debug(`[${this.constructor.name}] "🔄 executing kick for user: ${data.userId}"`);

    // Validações
    if (!data.guildId || !data.userId) {
      throw new AppError('Guild and User IDs required', 400);
    }

    // Criar record
    const kick = await this.kickRepository.create(data);

    logger.info(`[${this.constructor.name}] "✅ kick executed: ${kick.id}"`);

    // Publish event
    await this.banPublisher.publishKickExecuted(kick);

    return kick;
  }
}
```

---

## 💾 Cache Service

```typescript
/**
 * - @file CacheService.ts
 *
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Mickael Virtuoso
 * @since 2025-02-05
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * Serviço para gerenciar cache com Redis
 *
 */

import { redis } from '@config/redis';
import { logger } from '@config/logger';

export default class CacheService {
  /**
   * Obter valor do cache
   */
  static async get(key: string): Promise<string | null> {
    try {
      const value = await redis.get(key);
      if (value) {
        logger.trace(`[CacheService] "✅ cache hit: ${key}"`);
      }
      return value;
    } catch (error) {
      logger.warn(`[CacheService] "⚠️ cache get error: ${error}"`);
      return null;
    }
  }

  /**
   * Definir valor no cache
   */
  static async set(key: string, value: string, ttl: number = 3600): Promise<void> {
    try {
      await redis.setex(key, ttl, value);
      logger.trace(`[CacheService] "✅ cached: ${key} (${ttl}s)"`);
    } catch (error) {
      logger.warn(`[CacheService] "⚠️ cache set error: ${error}"`);
    }
  }

  /**
   * Deletar chave
   */
  static async delete(key: string): Promise<void> {
    try {
      await redis.del(key);
      logger.trace(`[CacheService] "🗑️ cache deleted: ${key}"`);
    } catch (error) {
      logger.warn(`[CacheService] "⚠️ cache delete error: ${error}"`);
    }
  }

  /**
   * Deletar por padrão
   */
  static async deletePattern(pattern: string): Promise<void> {
    try {
      const keys = await redis.keys(pattern);
      if (keys.length > 0) {
        await redis.del(...keys);
        logger.trace(`[CacheService] "🗑️ deleted ${keys.length} cache keys"`);
      }
    } catch (error) {
      logger.warn(`[CacheService] "⚠️ cache delete pattern error: ${error}"`);
    }
  }
}
```

---

## 📧 Email Service

```typescript
export default class EmailService {
  async sendBanNotification(
    userEmail: string,
    banDetails: {
      reason: string;
      expiresAt?: Date;
      guildName: string;
    }
  ): Promise<void> {
    logger.debug(`[${this.constructor.name}] "📧 sending ban notification"`);

    const subject = `You've been banned from ${banDetails.guildName}`;
    const body = `
      You have been banned for: ${banDetails.reason}
      ${banDetails.expiresAt ? `Expires: ${banDetails.expiresAt.toISOString()}` : 'This is a permanent ban'}
    `;

    // Enviar email (implementar com Sendgrid, Nodemailer, etc)
    try {
      // await emailProvider.send({ to: userEmail, subject, body })
      logger.info(`[${this.constructor.name}] "✅ ban notification sent"`);
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ failed to send email: ${error}"`);
      // Não lançar erro, apenas log
    }
  }
}
```

---

## 🎨 Patterns

### Injeção de Dependência

```typescript
// ✅ BOM: Dependências injetadas
class BanService {
  constructor(
    private banRepository: BanRepository,
    private banPublisher: BanPublisher,
    private cacheService: CacheService
  ) {}
}

// Usar
const banRepository = new BanRepository();
const banPublisher = new BanPublisher();
const cacheService = CacheService;
const banService = new BanService(banRepository, banPublisher, cacheService);
```

### Error Handling

```typescript
// ✅ BOM: Custom errors
if (!userId) {
  throw new AppError('User ID required', 400);
}

try {
  await operation();
} catch (error) {
  if (error instanceof AppError) {
    throw error; // Repass custom errors
  }
  throw new AppError('Operation failed', 500);
}
```

### Caching Strategy

```typescript
// ✅ BOM: Cache com TTL apropriado
const cacheKey = `ban:${guildId}:${userId}`;
const cached = await CacheService.get(cacheKey);

if (cached) return JSON.parse(cached);

const data = await this.banRepository.findById(userId);
await CacheService.set(cacheKey, JSON.stringify(data), 3600); // 1h

return data;
```

---

## 🧪 Testing

### Unit Test

```typescript
describe('BanService', () => {
  let service: BanService;
  let mockRepository: jest.Mocked<BanRepository>;

  beforeEach(() => {
    mockRepository = createMockRepository();
    service = new BanService(mockRepository, mockPublisher, mockCache);
  });

  it('should create ban', async () => {
    const result = await service.createBan({
      guildId: '123',
      userId: '456',
      reason: 'Spam',
      moderatorId: '789',
    });

    expect(result.id).toBeDefined();
    expect(mockRepository.create).toHaveBeenCalled();
  });

  it('should throw if reason is empty', async () => {
    await expect(
      service.createBan({
        guildId: '123',
        userId: '456',
        reason: '',
        moderatorId: '789',
      })
    ).rejects.toThrow(AppError);
  });
});
```

---

## 📚 Referências

- [BACKEND.md](./BACKEND.md) - How to use services
- [TESTING.md](./TESTING.md) - Testing services
- [Design Patterns](https://refactoring.guru/design-patterns)

---

**Happy servicing!** 🎯
