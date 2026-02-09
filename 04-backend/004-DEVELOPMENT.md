# Backend - Desenvolvimento

## Criar Novo Endpoint

Exemplo: Endpoint para listar todos os bans.

---

### Passo 1: Definir Tipo

`types/api.types.ts`

```typescript
export interface BanDTO {
  id: string;
  guildId: string;
  userId: string;
  reason: string;
  expiresAt: Date | null;
  createdAt: Date;
}

export interface ListBansResponse {
  data: BanDTO[];
  total: number;
  page: number;
  pageSize: number;
}
```

---

### Passo 2: Criar Repository

`database/repositories/moderation/BanRepository.ts`

```typescript
import type { SelectBan } from '@database/schemas/moderation';
import { bansTable } from '@database/schemas/moderation';
import { db } from '@database/client';
import { logger } from '@config/logger';

export interface CreateBanDTO {
  guildId: string;
  userId: string;
  reason: string;
  expiresAt?: Date;
}

export default class BanRepository {
  async findAll(guildId: string, limit: number = 50, offset: number = 0): Promise<SelectBan[]> {
    logger.debug(`[${this.constructor.name}] "🔍 finding all bans for guild: ${guildId}"`);

    const bans = await db
      .select()
      .from(bansTable)
      .where(eq(bansTable.guildId, guildId))
      .limit(limit)
      .offset(offset);

    return bans;
  }

  async findById(id: string): Promise<SelectBan | null> {
    logger.debug(`[${this.constructor.name}] "🔍 finding ban by id: ${id}"`);

    const ban = await db.select().from(bansTable).where(eq(bansTable.id, id)).limit(1);

    return ban[0] ?? null;
  }

  async create(data: CreateBanDTO): Promise<SelectBan> {
    logger.info(`[${this.constructor.name}] "➕ creating ban for user: ${data.userId}"`);

    const [ban] = await db
      .insert(bansTable)
      .values({
        ...data,
        createdAt: new Date(),
      })
      .returning();

    return ban;
  }

  async delete(id: string): Promise<boolean> {
    logger.info(`[${this.constructor.name}] "🗑️ deleting ban: ${id}"`);

    const result = await db.delete(bansTable).where(eq(bansTable.id, id));

    return result.changes > 0;
  }
}
```

---

### Passo 3: Criar Service

`database/services/moderation/BanService.ts`

```typescript
import BanRepository, { type CreateBanDTO } from '@database/repositories/moderation/BanRepository';
import { logger } from '@config/logger';
import { redis } from '@config/redis';
import { AppError } from '@utils/errors';

export default class BanService {
  constructor(private banRepository: BanRepository) {}

  async listBans(guildId: string, page: number = 1, pageSize: number = 50) {
    logger.debug(`[${this.constructor.name}] "📋 listing bans for guild: ${guildId}"`);

    if (!guildId) {
      throw new AppError('Guild ID é obrigatório', 400);
    }

    if (page < 1 || pageSize < 1) {
      throw new AppError('Page e pageSize devem ser maiores que 0', 400);
    }

    const offset = (page - 1) * pageSize;
    const bans = await this.banRepository.findAll(guildId, pageSize, offset);

    return {
      data: bans,
      total: bans.length,
      page,
      pageSize,
    };
  }

  async createBan(data: CreateBanDTO) {
    logger.info(`[${this.constructor.name}] "🚫 creating ban for user: ${data.userId}"`);

    if (!data.userId || !data.guildId) {
      throw new AppError('User ID e Guild ID são obrigatórios', 400);
    }

    if (!data.reason) {
      throw new AppError('Razão do ban é obrigatória', 400);
    }

    const ban = await this.banRepository.create(data);

    // Publica evento no Redis
    await redis.publish(
      'ban:created',
      JSON.stringify({
        id: ban.id,
        guildId: ban.guildId,
        userId: ban.userId,
        reason: ban.reason,
        timestamp: ban.createdAt,
      })
    );

    logger.info(`[${this.constructor.name}] "✅ ban created successfully: ${ban.id}"`);

    return ban;
  }

  async removeBan(id: string) {
    logger.info(`[${this.constructor.name}] "➖ removing ban: ${id}"`);

    const ban = await this.banRepository.findById(id);

    if (!ban) {
      throw new AppError('Ban not found', 404);
    }

    await this.banRepository.delete(id);

    await redis.publish(
      'ban:removed',
      JSON.stringify({
        id: ban.id,
        guildId: ban.guildId,
      })
    );

    return { success: true };
  }
}
```

---

### Passo 4: Criar Controller

`api/controllers/moderation/BanController.ts`

```typescript
import type { Request, Response } from 'express';
import BanService from '@database/services/moderation/BanService';
import BanRepository from '@database/repositories/moderation/BanRepository';
import { logger } from '@config/logger';

export default class BanController {
  private service: BanService;

  constructor() {
    const repository = new BanRepository();
    this.service = new BanService(repository);
  }

  public async list(req: Request, res: Response): Promise<void> {
    logger.debug(`[${this.constructor.name}] "📨 GET /bans received"`);

    try {
      const { guildId, page = '1', pageSize = '50' } = req.query;

      if (!guildId || typeof guildId !== 'string') {
        res.status(400).json({ error: 'guildId query parameter is required' });
        return;
      }

      const result = await this.service.listBans(
        guildId,
        parseInt(page as string),
        parseInt(pageSize as string)
      );

      res.json(result);
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ error listing bans: ${error}"`);
      res.status(500).json({ error: 'Internal server error' });
    }
  }

  public async create(req: Request, res: Response): Promise<void> {
    logger.debug(`[${this.constructor.name}] "📨 POST /bans received"`);

    try {
      const { guildId, userId, reason, expiresAt } = req.body;

      const ban = await this.service.createBan({
        guildId,
        userId,
        reason,
        expiresAt: expiresAt ? new Date(expiresAt) : undefined,
      });

      res.status(201).json(ban);
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ error creating ban: ${error}"`);
      res.status(500).json({ error: 'Internal server error' });
    }
  }

  public async remove(req: Request, res: Response): Promise<void> {
    logger.debug(`[${this.constructor.name}] "📨 DELETE /bans/:id received"`);

    try {
      const { id } = req.params;

      const result = await this.service.removeBan(id);

      res.json(result);
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ error removing ban: ${error}"`);
      res.status(500).json({ error: 'Internal server error' });
    }
  }
}
```

---

### Passo 5: Criar Routes

`api/routes/v1/moderation/bans.routes.ts`

```typescript
import { Router } from 'express';
import BanController from '@api/controllers/moderation/BanController';

const router = Router();
const controller = new BanController();

// GET /api/v1/moderation/bans?guildId=123&page=1
router.get('/', (req, res) => controller.list(req, res));

// POST /api/v1/moderation/bans
router.post('/', (req, res) => controller.create(req, res));

// DELETE /api/v1/moderation/bans/:id
router.delete('/:id', (req, res) => controller.remove(req, res));

export default router;
```

---

### Passo 6: Registrar Routes

`api/routes/v1/moderation/index.routes.ts`

```typescript
import { Router } from 'express';
import bansRoutes from './bans.routes';
import kicksRoutes from './kicks.routes';

const router = Router();

router.use('/bans', bansRoutes);
router.use('/kicks', kicksRoutes);
// ... outros

export default router;
```

---

### Passo 7: Testar

```bash
# Listar bans
curl -X GET "http://localhost:3000/api/v1/moderation/bans?guildId=123"

# Criar ban
curl -X POST http://localhost:3000/api/v1/moderation/bans \
  -H "Content-Type: application/json" \
  -d '{
    "guildId": "123",
    "userId": "456",
    "reason": "Spam"
  }'

# Remover ban
curl -X DELETE http://localhost:3000/api/v1/moderation/bans/ban_123
```

---

## Links Relacionados

- [Estrutura](./STRUCTURE.md)
- [Padrões](./PATTERNS.md)
- [Testing](./TESTING.md)
- [API Docs](../08-api/API.md)
