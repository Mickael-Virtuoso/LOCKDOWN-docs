# Redis - Pub/Sub (Event Bus)

## Visão Geral

Redis Pub/Sub é usado no LOCKDOWN como **Event Bus** para comunicação em tempo real entre diferentes partes do sistema (bots, backend, workers).

---

## Arquitetura Event-Driven

```
┌─────────────┐                  ┌─────────────┐
│ Discord Bot │──┐            ┌──│  Backend    │
└─────────────┘  │            │  └─────────────┘
                 │            │
┌─────────────┐  │  ┌──────┐  │  ┌─────────────┐
│ Telegram Bot│──┼─►│REDIS │◄─┼──│  Workers    │
└─────────────┘  │  │PUB/SUB│  │  └─────────────┘
                 │  └──────┘  │
┌─────────────┐  │            │  ┌─────────────┐
│   Webhook   │──┘            └──│  Frontend   │
└─────────────┘                  └─────────────┘

Publishers ──► Channels ──► Subscribers
```

---

## Conceitos Básicos

### Publisher (Publicador)

Envia mensagens para um **channel** (canal).

```typescript
await redis.publish('channel-name', 'message-data');
```

### Subscriber (Assinante)

Escuta mensagens de um **channel**.

```typescript
redis.subscribe('channel-name');

redis.on('message', (channel, message) => {
  console.log(`${channel}: ${message}`);
});
```

### Channels (Canais)

Tópicos onde mensagens são enviadas.

```
moderation:action
moderation:log
user:joined
user:left
message:created
message:deleted
```

---

## Implementação no LOCKDOWN

### 1. Event Publisher Service

**`packages/backend/src/services/event-publisher.ts`**

```typescript
import { getRedis } from '../lib/redis';
import { logger } from '../lib/logger';

export interface Event {
  type: string;
  payload: any;
  timestamp: string;
  source: string;
}

export class EventPublisher {
  private redis = getRedis();

  async publish(channel: string, event: Event): Promise<number> {
    try {
      const message = JSON.stringify(event);
      const subscribers = await this.redis.publish(channel, message);

      logger.info(`Event published to ${channel}`, {
        type: event.type,
        subscribers
      });

      return subscribers;
    } catch (error) {
      logger.error('Failed to publish event', { channel, error });
      throw error;
    }
  }

  // Publish moderation action
  async publishModerationAction(data: {
    action: string;
    userId: string;
    guildId: string;
    moderatorId: string;
    reason?: string;
  }): Promise<void> {
    await this.publish('moderation:action', {
      type: 'MODERATION_ACTION',
      payload: data,
      timestamp: new Date().toISOString(),
      source: 'backend'
    });
  }

  // Publish user event
  async publishUserEvent(type: string, userId: string, data: any): Promise<void> {
    await this.publish(`user:${type}`, {
      type: `USER_${type.toUpperCase()}`,
      payload: { userId, ...data },
      timestamp: new Date().toISOString(),
      source: 'backend'
    });
  }
}

export const eventPublisher = new EventPublisher();
```

### 2. Event Subscriber Service

**`packages/backend/src/services/event-subscriber.ts`**

```typescript
import Redis from 'ioredis';
import { createRedisClient } from '../lib/redis';
import { logger } from '../lib/logger';
import type { Event } from './event-publisher';

type EventHandler = (event: Event) => void | Promise<void>;

export class EventSubscriber {
  private subscriber: Redis;
  private handlers = new Map<string, EventHandler[]>();

  constructor() {
    // Criar conexão separada para subscriber
    this.subscriber = createRedisClient();
    this.setupListeners();
  }

  private setupListeners(): void {
    this.subscriber.on('message', async (channel: string, message: string) => {
      try {
        const event: Event = JSON.parse(message);
        const handlers = this.handlers.get(channel) || [];

        logger.info(`Event received from ${channel}`, {
          type: event.type,
          source: event.source
        });

        // Execute all handlers
        await Promise.all(
          handlers.map(handler => handler(event))
        );
      } catch (error) {
        logger.error('Error processing event', { channel, error });
      }
    });

    this.subscriber.on('subscribe', (channel, count) => {
      logger.info(`Subscribed to ${channel} (total: ${count})`);
    });

    this.subscriber.on('unsubscribe', (channel, count) => {
      logger.info(`Unsubscribed from ${channel} (total: ${count})`);
    });
  }

  // Subscribe to channel
  subscribe(channel: string, handler: EventHandler): void {
    const handlers = this.handlers.get(channel) || [];
    handlers.push(handler);
    this.handlers.set(channel, handlers);

    // Subscribe to Redis channel
    this.subscriber.subscribe(channel);
  }

  // Unsubscribe from channel
  unsubscribe(channel: string): void {
    this.handlers.delete(channel);
    this.subscriber.unsubscribe(channel);
  }

  // Close subscriber
  async close(): Promise<void> {
    await this.subscriber.quit();
  }
}

export const eventSubscriber = new EventSubscriber();
```

### 3. Uso no Backend

**`packages/backend/src/events/handlers.ts`**

```typescript
import { eventSubscriber } from '../services/event-subscriber';
import { logger } from '../lib/logger';
import { db } from '../db';

export function setupEventHandlers() {
  // Handle moderation actions
  eventSubscriber.subscribe('moderation:action', async (event) => {
    const { action, userId, guildId, moderatorId, reason } = event.payload;

    // Save to database
    await db.insert(moderationLogs).values({
      action,
      userId,
      guildId,
      moderatorId,
      reason,
      timestamp: new Date()
    });

    logger.info('Moderation action logged', event.payload);
  });

  // Handle user joins
  eventSubscriber.subscribe('user:joined', async (event) => {
    const { userId, guildId } = event.payload;

    // Update analytics
    await db.insert(userActivity).values({
      userId,
      guildId,
      type: 'JOIN',
      timestamp: new Date()
    });

    logger.info('User joined', event.payload);
  });

  // Handle message events
  eventSubscriber.subscribe('message:created', async (event) => {
    const { messageId, content, authorId } = event.payload;

    // Process message for moderation
    // Run through policy engine
    // etc...

    logger.info('Message created', event.payload);
  });
}
```

### 4. Uso no Bot Discord

**`packages/lockdown-bot/src/events/message.ts`**

```typescript
import { Client, Message } from 'discord.js';
import { eventPublisher } from '../services/event-publisher';

export function setupMessageEvents(client: Client) {
  client.on('messageCreate', async (message: Message) => {
    if (message.author.bot) return;

    // Publish event to Redis
    await eventPublisher.publish('message:created', {
      type: 'MESSAGE_CREATED',
      payload: {
        messageId: message.id,
        content: message.content,
        authorId: message.author.id,
        guildId: message.guildId,
        channelId: message.channelId
      },
      timestamp: new Date().toISOString(),
      source: 'discord-bot'
    });
  });
}
```

---

## Channels Padrão no LOCKDOWN

### Moderation Channels

```typescript
'moderation:action'    // Ban, kick, warn, mute
'moderation:log'       // Logs de moderação
'moderation:automod'   // Ações automáticas
```

### User Channels

```typescript
'user:joined'     // Usuário entrou no servidor
'user:left'       // Usuário saiu do servidor
'user:updated'    // Perfil atualizado
'user:banned'     // Usuário banido
```

### Message Channels

```typescript
'message:created'   // Nova mensagem
'message:deleted'   // Mensagem deletada
'message:edited'    // Mensagem editada
```

### Guild Channels

```typescript
'guild:config'      // Configuração mudou
'guild:update'      // Guild atualizada
```

---

## Patterns Úteis

### Pattern Matching (Glob)

```typescript
// Subscribe to all moderation channels
redis.psubscribe('moderation:*');

redis.on('pmessage', (pattern, channel, message) => {
  console.log(`${pattern} → ${channel}: ${message}`);
});
```

### Request-Response Pattern

```typescript
import { v4 as uuid } from 'uuid';

// Requester
async function request(channel: string, data: any): Promise<any> {
  const correlationId = uuid();
  const responseChannel = `response:${correlationId}`;

  return new Promise((resolve) => {
    // Subscribe to response
    redis.subscribe(responseChannel);

    redis.on('message', (ch, message) => {
      if (ch === responseChannel) {
        redis.unsubscribe(responseChannel);
        resolve(JSON.parse(message));
      }
    });

    // Publish request
    redis.publish(channel, JSON.stringify({
      correlationId,
      responseChannel,
      data
    }));

    // Timeout after 5s
    setTimeout(() => {
      redis.unsubscribe(responseChannel);
      resolve({ error: 'Timeout' });
    }, 5000);
  });
}

// Responder
redis.subscribe('request:channel');

redis.on('message', async (channel, message) => {
  const { correlationId, responseChannel, data } = JSON.parse(message);

  // Process request
  const result = await processRequest(data);

  // Send response
  await redis.publish(responseChannel, JSON.stringify(result));
});
```

---

## Performance e Limites

### Throughput

```
Redis Pub/Sub pode processar:
- 100,000+ mensagens/segundo
- Latência < 1ms
```

### Limitações

```
❌ Mensagens não são persistidas
   - Se subscriber offline, perde mensagens

❌ No acknowledgment
   - Não sabe se subscriber recebeu

❌ No message queue
   - Mensagens são instantâneas, não ficam em fila
```

### Quando NÃO Usar Pub/Sub

Para **garantias de entrega**, use:
- **Redis Streams** (garantia de entrega)
- **RabbitMQ** (message queue robusto)
- **Kafka** (event streaming)

---

## Monitoring

### Ver subscribers ativos

```bash
redis-cli PUBSUB NUMSUB channel-name
```

### Ver todos os channels

```bash
redis-cli PUBSUB CHANNELS
```

### Monitorar mensagens (debug)

```bash
redis-cli MONITOR
```

---

## Testing

### Test Publisher

```typescript
import { eventPublisher } from './services/event-publisher';

async function testPublisher() {
  await eventPublisher.publish('test:channel', {
    type: 'TEST_EVENT',
    payload: { message: 'Hello Pub/Sub!' },
    timestamp: new Date().toISOString(),
    source: 'test'
  });

  console.log('✅ Event published');
}
```

### Test Subscriber

```typescript
import { eventSubscriber } from './services/event-subscriber';

eventSubscriber.subscribe('test:channel', (event) => {
  console.log('Received event:', event);
});

console.log('✅ Listening for events...');
```

---

## Troubleshooting

### Eventos não chegam

```typescript
// Verificar se subscriber está conectado
const channels = await redis.pubsub('CHANNELS');
console.log('Active channels:', channels);

// Verificar subscribers
const count = await redis.pubsub('NUMSUB', 'channel-name');
console.log('Subscribers:', count);
```

### Múltiplos subscribers recebem duplicado

Isso é **normal** no Pub/Sub. Todos os subscribers recebem todas as mensagens.

---

## Próximos Passos

- [004-CACHING.md](./004-CACHING.md) - Implementar cache
- [../../02-platforms/002-EVENTS.md](../../02-platforms/002-EVENTS.md) - Event protocol

---

**Eventos em tempo real com Redis Pub/Sub!** 🔴
