# Bot Commands

Sistema de comandos slash do LOCKDOWN.

---

## Estrutura de Comando

```typescript
import { Command } from '@bot-framework/decorators';
import { BaseCommand } from '@bot-framework/base';
import { ChatInputCommandInteraction } from 'discord.js';

@Command({
  name: 'ban',
  description: 'Ban a user from the server',
  category: 'moderation',
  options: [
    {
      name: 'user',
      description: 'User to ban',
      type: 'USER',
      required: true,
    },
    {
      name: 'reason',
      description: 'Reason for ban',
      type: 'STRING',
      required: false,
    },
  ],
})
export class BanCommand extends BaseCommand {
  async execute(interaction: ChatInputCommandInteraction) {
    const user = interaction.options.getUser('user', true);
    const reason = interaction.options.getString('reason') || 'No reason';

    // Lógica do ban
  }
}
```

---

## Categorias

| Categoria | Descrição |
|-----------|-----------|
| `moderation` | Ban, kick, mute, warn |
| `config` | Configurações do servidor |
| `utility` | Comandos utilitários |
| `info` | Informações do bot/server |

---

## Opções de Comando

| Tipo | Uso |
|------|-----|
| `STRING` | Texto livre |
| `INTEGER` | Número inteiro |
| `BOOLEAN` | Verdadeiro/falso |
| `USER` | Menção de usuário |
| `CHANNEL` | Menção de canal |
| `ROLE` | Menção de role |

---

## Comandos de Moderação

### /ban

```
/ban user:@User reason:Spam
```

### /kick

```
/kick user:@User reason:Disruptive
```

### /mute

```
/mute user:@User duration:60 reason:Spam
```

### /warn

```
/warn user:@User reason:First warning
```

---

## Subcomandos

```typescript
@Command({
  name: 'config',
  description: 'Server configuration',
  subcommands: [
    {
      name: 'logs',
      description: 'Set log channel',
    },
    {
      name: 'roles',
      description: 'Configure auto-roles',
    },
  ],
})
export class ConfigCommand extends BaseCommand {
  async execute(interaction: ChatInputCommandInteraction) {
    const subcommand = interaction.options.getSubcommand();

    switch (subcommand) {
      case 'logs':
        // ...
        break;
      case 'roles':
        // ...
        break;
    }
  }
}
```

---

## Links Relacionados

- [Framework](./FRAMEWORK.md)
- [Guards](./GUARDS.md)
