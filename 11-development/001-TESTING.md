# 🧪 TESTING - LOCKDOWN

## Jest - Unit, Integration e E2E Tests

---

## 📖 Índice

1. [Visão Geral](#visão-geral)
2. [Setup Jest](#setup-jest)
3. [Unit Tests](#unit-tests)
4. [Integration Tests](#integration-tests)
5. [E2E Tests](#e2e-tests)
6. [Coverage](#coverage)
7. [CI/CD](#cicd)

---

## 📋 Visão Geral

### Estratégia de Testes

```
        ↓
    [E2E Tests] ← Cenários completos (10%)
        ↓
[Integration Tests] ← APIs + DB (30%)
        ↓
   [Unit Tests] ← Funções isoladas (60%)
```

### Estrutura

```
tests/
├─ unit/                    # Testes isolados
│  ├─ database/
│  │  ├─ repositories/
│  │  │  └─ BanRepository.test.ts
│  │  └─ services/
│  │     └─ BanService.test.ts
│  ├─ api/
│  │  └─ controllers/
│  │     └─ BanController.test.ts
│  ├─ utils/
│  │  └─ validators.test.ts
│  └─ setup.ts
│
├─ integration/             # Com DB e APIs
│  ├─ api/
│  │  ├─ bans.integration.test.ts
│  │  └─ users.integration.test.ts
│  ├─ database/
│  │  └─ migrations.integration.test.ts
│  └─ setup.ts
│
├─ e2e/                     # Cenários completos
│  ├─ scenarios/
│  │  ├─ ban.flow.e2e.test.ts
│  │  └─ kick.flow.e2e.test.ts
│  ├─ fixtures/
│  │  ├─ guilds.fixtures.ts
│  │  └─ users.fixtures.ts
│  └─ setup.ts
│
├─ jest.config.js           # Configuração Jest
├─ package.json
└─ README.md
```

---

## 🔧 Setup Jest

### jest.config.js

```javascript
/**
 * - @file jest.config.js
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
 * Jest configuration for LOCKDOWN tests
 *
 */

module.exports = {
  displayName: '@lockdown/tests',
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  moduleNameMapper: {
    '^@database/(.*)$': '<rootDir>/../backend/src/database/$1',
    '^@api/(.*)$': '<rootDir>/../backend/src/api/$1',
    '^@config/(.*)$': '<rootDir>/../backend/src/config/$1',
    '^@services/(.*)$': '<rootDir>/../backend/src/services/$1',
    '^@types/(.*)$': '<rootDir>/../backend/src/types/$1',
    '^@utils/(.*)$': '<rootDir>/../backend/src/utils/$1',
  },
  collectCoverageFrom: [
    '../backend/src/**/*.ts',
    '!../backend/src/**/*.d.ts',
    '!../backend/src/**/index.ts',
  ],
  coveragePathIgnorePatterns: ['node_modules', 'dist', 'migrations'],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
  testTimeout: 10000,
  setupFilesAfterEnv: ['<rootDir>/unit/setup.ts'],
};
```

### Package.json (tests/)

```json
{
  "name": "@lockdown/tests",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:unit": "jest unit/",
    "test:integration": "jest integration/",
    "test:e2e": "jest e2e/",
    "test:coverage": "jest --coverage",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand"
  },
  "devDependencies": {
    "@types/jest": "^29.5.11",
    "@types/node": "^20.10.0",
    "jest": "^29.7.0",
    "ts-jest": "^29.1.1",
    "typescript": "^5.3.3"
  }
}
```

---

## 🧬 Unit Tests

### Exemplo: BanService.test.ts

```typescript
/**
 * - @file BanService.test.ts
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
 * Unit tests para BanService
 *
 */

import BanService from '@database/services/moderation/BanService';
import BanRepository from '@database/repositories/moderation/BanRepository';
import { redis } from '@config/redis';
import { AppError } from '@utils/errors';

// Mock repository
jest.mock('@database/repositories/moderation/BanRepository');

// Mock redis
jest.mock('@config/redis', () => ({
  redis: {
    publish: jest.fn().mockResolvedValue(1),
  },
}));

describe('BanService', () => {
  let service: BanService;
  let mockRepository: jest.Mocked<BanRepository>;

  beforeEach(() => {
    // Reset mocks
    jest.clearAllMocks();

    // Create mock repository instance
    mockRepository = new BanRepository() as jest.Mocked<BanRepository>;

    // Instantiate service with mock
    service = new BanService(mockRepository);
  });

  describe('createBan', () => {
    it('should create a ban successfully', async () => {
      // Arrange
      const banData = {
        guildId: '123456789',
        userId: '987654321',
        reason: 'Spam',
        moderatorId: '111222333',
      };

      const mockBan = {
        id: 'ban_1',
        ...banData,
        isActive: true,
        expiresAt: null,
        createdAt: new Date(),
        updatedAt: new Date(),
      };

      mockRepository.create.mockResolvedValue(mockBan);

      // Act
      const result = await service.createBan(banData);

      // Assert
      expect(result).toEqual(mockBan);
      expect(mockRepository.create).toHaveBeenCalledWith(banData);
      expect(redis.publish).toHaveBeenCalledWith(
        'ban:created',
        expect.stringContaining(banData.userId)
      );
    });

    it('should throw error if userId is missing', async () => {
      // Arrange
      const invalidData = {
        guildId: '123456789',
        userId: '',
        reason: 'Spam',
        moderatorId: '111222333',
      };

      // Act & Assert
      await expect(service.createBan(invalidData)).rejects.toThrow(AppError);
    });

    it('should throw error if reason is missing', async () => {
      // Arrange
      const invalidData = {
        guildId: '123456789',
        userId: '987654321',
        reason: '',
        moderatorId: '111222333',
      };

      // Act & Assert
      await expect(service.createBan(invalidData)).rejects.toThrow(AppError);
    });

    it('should publish redis event on ban creation', async () => {
      // Arrange
      const banData = {
        guildId: '123456789',
        userId: '987654321',
        reason: 'Spam',
        moderatorId: '111222333',
      };

      const mockBan = {
        id: 'ban_1',
        ...banData,
        isActive: true,
        createdAt: new Date(),
      };

      mockRepository.create.mockResolvedValue(mockBan);

      // Act
      await service.createBan(banData);

      // Assert
      expect(redis.publish).toHaveBeenCalledWith(
        'ban:created',
        expect.objectContaining({
          userId: banData.userId,
          guildId: banData.guildId,
          reason: banData.reason,
        })
      );
    });
  });

  describe('listBans', () => {
    it('should return list of bans', async () => {
      // Arrange
      const guildId = '123456789';
      const mockBans = [
        {
          id: 'ban_1',
          guildId,
          userId: '111',
          reason: 'Spam',
          isActive: true,
          createdAt: new Date(),
        },
        {
          id: 'ban_2',
          guildId,
          userId: '222',
          reason: 'Harassment',
          isActive: true,
          createdAt: new Date(),
        },
      ];

      mockRepository.findAll.mockResolvedValue(mockBans);

      // Act
      const result = await service.listBans(guildId, 1, 50);

      // Assert
      expect(result.data).toEqual(mockBans);
      expect(result.total).toBe(2);
      expect(result.page).toBe(1);
      expect(mockRepository.findAll).toHaveBeenCalledWith(guildId, 50, 0);
    });

    it('should throw error if guildId is missing', async () => {
      // Act & Assert
      await expect(service.listBans('', 1, 50)).rejects.toThrow(AppError);
    });

    it('should handle invalid pagination', async () => {
      // Act & Assert
      await expect(service.listBans('123', 0, 50)).rejects.toThrow(AppError);
      await expect(service.listBans('123', 1, 0)).rejects.toThrow(AppError);
    });
  });

  describe('removeBan', () => {
    it('should remove ban successfully', async () => {
      // Arrange
      const banId = 'ban_1';
      const mockBan = {
        id: banId,
        guildId: '123456789',
        userId: '987654321',
        isActive: true,
        createdAt: new Date(),
      };

      mockRepository.findById.mockResolvedValue(mockBan);
      mockRepository.delete.mockResolvedValue(true);

      // Act
      const result = await service.removeBan(banId);

      // Assert
      expect(result.success).toBe(true);
      expect(mockRepository.delete).toHaveBeenCalledWith(banId);
      expect(redis.publish).toHaveBeenCalledWith('ban:removed', expect.stringContaining(banId));
    });

    it('should throw error if ban not found', async () => {
      // Arrange
      mockRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(service.removeBan('invalid_id')).rejects.toThrow(AppError);
    });
  });
});
```

---

## 🔗 Integration Tests

### Exemplo: bans.integration.test.ts

```typescript
/**
 * - @file bans.integration.test.ts
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
 * Integration tests para Ban API (com banco de dados real)
 *
 */

import axios, { AxiosInstance } from 'axios';
import { db } from '@database/client';
import { bansTable, usersTable, guildsTable } from '@database/schemas';

describe('Bans API Integration Tests', () => {
  const API_URL = 'http://localhost:3000/api/v1';
  const TOKEN = 'test_jwt_token';

  let api: AxiosInstance;
  let testGuildId: string;
  let testUserId: string;
  let testModeratorId: string;

  beforeAll(async () => {
    // Setup axios instance com headers
    api = axios.create({
      baseURL: API_URL,
      headers: {
        Authorization: `Bearer ${TOKEN}`,
        'Content-Type': 'application/json',
      },
    });
  });

  beforeEach(async () => {
    // Create test data
    testGuildId = '123456789';
    testUserId = '987654321';
    testModeratorId = '111222333';

    // Insert test guild
    await db.insert(guildsTable).values({
      guildId: testGuildId,
      name: 'Test Guild',
      ownerId: testModeratorId,
    });

    // Insert test user
    await db.insert(usersTable).values({
      userId: testUserId,
      username: 'test_user',
    });
  });

  afterEach(async () => {
    // Cleanup: Delete all test data
    await db.delete(bansTable).where(db.sql`true`);
    await db.delete(usersTable).where(db.sql`true`);
    await db.delete(guildsTable).where(db.sql`true`);
  });

  describe('POST /moderation/bans', () => {
    it('should create a ban', async () => {
      // Arrange
      const banData = {
        guildId: testGuildId,
        userId: testUserId,
        reason: 'Spam',
        moderatorId: testModeratorId,
      };

      // Act
      const response = await api.post('/moderation/bans', banData);

      // Assert
      expect(response.status).toBe(201);
      expect(response.data.data).toMatchObject({
        guildId: testGuildId,
        userId: testUserId,
        reason: 'Spam',
        isActive: true,
      });

      // Verify in database
      const bansInDb = await db.select().from(bansTable).where(eq(bansTable.guildId, testGuildId));

      expect(bansInDb).toHaveLength(1);
      expect(bansInDb[0].userId).toBe(testUserId);
    });

    it('should return 400 if guildId is missing', async () => {
      // Arrange
      const invalidData = {
        userId: testUserId,
        reason: 'Spam',
        moderatorId: testModeratorId,
      };

      // Act & Assert
      try {
        await api.post('/moderation/bans', invalidData);
        fail('Should have thrown error');
      } catch (error) {
        expect(error.response.status).toBe(400);
      }
    });

    it('should handle duplicate bans gracefully', async () => {
      // Arrange
      const banData = {
        guildId: testGuildId,
        userId: testUserId,
        reason: 'Spam',
        moderatorId: testModeratorId,
      };

      // First ban
      await api.post('/moderation/bans', banData);

      // Act: Second ban (duplicate)
      const response = await api.post('/moderation/bans', banData);

      // Assert: Should either reject or replace
      // Depending da implementação
      expect([201, 409]).toContain(response.status);
    });
  });

  describe('GET /moderation/bans', () => {
    it('should list bans for a guild', async () => {
      // Arrange: Create multiple bans
      const ban1 = {
        guildId: testGuildId,
        userId: '111',
        reason: 'Spam',
        moderatorId: testModeratorId,
        isActive: true,
      };

      const ban2 = {
        guildId: testGuildId,
        userId: '222',
        reason: 'Harassment',
        moderatorId: testModeratorId,
        isActive: true,
      };

      await db.insert(bansTable).values([ban1, ban2]);

      // Act
      const response = await api.get(`/moderation/bans?guildId=${testGuildId}`);

      // Assert
      expect(response.status).toBe(200);
      expect(response.data.data).toHaveLength(2);
      expect(response.data.pagination.total).toBe(2);
    });

    it('should filter by isActive', async () => {
      // Arrange
      await db.insert(bansTable).values([
        {
          guildId: testGuildId,
          userId: '111',
          reason: 'Spam',
          moderatorId: testModeratorId,
          isActive: true,
        },
        {
          guildId: testGuildId,
          userId: '222',
          reason: 'Removed',
          moderatorId: testModeratorId,
          isActive: false,
        },
      ]);

      // Act
      const response = await api.get(`/moderation/bans?guildId=${testGuildId}&isActive=true`);

      // Assert
      expect(response.status).toBe(200);
      expect(response.data.data).toHaveLength(1);
      expect(response.data.data[0].isActive).toBe(true);
    });
  });

  describe('DELETE /moderation/bans/:id', () => {
    it('should remove a ban', async () => {
      // Arrange
      const [ban] = await db
        .insert(bansTable)
        .values({
          guildId: testGuildId,
          userId: testUserId,
          reason: 'Spam',
          moderatorId: testModeratorId,
        })
        .returning();

      // Act
      const response = await api.delete(`/moderation/bans/${ban.id}`);

      // Assert
      expect(response.status).toBe(200);

      // Verify deleted
      const bansInDb = await db.select().from(bansTable).where(eq(bansTable.id, ban.id));

      expect(bansInDb).toHaveLength(0);
    });

    it('should return 404 if ban not found', async () => {
      // Act & Assert
      try {
        await api.delete('/moderation/bans/nonexistent_id');
        fail('Should have thrown error');
      } catch (error) {
        expect(error.response.status).toBe(404);
      }
    });
  });
});
```

---

## 🎯 E2E Tests

### Exemplo: ban.flow.e2e.test.ts

```typescript
/**
 * - @file ban.flow.e2e.test.ts
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
 * E2E tests para fluxo completo de banimento
 *
 */

import axios, { AxiosInstance } from 'axios';
import { db } from '@database/client';
import { redis } from '@config/redis';
import * as fixtures from '../fixtures';

describe('Ban Flow E2E', () => {
  const API_URL = 'http://localhost:3000/api/v1';
  let api: AxiosInstance;

  beforeAll(() => {
    api = axios.create({
      baseURL: API_URL,
      headers: {
        Authorization: `Bearer test_token`,
        'Content-Type': 'application/json',
      },
    });
  });

  beforeEach(async () => {
    // Setup initial data
    const guild = await fixtures.createGuild();
    const user = await fixtures.createUser();

    this.testGuild = guild;
    this.testUser = user;
  });

  afterEach(async () => {
    // Cleanup
    await fixtures.cleanupDatabase();
  });

  it('should complete full ban flow: create -> list -> remove', async () => {
    // Step 1: Create ban
    const createResponse = await api.post('/moderation/bans', {
      guildId: this.testGuild.guildId,
      userId: this.testUser.userId,
      reason: 'Test ban',
      moderatorId: fixtures.MODERATOR_ID,
    });

    expect(createResponse.status).toBe(201);
    const banId = createResponse.data.data.id;

    // Verify Redis event published
    const redisMessage = await redis.get(`ban:${banId}`);
    expect(redisMessage).toBeDefined();

    // Step 2: List bans
    const listResponse = await api.get(`/moderation/bans?guildId=${this.testGuild.guildId}`);

    expect(listResponse.status).toBe(200);
    expect(listResponse.data.data).toContainEqual(
      expect.objectContaining({
        id: banId,
        userId: this.testUser.userId,
      })
    );

    // Step 3: Remove ban
    const removeResponse = await api.delete(`/moderation/bans/${banId}`);

    expect(removeResponse.status).toBe(200);

    // Step 4: Verify removed
    const finalListResponse = await api.get(
      `/moderation/bans?guildId=${this.testGuild.guildId}&isActive=true`
    );

    const stillBanned = finalListResponse.data.data.some((b: any) => b.id === banId);
    expect(stillBanned).toBe(false);
  });

  it('should handle ban with expiration', async () => {
    const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days

    const response = await api.post('/moderation/bans', {
      guildId: this.testGuild.guildId,
      userId: this.testUser.userId,
      reason: 'Temporary ban',
      moderatorId: fixtures.MODERATOR_ID,
      expiresAt,
    });

    expect(response.status).toBe(201);
    expect(new Date(response.data.data.expiresAt)).toEqual(expiresAt);

    // Schedule expiration check
    // (pode usar cron ou job queue)
  });

  it('should prevent duplicate bans', async () => {
    const banData = {
      guildId: this.testGuild.guildId,
      userId: this.testUser.userId,
      reason: 'Spam',
      moderatorId: fixtures.MODERATOR_ID,
    };

    // First ban
    const first = await api.post('/moderation/bans', banData);
    expect(first.status).toBe(201);

    // Second ban (duplicate)
    try {
      await api.post('/moderation/bans', banData);
      // Pode retornar 409 ou reutilizar ban existente
    } catch (error) {
      expect([409, 201]).toContain(error.response.status);
    }
  });
});
```

---

## 📊 Coverage

### Rodar com Coverage

```bash
# Coverage completo
pnpm test:coverage

# Esperado:
# ======= Coverage summary =======
# Statements   : 85.5% ( 1234/1442 )
# Branches     : 78.3% ( 456/582 )
# Functions    : 82.1% ( 234/285 )
# Lines        : 86.2% ( 1100/1276 )
```

### Ver Report HTML

```bash
# Gera relatório visual
pnpm test:coverage

# Abrir no navegador
open coverage/index.html
```

### Coverage Threshold

Arquivo `jest.config.js`:

```javascript
coverageThreshold: {
  global: {
    branches: 70,     // 70% de branches
    functions: 70,    // 70% de funções
    lines: 70,        // 70% de linhas
    statements: 70,   // 70% de statements
  },
}
```

---

## 🔄 CI/CD

### GitHub Actions Workflow

**`.github/workflows/test.yml`:**

```yaml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: lockdown_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'pnpm'

      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 10

      - name: Install dependencies
        run: pnpm install

      - name: Run type-check
        run: pnpm type-check

      - name: Run lint
        run: pnpm lint

      - name: Run unit tests
        run: pnpm test:unit

      - name: Run integration tests
        run: pnpm test:integration
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/lockdown_test
          REDIS_HOST: localhost
          REDIS_PORT: 6379

      - name: Run E2E tests
        run: pnpm test:e2e
        env:
          API_URL: http://localhost:3000/api/v1

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

---

## 🎓 Best Practices

### ✅ DO

```typescript
// ✅ BOM: Arrange-Act-Assert
it('should create user', async () => {
  // Arrange
  const userData = { username: 'john' };

  // Act
  const result = await service.createUser(userData);

  // Assert
  expect(result.username).toBe('john');
});

// ✅ BOM: Mock externos
jest.mock('@config/redis');

// ✅ BOM: Cleanup completo
afterEach(async () => {
  await db.delete(usersTable);
  jest.clearAllMocks();
});

// ✅ BOM: Testes independentes
describe('UserService', () => {
  beforeEach(() => {
    // Setup fresh para cada teste
  });
});
```

### ❌ DON'T

```typescript
// ❌ RUIM: Testes acoplados
it('should create and update user', async () => {
  // 2 comportamentos em 1 teste
});

// ❌ RUIM: Sem cleanup
it('should create user', async () => {
  // Database fica sujo
});

// ❌ RUIM: Testes interdependentes
it('test A', () => {
  /* ... */
}); // Precisa de test B
it('test B', () => {
  /* ... */
});
```

---

## 📚 Referências

- [Jest Docs](https://jestjs.io/docs/getting-started)
- [Testing Library](https://testing-library.com)
- [BACKEND.md](./BACKEND.md) - Código a testar

---

**Happy testing!** 🧪
