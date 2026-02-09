# Bot Events

Sistema de eventos Discord no LOCKDOWN.

---

## Estrutura de Evento

```typescript
import { Event } from '@bot-framework/decorators';
import { BaseEvent } from '@bot-framework/base';
import { Message } from 'discord.js';

@Event('messageCreate')
export class MessageCreateHandler extends BaseEvent {
  async execute(message: Message) {
    if (message.author.bot) return;

    // Lógica do evento
  }
}
```

---

## Eventos Principais

### Guild Events

| Evento | Descrição |
|--------|-----------|
| `guildCreate` | Bot adicionado a servidor |
| `guildDelete` | Bot removido de servidor |
| `guildMemberAdd` | Novo membro entrou |
| `guildMemberRemove` | Membro saiu/foi removido |
| `guildBanAdd` | Usuário banido |
| `guildBanRemove` | Usuário desbanido |

### Message Events

| Evento | Descrição |
|--------|-----------|
| `messageCreate` | Nova mensagem |
| `messageDelete` | Mensagem deletada |
| `messageUpdate` | Mensagem editada |

### Interaction Events

| Evento | Descrição |
|--------|-----------|
| `interactionCreate` | Slash command/button |

---

## Exemplos

### Guild Member Add

```typescript
@Event('guildMemberAdd')
export class WelcomeHandler extends BaseEvent {
  async execute(member: GuildMember) {
    // Auto-role
    const autoRoles = await getAutoRoles(member.guild.id);
    await member.roles.add(autoRoles);

    // Welcome message
    const channel = member.guild.systemChannel;
    await channel?.send(`Welcome ${member}!`);

    // Sync with backend
    await api.post('/core/guilds/:guildId/members', {
      userId: member.id,
      joinedAt: member.joinedAt,
    });
  }
}
```

### Guild Ban Add

```typescript
@Event('guildBanAdd')
export class BanSyncHandler extends BaseEvent {
  async execute(ban: GuildBan) {
    await api.post('/moderation/bans', {
      guildId: ban.guild.id,
      userId: ban.user.id,
      reason: ban.reason || 'No reason',
      moderatorId: client.user!.id,
    });
  }
}
```

---

## Intents

```typescript
const client = new ClientManager({
  intents: [
    'Guilds',
    'GuildMembers',
    'GuildBans',
    'GuildMessages',
    'MessageContent',
  ],
});
```

---

## Links Relacionados

- [Framework](./FRAMEWORK.md)
- [Events Redis](../02-platforms/EVENTS.md)
