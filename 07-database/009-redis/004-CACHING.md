# Redis - Caching Strategies

## Visão Geral

Cache é uma das principais uses cases do Redis. Armazenar dados frequentemente acessados na memória reduz drasticamente a latência e carga no banco de dados.

---

## Cache-Aside Pattern (mais comum)

```typescript
async function getUser(userId: string) {
  const cacheKey = `user:${userId}`;

  // 1. Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // 2. Cache miss → query database
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId)
  });

  // 3. Store in cache (TTL 5 min)
  await redis.setex(cacheKey, 300, JSON.stringify(user));

  return user;
}
```

---

## Implementação de Cache Service

**`packages/backend/src/services/cache.ts`**

```typescript
import { getRedis } from '../lib/redis';
import { logger } from '../lib/logger';

export class CacheService {
  private redis = getRedis();
  private defaultTTL = 300; // 5 minutes

  // Get from cache
  async get<T>(key: string): Promise<T | null> {
    try {
      const value = await this.redis.get(key);
      if (!value) return null;

      return JSON.parse(value) as T;
    } catch (error) {
      logger.error('Cache get error', { key, error });
      return null;
    }
  }

  // Set to cache
  async set(key: string, value: any, ttl: number = this.defaultTTL): Promise<void> {
    try {
      await this.redis.setex(key, ttl, JSON.stringify(value));
      logger.debug('Cache set', { key, ttl });
    } catch (error) {
      logger.error('Cache set error', { key, error });
    }
  }

  // Delete from cache
  async del(key: string): Promise<void> {
    try {
      await this.redis.del(key);
      logger.debug('Cache deleted', { key });
    } catch (error) {
      logger.error('Cache delete error', { key, error });
    }
  }

  // Delete multiple keys
  async delPattern(pattern: string): Promise<void> {
    try {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        await this.redis.del(...keys);
        logger.debug('Cache pattern deleted', { pattern, count: keys.length });
      }
    } catch (error) {
      logger.error('Cache delete pattern error', { pattern, error });
    }
  }

  // Check if key exists
  async exists(key: string): Promise<boolean> {
    try {
      const result = await this.redis.exists(key);
      return result === 1;
    } catch (error) {
      logger.error('Cache exists error', { key, error });
      return false;
    }
  }

  // Get TTL of key
  async ttl(key: string): Promise<number> {
    try {
      return await this.redis.ttl(key);
    } catch (error) {
      logger.error('Cache TTL error', { key, error });
      return -1;
    }
  }

  // Increment counter
  async incr(key: string, ttl?: number): Promise<number> {
    try {
      const value = await this.redis.incr(key);
      if (ttl && value === 1) {
        await this.redis.expire(key, ttl);
      }
      return value;
    } catch (error) {
      logger.error('Cache incr error', { key, error });
      return 0;
    }
  }

  // Remember pattern (cache or compute)
  async remember<T>(
    key: string,
    ttl: number,
    callback: () => Promise<T>
  ): Promise<T> {
    // Try cache
    const cached = await this.get<T>(key);
    if (cached !== null) {
      return cached;
    }

    // Compute value
    const value = await callback();

    // Store in cache
    await this.set(key, value, ttl);

    return value;
  }
}

export const cache = new CacheService();
```

---

## Casos de Uso

### 1. Cache de Database Queries

```typescript
import { cache } from '../services/cache';
import { db } from '../db';

// Get user with cache
async function getUser(userId: string) {
  return cache.remember(
    `user:${userId}`,
    300, // 5 min
    async () => {
      return await db.query.users.findFirst({
        where: eq(users.id, userId)
      });
    }
  );
}

// Invalidate on update
async function updateUser(userId: string, data: any) {
  await db.update(users).set(data).where(eq(users.id, userId));

  // Invalidate cache
  await cache.del(`user:${userId}`);
}
```

### 2. Cache de API Responses

```typescript
app.get('/api/guilds/:id', async (request, reply) => {
  const { id } = request.params;

  const guild = await cache.remember(
    `guild:${id}`,
    600, // 10 min
    async () => {
      return await fetchGuildFromAPI(id);
    }
  );

  return guild;
});
```

### 3. Rate Limiting

```typescript
async function checkRateLimit(
  userId: string,
  limit: number = 10,
  window: number = 60
): Promise<boolean> {
  const key = `ratelimit:${userId}`;
  const current = await cache.incr(key, window);

  return current <= limit;
}

// Usage
if (!await checkRateLimit(userId)) {
  throw new Error('Rate limit exceeded');
}
```

### 4. Session Storage

```typescript
// Create session
async function createSession(userId: string, data: any): Promise<string> {
  const sessionId = uuid();
  const key = `session:${sessionId}`;

  await cache.set(key, { userId, ...data }, 3600); // 1 hour

  return sessionId;
}

// Get session
async function getSession(sessionId: string) {
  return await cache.get(`session:${sessionId}`);
}

// Destroy session
async function destroySession(sessionId: string) {
  await cache.del(`session:${sessionId}`);
}
```

### 5. Leaderboard/Rankings

```typescript
// Add score
await redis.zadd('leaderboard', score, userId);

// Get top 10
const top10 = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES');

// Get user rank
const rank = await redis.zrevrank('leaderboard', userId);

// Get user score
const score = await redis.zscore('leaderboard', userId);
```

---

## Cache Key Naming Convention

```typescript
// Pattern: entity:id:field
'user:123'                  // User object
'user:123:profile'          // User profile
'guild:456'                 // Guild object
'guild:456:config'          // Guild config

// Pattern: entity:list:filter
'users:list:active'         // Active users list
'guilds:list:premium'       // Premium guilds

// Pattern: action:entity:id
'ratelimit:user:123'        // Rate limit counter
'session:abc-def'           // Session data
```

---

## TTL Recommendations

```typescript
// Very frequently accessed, rarely changes
const CACHE_FOREVER = -1;          // No expiration (use carefully!)
const CACHE_DAY = 86400;           // 24 hours

// Frequently accessed
const CACHE_HOUR = 3600;           // 1 hour
const CACHE_15MIN = 900;           // 15 minutes

// Moderately accessed
const CACHE_5MIN = 300;            // 5 minutes (default)
const CACHE_1MIN = 60;             // 1 minute

// Temporary data
const CACHE_30SEC = 30;            // 30 seconds
const CACHE_10SEC = 10;            // 10 seconds
```

---

## Cache Invalidation Strategies

### 1. Time-Based (TTL)

```typescript
// Automatically expires after TTL
await cache.set('key', value, 300); // 5 min
```

### 2. Event-Based

```typescript
// Invalidate on specific events
eventSubscriber.subscribe('user:updated', async (event) => {
  const { userId } = event.payload;
  await cache.del(`user:${userId}`);
});
```

### 3. Pattern-Based

```typescript
// Invalidate all related caches
async function invalidateUserCaches(userId: string) {
  await cache.delPattern(`user:${userId}:*`);
}
```

### 4. Write-Through

```typescript
// Update cache immediately after write
async function updateUser(userId: string, data: any) {
  await db.update(users).set(data).where(eq(users.id, userId));

  const updated = await db.query.users.findFirst({
    where: eq(users.id, userId)
  });

  await cache.set(`user:${userId}`, updated, 300);
}
```

---

## Cache Warming

Pré-popular cache com dados importantes:

```typescript
async function warmCache() {
  logger.info('Warming cache...');

  // Top 100 users
  const topUsers = await db.select().from(users).limit(100);
  for (const user of topUsers) {
    await cache.set(`user:${user.id}`, user, 3600);
  }

  // All guilds
  const guilds = await db.select().from(guilds);
  for (const guild of guilds) {
    await cache.set(`guild:${guild.id}`, guild, 3600);
  }

  logger.info('Cache warmed');
}

// Run on startup
warmCache();
```

---

## Monitoring Cache Performance

### Hit Rate Tracking

```typescript
export class CacheService {
  private hits = 0;
  private misses = 0;

  async get<T>(key: string): Promise<T | null> {
    const value = await this.redis.get(key);

    if (value) {
      this.hits++;
    } else {
      this.misses++;
    }

    return value ? JSON.parse(value) : null;
  }

  getStats() {
    const total = this.hits + this.misses;
    const hitRate = total > 0 ? (this.hits / total) * 100 : 0;

    return {
      hits: this.hits,
      misses: this.misses,
      hitRate: `${hitRate.toFixed(2)}%`
    };
  }
}
```

### Metrics Endpoint

```typescript
app.get('/metrics/cache', async () => {
  const stats = cache.getStats();
  const info = await redis.info('memory');

  return {
    ...stats,
    memory: info
  };
});
```

---

## Best Practices

### ✅ DO

```typescript
// 1. Always set TTL
await cache.set(key, value, 300);

// 2. Handle cache failures gracefully
const cached = await cache.get(key);
if (!cached) {
  // Fallback to database
}

// 3. Use consistent key naming
const key = `user:${userId}:profile`;

// 4. Invalidate on updates
await cache.del(key);

// 5. Monitor hit rate
logger.info('Cache stats', cache.getStats());
```

### ❌ DON'T

```typescript
// 1. Don't cache everything
// Cache only frequently accessed data

// 2. Don't cache large objects (> 1MB)
// Use compression or store reference

// 3. Don't use cache as primary storage
// Cache should be ephemeral

// 4. Don't forget to invalidate
// Stale cache = bugs

// 5. Don't use long TTL for dynamic data
// Balance freshness vs performance
```

---

## Troubleshooting

### High memory usage

```bash
# Ver uso de memória
redis-cli INFO memory

# Ver keys maiores
redis-cli --bigkeys

# Analisar keys
redis-cli --scan --pattern 'user:*' | wc -l
```

### Low hit rate

```bash
# Verificar TTLs
redis-cli TTL key-name

# Ver keys expirando
redis-cli --scan --pattern '*'
```

---

**Cache eficiente com Redis!** ⚡
