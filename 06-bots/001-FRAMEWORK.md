# Bot Framework

Framework base para desenvolvimento de bots Discord no LOCKDOWN.

---

## Estrutura

```
packages/bot-framework/src/
├─ base/           # Classes base
├─ client/         # Client manager
├─ constants/      # Constantes
├─ decorators/     # Decoradores
├─ error-handling/ # Tratamento de erros
├─ events/         # Event handlers
├─ handlers/       # Command/Event handlers
├─ middleware/     # Guards e rate limiting
├─ plugins/        # Sistema de plugins
├─ types/          # TypeScript types
└─ utils/          # Utilitários
```

---

## Decoradores

### @Command

```typescript
import { Command } from '@bot-framework/decorators';

@Command({
  name: 'ban',
  description: 'Ban a user',
  category: 'moderation',
})
export class BanCommand extends BaseCommand {
  async execute(interaction: ChatInputCommandInteraction) {
    // ...
  }
}
```

### @Event

```typescript
import { Event } from '@bot-framework/decorators';

@Event('messageCreate')
export class MessageHandler extends BaseEvent {
  async execute(message: Message) {
    // ...
  }
}
```

### @Guard

```typescript
import { Guard, PermissionGuard } from '@bot-framework/decorators';

@Guard(PermissionGuard(['MANAGE_MESSAGES']))
@Command({ name: 'clear' })
export class ClearCommand extends BaseCommand {
  // ...
}
```

---

## Client Manager

```typescript
import { ClientManager } from '@bot-framework/client';

const client = new ClientManager({
  intents: ['Guilds', 'GuildMessages'],
  token: process.env.DISCORD_TOKEN,
});

await client.start();
```

---

## Links Relacionados

- [Commands](./COMMANDS.md)
- [Events](./EVENTS.md)
- [Guards](./GUARDS.md)
