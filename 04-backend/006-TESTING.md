# Backend - Testing

## Comandos

```bash
pnpm test             # Todos os testes
pnpm test:unit        # Testes unitários
pnpm test:integration # Testes de integração
pnpm test:coverage    # Com cobertura
```

---

## Teste Unitário

### Exemplo: `BanService.test.ts`

```typescript
import BanService from '@database/services/moderation/BanService';
import BanRepository from '@database/repositories/moderation/BanRepository';

jest.mock('@database/repositories/moderation/BanRepository');

describe('BanService', () => {
  let service: BanService;
  let repository: jest.Mocked<BanRepository>;

  beforeEach(() => {
    repository = new BanRepository() as jest.Mocked<BanRepository>;
    service = new BanService(repository);
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('createBan', () => {
    it('should create a ban successfully', async () => {
      const data = {
        guildId: '123',
        userId: '456',
        reason: 'Spam',
      };

      repository.create.mockResolvedValue({
        id: 'ban_123',
        ...data,
        expiresAt: null,
        createdAt: new Date(),
      });

      const result = await service.createBan(data);

      expect(result.id).toBe('ban_123');
      expect(repository.create).toHaveBeenCalledWith(data);
    });

    it('should throw error when userId is missing', async () => {
      const data = {
        guildId: '123',
        userId: '',
        reason: 'Spam',
      };

      await expect(service.createBan(data)).rejects.toThrow('User ID e Guild ID são obrigatórios');
    });
  });

  describe('listBans', () => {
    it('should return paginated bans', async () => {
      repository.findAll.mockResolvedValue([
        { id: 'ban_1', guildId: '123', userId: '456', reason: 'Test', expiresAt: null, createdAt: new Date() },
      ]);

      const result = await service.listBans('123', 1, 10);

      expect(result.data).toHaveLength(1);
      expect(result.page).toBe(1);
      expect(result.pageSize).toBe(10);
    });
  });
});
```

---

## Teste de Integração

### Exemplo: API Integration

```typescript
import axios from 'axios';

describe('BAN API Integration', () => {
  const API_URL = 'http://localhost:3000/api/v1';
  let createdBanId: string;

  beforeAll(async () => {
    // Setup: garantir servidor rodando
  });

  afterAll(async () => {
    // Cleanup: remover dados de teste
    if (createdBanId) {
      await axios.delete(`${API_URL}/moderation/bans/${createdBanId}`);
    }
  });

  it('should create a ban', async () => {
    const response = await axios.post(`${API_URL}/moderation/bans`, {
      guildId: 'test_guild_123',
      userId: 'test_user_456',
      reason: 'Integration test',
    });

    expect(response.status).toBe(201);
    expect(response.data.id).toBeDefined();
    createdBanId = response.data.id;
  });

  it('should list bans for guild', async () => {
    const response = await axios.get(`${API_URL}/moderation/bans?guildId=test_guild_123`);

    expect(response.status).toBe(200);
    expect(response.data.data).toBeInstanceOf(Array);
    expect(response.data.data).toContainEqual(
      expect.objectContaining({
        id: createdBanId,
      })
    );
  });

  it('should delete a ban', async () => {
    const response = await axios.delete(`${API_URL}/moderation/bans/${createdBanId}`);

    expect(response.status).toBe(200);
    expect(response.data.success).toBe(true);
    createdBanId = ''; // Mark as cleaned
  });
});
```

---

## Estrutura de Testes

```
tests/
├─ unit/
│  ├─ services/
│  │  └─ BanService.test.ts
│  ├─ repositories/
│  │  └─ BanRepository.test.ts
│  └─ utils/
│     └─ validators.test.ts
│
├─ integration/
│  ├─ api/
│  │  ├─ bans.integration.test.ts
│  │  └─ users.integration.test.ts
│  └─ database/
│     └─ migrations.test.ts
│
├─ e2e/
│  └─ scenarios/
│     └─ ban.flow.e2e.test.ts
│
├─ fixtures/
│  ├─ guilds.fixtures.ts
│  └─ users.fixtures.ts
│
└─ setup.ts
```

---

## Boas Práticas

### 1. AAA Pattern

```typescript
it('should create ban', async () => {
  // Arrange
  const data = { guildId: '123', userId: '456', reason: 'Test' };

  // Act
  const result = await service.createBan(data);

  // Assert
  expect(result.id).toBeDefined();
});
```

### 2. Um assert por teste

```typescript
// ✅ CORRETO
it('should return ban id', () => {
  expect(result.id).toBeDefined();
});

it('should return correct guildId', () => {
  expect(result.guildId).toBe('123');
});

// ❌ EVITAR: Múltiplos asserts não relacionados
it('should create ban', () => {
  expect(result.id).toBeDefined();
  expect(result.guildId).toBe('123');
  expect(result.userId).toBe('456');
  expect(logger.info).toHaveBeenCalled();
});
```

### 3. Nomenclatura descritiva

```typescript
// ✅ CORRETO
describe('createBan', () => {
  it('should throw error when reason is empty', () => {});
  it('should publish event to Redis after creation', () => {});
});

// ❌ EVITAR
describe('createBan', () => {
  it('test1', () => {});
  it('works', () => {});
});
```

---

## Links Relacionados

- [Testing Geral](../11-development/TESTING.md)
- [Padrões](./PATTERNS.md)
- [Setup](./SETUP.md)
