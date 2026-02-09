# 📡 EVENTS - LOCKDOWN

## Redis Pub/Sub, Event Publishing e Real-time Synchronization

---

## 📖 Índice

1. [Visão Geral](#visão-geral)
2. [Arquitetura de Eventos](#arquitetura-de-eventos)
3. [Publishers](#publishers)
4. [Subscribers](#subscribers)
5. [Event Manager](#event-manager)
6. [Event Types](#event-types)
7. [Tratamento de Erros](#tratamento-de-erros)
8. [Monitoring](#monitoring)

---

## 📋 Visão Geral

### O que são Eventos?

Mensagens assíncronas publicadas quando algo importante acontece:

```
Backend cria ban
    ↓
Publica evento: "ban:created"
    ↓
Redis recebe e armazena
    ↓
Bot e outros subscribers recebem
    ↓
Executam ações (log, mensagem Discord, cache)
```

### Benefícios

✅ Real-time updates
✅ Loose coupling (serviços independentes)
✅ Escalabilidade (múltiplos subscribers)
✅ Resiliência (Redis persiste mensagens)
✅ Auditoria (histórico de eventos)

---

## 🏗️ Arquitetura de Eventos

### Event Flow

```
Backend Service
    ↓
await banService.createBan(data)
    ↓
Service Logic + Validações
    ↓
INSERT INTO bans (...)
    ↓
Publish Event: redis.publish('ban:created', event)
    ↓
Redis Broadcasting
    ↓
Bot, Cache, Audit recebem
    ↓
Cada um processa independentemente
```

### Channels Disponíveis

```
ban:created           → Ban criado
ban:removed           → Ban removido
kick:executed         → Kick executado
mute:applied          → Mute aplicado
warning:added         → Aviso adicionado
audit:logged          → Log de auditoria
```

---

## 📢 Publishers

### BanPublisher Implementation

```typescript
import { redis } from '@config/redis';
import { logger } from '@config/logger';
import { v4 as uuid } from 'uuid';

export default class BanPublisher {
  async publishBanCreated(ban: any): Promise<void> {
    logger.debug(`[${this.constructor.name}] "📢 publishing ban:created"`);

    const event = {
      eventId: uuid(),
      eventType: 'ban:created',
      timestamp: new Date().toISOString(),
      data: {
        id: ban.id,
        guildId: ban.guildId,
        userId: ban.userId,
        reason: ban.reason,
        moderatorId: ban.moderatorId,
        expiresAt: ban.expiresAt?.toISOString() ?? null,
      },
    };

    try {
      const numSubscribers = await redis.publish('ban:created', JSON.stringify(event));

      logger.info(`[${this.constructor.name}] "✅ published to ${numSubscribers} subscribers"`);
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ failed to publish: ${error}"`);
      throw error;
    }
  }

  async publishBanRemoved(ban: any): Promise<void> {
    logger.debug(`[${this.constructor.name}] "📢 publishing ban:removed"`);

    const event = {
      eventId: uuid(),
      eventType: 'ban:removed',
      timestamp: new Date().toISOString(),
      data: {
        id: ban.id,
        guildId: ban.guildId,
        userId: ban.userId,
      },
    };

    try {
      await redis.publish('ban:removed', JSON.stringify(event));
      logger.info(`[${this.constructor.name}] "✅ ban:removed published"`);
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ failed to publish: ${error}"`);
      throw error;
    }
  }
}
```

---

## 👂 Subscribers

### Bot Subscriber

```typescript
import { Client, EmbedBuilder } from 'discord.js';
import { redis } from '@config/redis';
import { logger } from '@config/logger';

export default {
  name: 'redisEvents',

  async execute(client: Client): Promise<void> {
    logger.info(`[${this.name}] "🔴 subscribing to Redis events"`);

    const subscriber = redis.duplicate();

    const channels = [
      'ban:created',
      'ban:removed',
      'kick:executed',
      'mute:applied',
      'warning:added',
    ];

    await subscriber.subscribe(channels);

    subscriber.on('message', async (channel, message) => {
      try {
        const event = JSON.parse(message);

        if (channel === 'ban:created') {
          await handleBanCreated(client, event);
        }
      } catch (error) {
        logger.error(`[${this.name}] "❌ error: ${error}"`);
      }
    });
  },
};

async function handleBanCreated(client: Client, event: any) {
  const guild = client.guilds.cache.get(event.data.guildId);
  if (!guild) return;

  const logsChannel = guild.channels.cache.find((ch) => ch.name === 'mod-logs' && ch.isTextBased());

  if (!logsChannel) return;

  const embed = new EmbedBuilder()
    .setColor('#ff0000')
    .setTitle('🚫 User Banned')
    .addFields(
      { name: 'User ID', value: event.data.userId },
      { name: 'Reason', value: event.data.reason || 'No reason' }
    )
    .setTimestamp();

  await logsChannel.send({ embeds: [embed] });
}
```

---

## 🎛️ Event Manager

```typescript
import { redis } from '@config/redis';
import { logger } from '@config/logger';

export default class EventManager {
  async healthCheck(): Promise<boolean> {
    try {
      const response = await redis.ping();
      return response === 'PONG';
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ health check failed: ${error}"`);
      return false;
    }
  }

  async getStats(): Promise<{ channels: number; events: number }> {
    try {
      const eventKeys = await redis.keys('event:*');
      const channelInfo = await redis.execute(['PUBSUB', 'CHANNELS']);

      return {
        channels: (channelInfo as any[]).length,
        events: eventKeys.length,
      };
    } catch (error) {
      logger.error(`[${this.constructor.name}] "❌ failed to get stats: ${error}"`);
      return { channels: 0, events: 0 };
    }
  }
}
```

---

## 📋 Event Types

```typescript
export interface BanCreatedEvent {
  eventId: string;
  eventType: 'ban:created';
  timestamp: string;
  data: {
    id: string;
    guildId: string;
    userId: string;
    reason: string | null;
    moderatorId: string;
    expiresAt: string | null;
  };
}

export interface KickExecutedEvent {
  eventId: string;
  eventType: 'kick:executed';
  timestamp: string;
  data: {
    id: string;
    guildId: string;
    userId: string;
    reason: string | null;
  };
}

export type Event = BanCreatedEvent | KickExecutedEvent;
```

---

## ❌ Tratamento de Erros

### Retry com Backoff

```typescript
async function publishWithRetry(
  channel: string,
  message: string,
  maxRetries: number = 3
): Promise<void> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      await redis.publish(channel, message);
      logger.info(`✅ Published to ${channel}`);
      return;
    } catch (error) {
      const delay = Math.pow(2, attempt) * 1000;
      logger.warn(`⚠️ Attempt ${attempt} failed, retrying in ${delay}ms`);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }

  throw new Error(`Failed to publish after ${maxRetries} attempts`);
}
```

### Dead Letter Queue

```typescript
async function handleFailedEvent(event: any, error: Error): Promise<void> {
  const dlqKey = `dlq:${event.eventId}`;
  const dlqData = {
    event,
    error: error.message,
    timestamp: new Date().toISOString(),
  };

  await redis.setex(dlqKey, 24 * 60 * 60, JSON.stringify(dlqData));
  logger.info(`📦 Event moved to DLQ: ${dlqKey}`);
}
```

---

## 📊 Monitoring

### Health Check

```bash
# Check Redis
redis-cli ping

# Subscribe a um canal
redis-cli SUBSCRIBE ban:created

# Ver histórico
redis-cli KEYS event:*
```

---

## 📚 Referências

- [BACKEND.md](./BACKEND.md) - Service layer
- [BOT.md](./BOT.md) - Event handling em Discord
- [Redis Pub/Sub](https://redis.io/docs/latest/develop/interact/pubsub/)

---

**Happy eventing!** 📡
