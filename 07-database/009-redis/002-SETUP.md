# Redis - Setup e Configuração

## Instalação Local (Development)

### Opção 1: Docker (Recomendado)

```bash
# Pull image
docker pull redis:alpine

# Run container
docker run -d \
  --name lockdown-redis \
  -p 6379:6379 \
  redis:alpine

# Verificar se está rodando
docker ps | grep redis
```

### Opção 2: Docker Compose

Adicione ao `docker-compose.yml`:

```yaml
version: '3.8'

services:
  redis:
    image: redis:alpine
    container_name: lockdown-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped

volumes:
  redis-data:
```

```bash
# Start
docker-compose up -d redis

# Logs
docker-compose logs -f redis
```

### Opção 3: Native (Linux/Mac)

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install redis-server

# macOS
brew install redis

# Start service
redis-server

# Start as service
sudo systemctl start redis
sudo systemctl enable redis
```

---

## Instalação do Cliente (ioredis)

### No Backend

```bash
cd packages/backend
pnpm add ioredis
pnpm add -D @types/ioredis
```

---

## Configuração

### 1. Variáveis de Ambiente

**`packages/backend/.env`**

```ini
# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0

# Em produção
# REDIS_HOST=redis.production.com
# REDIS_PASSWORD=strong-password-here
# REDIS_TLS=true
```

### 2. Cliente Redis Singleton

**`packages/backend/src/lib/redis.ts`**

```typescript
import Redis from 'ioredis';
import { logger } from './logger';

// Singleton instance
let redis: Redis | null = null;

export function createRedisClient(): Redis {
  if (redis) {
    return redis;
  }

  const config = {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379'),
    password: process.env.REDIS_PASSWORD || undefined,
    db: parseInt(process.env.REDIS_DB || '0'),

    // Retry strategy
    retryStrategy: (times: number) => {
      const delay = Math.min(times * 50, 2000);
      return delay;
    },

    // Reconnect on error
    reconnectOnError: (err) => {
      const targetError = 'READONLY';
      if (err.message.includes(targetError)) {
        return true;
      }
      return false;
    },

    // Connection timeout
    connectTimeout: 10000,

    // Max retry time
    maxRetriesPerRequest: 3,
  };

  redis = new Redis(config);

  // Event handlers
  redis.on('connect', () => {
    logger.info('Redis connected');
  });

  redis.on('ready', () => {
    logger.info('Redis ready to accept commands');
  });

  redis.on('error', (err) => {
    logger.error('Redis error:', err);
  });

  redis.on('close', () => {
    logger.warn('Redis connection closed');
  });

  redis.on('reconnecting', (ms) => {
    logger.info(`Redis reconnecting in ${ms}ms`);
  });

  redis.on('end', () => {
    logger.warn('Redis connection ended');
  });

  return redis;
}

// Export singleton
export const getRedis = (): Redis => {
  if (!redis) {
    redis = createRedisClient();
  }
  return redis;
};

// Graceful shutdown
export async function closeRedis(): Promise<void> {
  if (redis) {
    await redis.quit();
    redis = null;
    logger.info('Redis connection closed gracefully');
  }
}
```

### 3. Uso no Backend

**`packages/backend/src/index.ts`**

```typescript
import { createRedisClient, closeRedis } from './lib/redis';

async function start() {
  // Initialize Redis
  const redis = createRedisClient();

  // Test connection
  await redis.ping();
  console.log('✅ Redis connected');

  // Your app initialization...

  // Graceful shutdown
  process.on('SIGTERM', async () => {
    await closeRedis();
    process.exit(0);
  });
}

start();
```

---

## Health Check

### Endpoint de Health

**`packages/backend/src/routes/health.ts`**

```typescript
import { FastifyInstance } from 'fastify';
import { getRedis } from '../lib/redis';

export async function healthRoutes(app: FastifyInstance) {
  app.get('/health', async (request, reply) => {
    try {
      const redis = getRedis();

      // Test Redis
      const redisPing = await redis.ping();

      return {
        status: 'ok',
        redis: redisPing === 'PONG' ? 'connected' : 'error',
        timestamp: new Date().toISOString()
      };
    } catch (error) {
      return reply.status(503).send({
        status: 'error',
        redis: 'disconnected',
        error: error.message
      });
    }
  });
}
```

---

## Configuração Avançada

### Redis com TLS (Production)

```typescript
import Redis from 'ioredis';
import fs from 'fs';

const redis = new Redis({
  host: 'redis.production.com',
  port: 6380,
  password: process.env.REDIS_PASSWORD,
  tls: {
    ca: fs.readFileSync('/path/to/ca.pem'),
    cert: fs.readFileSync('/path/to/cert.pem'),
    key: fs.readFileSync('/path/to/key.pem'),
  }
});
```

### Redis Cluster

```typescript
import { Cluster } from 'ioredis';

const cluster = new Cluster([
  {
    host: 'redis-node-1.com',
    port: 6379
  },
  {
    host: 'redis-node-2.com',
    port: 6379
  },
  {
    host: 'redis-node-3.com',
    port: 6379
  }
], {
  redisOptions: {
    password: process.env.REDIS_PASSWORD
  }
});
```

### Redis Sentinel (High Availability)

```typescript
const redis = new Redis({
  sentinels: [
    { host: 'sentinel-1.com', port: 26379 },
    { host: 'sentinel-2.com', port: 26379 },
    { host: 'sentinel-3.com', port: 26379 }
  ],
  name: 'mymaster',
  password: process.env.REDIS_PASSWORD
});
```

---

## Testes de Conexão

### CLI Test

```bash
# Ping
redis-cli ping
# PONG

# Set/Get
redis-cli set test "Hello"
redis-cli get test
# "Hello"

# Com senha
redis-cli -a your-password ping

# Conectar a host remoto
redis-cli -h redis.example.com -p 6379 -a password
```

### Node.js Test

```typescript
import { getRedis } from './lib/redis';

async function testRedis() {
  const redis = getRedis();

  // 1. Ping
  console.log(await redis.ping()); // PONG

  // 2. Set/Get
  await redis.set('test', 'Hello Redis');
  console.log(await redis.get('test')); // Hello Redis

  // 3. Increment
  await redis.incr('counter');
  console.log(await redis.get('counter')); // 1

  // 4. Expire
  await redis.setex('temp', 60, 'expires in 60s');
  console.log(await redis.ttl('temp')); // 60

  // 5. Hash
  await redis.hset('user:1', 'name', 'John');
  console.log(await redis.hget('user:1', 'name')); // John

  console.log('✅ All Redis tests passed');
}

testRedis();
```

---

## Troubleshooting

### Problema: Connection refused

```bash
# Verificar se Redis está rodando
redis-cli ping
# Se erro: Could not connect

# Start Redis
docker start lockdown-redis

# Ou
redis-server
```

### Problema: NOAUTH Authentication required

```bash
# Adicione senha no .env
REDIS_PASSWORD=your-password

# Ou no CLI
redis-cli -a your-password
```

### Problema: Redis muito lento

```bash
# Ver slow log
redis-cli SLOWLOG GET 10

# Ver latência
redis-cli --latency

# Ver stats
redis-cli INFO stats
```

### Problema: Memory full

```bash
# Ver uso de memória
redis-cli INFO memory

# Limpar banco 0 (CUIDADO!)
redis-cli -n 0 FLUSHDB

# Limpar tudo (MUITO CUIDADO!)
redis-cli FLUSHALL
```

---

## Scripts Úteis

### Start Redis (Development)

**`scripts/redis-start.sh`**

```bash
#!/bin/bash
docker run -d \
  --name lockdown-redis \
  -p 6379:6379 \
  redis:alpine \
  redis-server --appendonly yes

echo "✅ Redis started on port 6379"
```

### Stop Redis

**`scripts/redis-stop.sh`**

```bash
#!/bin/bash
docker stop lockdown-redis
docker rm lockdown-redis

echo "✅ Redis stopped"
```

### Redis Monitor

**`scripts/redis-monitor.sh`**

```bash
#!/bin/bash
docker exec -it lockdown-redis redis-cli MONITOR
```

---

## package.json Scripts

Adicione em `packages/backend/package.json`:

```json
{
  "scripts": {
    "redis:start": "bash ../../scripts/redis-start.sh",
    "redis:stop": "bash ../../scripts/redis-stop.sh",
    "redis:cli": "docker exec -it lockdown-redis redis-cli",
    "redis:monitor": "bash ../../scripts/redis-monitor.sh",
    "redis:logs": "docker logs -f lockdown-redis"
  }
}
```

---

## Próximos Passos

Agora que o Redis está configurado, veja:
- [003-PUBSUB.md](./003-PUBSUB.md) - Implementar Pub/Sub
- [004-CACHING.md](./004-CACHING.md) - Implementar cache
- [005-PRODUCTION.md](./005-PRODUCTION.md) - Deploy em produção

---

**Redis configurado e pronto para uso!** 🔴
