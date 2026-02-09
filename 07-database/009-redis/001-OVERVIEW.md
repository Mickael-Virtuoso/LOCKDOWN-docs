# Redis - Overview

## O que é Redis?

**Redis** (Remote Dictionary Server) é um banco de dados **in-memory** chave-valor extremamente rápido. Usado para cache, Pub/Sub e armazenamento de dados temporários.

---

## Por Que Usamos Redis no LOCKDOWN?

### 1. **Pub/Sub de Eventos** 🔄
- Comunicação em tempo real entre bots e backend
- Event-driven architecture
- Desacoplamento de serviços

### 2. **Cache de Dados** ⚡
- Cache de queries do banco de dados
- Cache de respostas de API
- Reduz latência drasticamente

### 3. **Sessões e Rate Limiting** 🔐
- Gerenciamento de sessões de usuário
- Rate limiting de APIs
- Controle de abuso

### 4. **Dados Temporários** ⏱️
- Dados de curta duração (TTL automático)
- Filas de processamento
- Contadores e estatísticas

---

## Driver: ioredis

Usamos **ioredis**, o cliente Redis mais popular para Node.js.

### Por Que ioredis?

```typescript
✅ Full TypeScript support
✅ Cluster support
✅ Sentinel support
✅ Pipelining e transactions
✅ Pub/Sub robusto
✅ Auto-reconnect
✅ Promise-based API
```

### Instalação

```bash
pnpm add ioredis
pnpm add -D @types/ioredis
```

---

## Casos de Uso no LOCKDOWN

### 1. Event Bus (Pub/Sub)

```typescript
// Publisher (Bot Discord)
await redis.publish('moderation:action', JSON.stringify({
  type: 'BAN',
  userId: '123',
  guildId: '456',
  reason: 'Spam'
}));

// Subscriber (Backend)
redis.subscribe('moderation:action', (err, count) => {
  console.log(`Subscribed to ${count} channels`);
});

redis.on('message', (channel, message) => {
  const event = JSON.parse(message);
  // Process event
});
```

### 2. Cache de Database Queries

```typescript
import { redis } from './redis';
import { db } from './db';

async function getUser(userId: string) {
  // Try cache first
  const cached = await redis.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }

  // Fallback to database
  const user = await db.select().from(users).where(eq(users.id, userId));

  // Cache for 5 minutes
  await redis.setex(`user:${userId}`, 300, JSON.stringify(user));

  return user;
}
```

### 3. Rate Limiting

```typescript
async function checkRateLimit(userId: string, limit: number, window: number) {
  const key = `ratelimit:${userId}`;
  const current = await redis.incr(key);

  if (current === 1) {
    await redis.expire(key, window);
  }

  return current <= limit;
}

// Usage
const allowed = await checkRateLimit('user123', 10, 60); // 10 req/min
if (!allowed) {
  throw new Error('Rate limit exceeded');
}
```

### 4. Session Store

```typescript
// Store session
await redis.setex(
  `session:${sessionId}`,
  3600, // 1 hour
  JSON.stringify({ userId, roles })
);

// Get session
const session = await redis.get(`session:${sessionId}`);
if (!session) {
  throw new Error('Session expired');
}
```

---

## Estrutura de Dados Redis

### Strings (mais comum)

```typescript
await redis.set('key', 'value');
await redis.get('key');
await redis.incr('counter');
await redis.setex('key', 60, 'value'); // TTL 60s
```

### Hashes (objetos)

```typescript
await redis.hset('user:123', {
  name: 'John',
  email: 'john@example.com'
});

await redis.hget('user:123', 'name');
await redis.hgetall('user:123');
```

### Lists (filas)

```typescript
await redis.lpush('queue', 'job1');
await redis.rpop('queue');
```

### Sets (único)

```typescript
await redis.sadd('online:users', 'user1', 'user2');
await redis.smembers('online:users');
await redis.sismember('online:users', 'user1');
```

### Sorted Sets (ranking)

```typescript
await redis.zadd('leaderboard', 100, 'user1', 200, 'user2');
await redis.zrange('leaderboard', 0, 9); // Top 10
```

---

## Performance

### Latência Típica

```
GET/SET: < 1ms
PUBLISH: < 1ms
ZADD: < 1ms
```

### Throughput

```
Redis pode processar:
- 100,000+ operações/segundo (single instance)
- Milhões de ops/segundo (cluster)
```

---

## Development vs Production

### Development
```yaml
Host: localhost
Port: 6379
Password: (opcional)
Persistence: disabled
Max Memory: 256MB
```

### Production
```yaml
Host: redis.production.com
Port: 6379
Password: strong-password-here
Persistence: RDB + AOF
Max Memory: 4GB+
Replication: Master-Replica
```

---

## Segurança

### Best Practices

```bash
# 1. Sempre use senha
requirepass your-strong-password

# 2. Bind apenas IPs necessários
bind 127.0.0.1 192.168.1.100

# 3. Desabilite comandos perigosos
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command CONFIG ""

# 4. Use TLS em produção
tls-port 6380
tls-cert-file /path/to/cert.pem
```

---

## Monitoring

### Comandos Úteis

```bash
# Info geral
redis-cli INFO

# Memória
redis-cli INFO memory

# Stats
redis-cli INFO stats

# Clients conectados
redis-cli CLIENT LIST

# Slow queries
redis-cli SLOWLOG GET 10
```

---

## Limitações

```
❌ Não é um banco de dados primário
❌ Dados in-memory (limitado pela RAM)
❌ Single-threaded (um core CPU)
❌ Sem queries complexas (não é SQL)
```

### Quando NÃO Usar Redis

- Armazenamento permanente crítico → Use PostgreSQL
- Dados grandes (> 1GB por key) → Use object storage
- Queries complexas → Use SQL database
- ACID transactions → Use PostgreSQL

---

## Troubleshooting

### Redis não conecta

```bash
# Verificar se está rodando
redis-cli ping

# Ver logs
docker logs redis

# Testar porta
telnet localhost 6379
```

### Alta latência

```bash
# Ver slow queries
redis-cli SLOWLOG GET 10

# Verificar memória
redis-cli INFO memory

# Ver comandos lentos
redis-cli --latency
```

### Memory issues

```bash
# Ver uso de memória
redis-cli INFO memory

# Ver keys maiores
redis-cli --bigkeys

# Limpar tudo (CUIDADO!)
redis-cli FLUSHALL
```

---

## Recursos

- [Redis Documentation](https://redis.io/documentation)
- [ioredis GitHub](https://github.com/redis/ioredis)
- [Redis Commands](https://redis.io/commands)
- [Redis Best Practices](https://redis.io/topics/best-practices)

---

**Redis: velocidade in-memory para o LOCKDOWN!** 🔴
