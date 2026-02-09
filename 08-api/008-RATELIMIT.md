# API - Rate Limiting

## Headers

Toda resposta inclui headers de rate limit:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1740324000
```

| Header | Descrição |
|--------|-----------|
| `X-RateLimit-Limit` | Limite de requests na janela |
| `X-RateLimit-Remaining` | Requests restantes |
| `X-RateLimit-Reset` | Timestamp Unix do reset |

---

## Limites por Endpoint

| Endpoint | Limite | Janela |
|----------|--------|--------|
| `/core/*` | 100 | 60s |
| `/moderation/*` | 50 | 60s |
| `/audit/*` | 30 | 60s |
| `/config/*` | 25 | 60s |

---

## Resposta de Limite Excedido

```json
{
  "error": "Too Many Requests",
  "code": "RATE_LIMITED",
  "message": "Rate limit exceeded. Reset in 30 seconds",
  "status": 429,
  "retryAfter": 30
}
```

---

## Tratamento no Cliente

### JavaScript

```typescript
async function apiCall(endpoint: string, retries = 3) {
  try {
    return await api.get(endpoint);
  } catch (error) {
    if (error.response?.status === 429 && retries > 0) {
      const retryAfter = error.response.data.retryAfter || 5;
      await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      return apiCall(endpoint, retries - 1);
    }
    throw error;
  }
}
```

### Verificar Antes

```typescript
const remaining = parseInt(response.headers['x-ratelimit-remaining']);

if (remaining < 5) {
  console.warn('Poucas requisições restantes');
}
```

---

## Boas Práticas

### 1. Cache de Dados

```typescript
// Cachear dados frequentemente acessados
const cache = new Map();

async function getGuild(guildId: string) {
  if (cache.has(guildId)) {
    return cache.get(guildId);
  }

  const guild = await api.get(`/core/guilds/${guildId}`);
  cache.set(guildId, guild.data);

  return guild.data;
}
```

### 2. Batch Requests

```typescript
// Em vez de múltiplas chamadas
for (const userId of userIds) {
  await api.get(`/core/users/${userId}`);
}

// Use endpoints de lista
const users = await api.get('/core/users', {
  params: { ids: userIds.join(',') }
});
```

### 3. Exponential Backoff

```typescript
async function withBackoff(fn: () => Promise<any>, maxRetries = 5) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.response?.status !== 429) throw error;

      const delay = Math.pow(2, i) * 1000;
      await new Promise(r => setTimeout(r, delay));
    }
  }
  throw new Error('Max retries exceeded');
}
```

---

## Links Relacionados

- [Errors](./ERRORS.md)
- [Visão Geral](./OVERVIEW.md)
