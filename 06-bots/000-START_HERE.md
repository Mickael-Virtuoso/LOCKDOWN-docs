# 🤖 Bot Development Guide

## Visão Geral

Documentação completa para desenvolvimento de bots Discord no LOCKDOWN usando o bot framework customizado.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-FRAMEWORK.md](./001-FRAMEWORK.md)** ⭐ **COMECE AQUI!**
   - Bot framework architecture
   - Como o framework funciona
   - Estrutura de bots
   - Lifecycle e hooks

2. **[002-COMMANDS.md](./002-COMMANDS.md)**
   - Sistema de comandos
   - Como criar novos comandos
   - Slash commands vs prefix commands
   - Command options e validation

3. **[003-EVENTS.md](./003-EVENTS.md)**
   - Event handlers
   - Discord events
   - Custom events
   - Event priority

4. **[004-GUARDS.md](./004-GUARDS.md)**
   - Sistema de permissões
   - Guards e decorators
   - Role-based access
   - Custom guards

5. **[005-BOTS.md](./005-BOTS.md)**
   - Índice completo
   - Referência de comandos
   - Best practices
   - Troubleshooting

---

## 🎯 Framework Features

```
✅ Decorators (@Command, @Event, @Guard)
✅ Dependency Injection
✅ Type-safe commands
✅ Automatic command registration
✅ Permission system
✅ Event-driven architecture
```

---

## 🚀 Quick Start

```bash
# Setup bot
cd packages/lockdown-bot
pnpm install

# Configure .env
cp .env.example .env
# Adicione seu DISCORD_TOKEN

# Start bot
pnpm dev

# Bot online no Discord!
```

---

## 📁 Estrutura de um Bot

```
packages/lockdown-bot/
├── src/
│   ├── commands/     - Slash commands
│   ├── events/       - Event handlers
│   ├── guards/       - Permission guards
│   ├── services/     - Business logic
│   └── utils/        - Utilities
└── tests/            - Tests
```

---

## 💡 Criando Seu Primeiro Comando

```typescript
import { Command, Guard } from '@lockdown/bot-framework';

@Command({
  name: 'hello',
  description: 'Say hello!'
})
@Guard('ADMIN')
export class HelloCommand {
  async execute(interaction: CommandInteraction) {
    await interaction.reply('Hello, world!');
  }
}
```

---

## 🔒 Sistema de Permissões

Guards controlam quem pode usar comandos:
- `@Guard('ADMIN')` - Apenas admins
- `@Guard('MODERATOR')` - Moderadores e admins
- `@Guard('EVERYONE')` - Todos (padrão)

---

## 🔗 Documentação Relacionada

- **[../02-platforms/](../02-platforms/)** - Event protocol
- **[../08-api/](../08-api/)** - Backend API integration
- **[../03-core/](../03-core/)** - Policy engine

---

## 📚 Recursos

- [Discord.js Documentation](https://discord.js.org/)
- [Discord Developer Portal](https://discord.com/developers/docs)

---

**Construa bots poderosos e modulares!** 🤖
