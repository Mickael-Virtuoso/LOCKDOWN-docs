# Bot Guards

Sistema de permissões e guards no LOCKDOWN.

---

## Guard Decorator

```typescript
import { Guard } from '@bot-framework/decorators';
import { PermissionGuard } from '@bot-framework/middleware/guards';

@Guard(PermissionGuard(['MANAGE_MESSAGES']))
@Command({ name: 'clear' })
export class ClearCommand extends BaseCommand {
  // Apenas usuários com MANAGE_MESSAGES podem usar
}
```

---

## Guards Disponíveis

### PermissionGuard

Verifica permissões Discord.

```typescript
@Guard(PermissionGuard(['BAN_MEMBERS', 'KICK_MEMBERS']))
```

### RoleGuard

Verifica roles específicas.

```typescript
@Guard(RoleGuard(['Moderator', 'Admin']))
```

### OwnerGuard

Apenas o dono do servidor.

```typescript
@Guard(OwnerGuard())
```

### GuildGuard

Apenas em servidores (não DMs).

```typescript
@Guard(GuildGuard())
```

---

## Combinar Guards

```typescript
@Guard(GuildGuard())
@Guard(PermissionGuard(['MANAGE_GUILD']))
@Command({ name: 'config' })
export class ConfigCommand extends BaseCommand {
  // Precisa estar em guild E ter MANAGE_GUILD
}
```

---

## Custom Guard

```typescript
import { BaseMiddleware } from '@bot-framework/base';
import { ChatInputCommandInteraction } from 'discord.js';

export class PremiumGuard extends BaseMiddleware {
  async execute(interaction: ChatInputCommandInteraction): Promise<boolean> {
    const guild = interaction.guild;
    const isPremium = await checkPremiumStatus(guild.id);

    if (!isPremium) {
      await interaction.reply({
        content: 'This command requires premium!',
        ephemeral: true,
      });
      return false;
    }

    return true;
  }
}

// Uso
@Guard(PremiumGuard)
@Command({ name: 'premium-feature' })
export class PremiumCommand extends BaseCommand {}
```

---

## Error Handling

```typescript
import { GuardError } from '@bot-framework/error-handling';

// Guard falhou
try {
  await command.execute(interaction);
} catch (error) {
  if (error instanceof GuardError) {
    await interaction.reply({
      content: error.message,
      ephemeral: true,
    });
  }
}
```

---

## Links Relacionados

- [Framework](./FRAMEWORK.md)
- [Commands](./COMMANDS.md)
