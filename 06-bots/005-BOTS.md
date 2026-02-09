# Bots - LOCKDOWN

Documentação completa dos bots Discord da plataforma LOCKDOWN.

---

## Índice

| Documento | Descrição |
|-----------|-----------|
| [Framework](./FRAMEWORK.md) | Bot framework base |
| [Commands](./COMMANDS.md) | Sistema de comandos |
| [Events](./EVENTS.md) | Handlers de eventos |
| [Guards](./GUARDS.md) | Permissões e guards |

---

## Arquitetura

```
packages/
├─ bot-framework/     # Framework base reutilizável
├─ lockdown-bot/      # Bot principal de moderação
└─ utility-bot/       # Bot utilitário
```

---

## Stack

| Tecnologia | Propósito |
|------------|-----------|
| Discord.js | Discord API |
| TypeScript | Type safety |
| Decorators | Metaprogramação |
| Pino | Logging |

---

## Quick Start

```bash
# Iniciar bot principal
cd packages/lockdown-bot
pnpm dev

# Iniciar utility bot
cd packages/utility-bot
pnpm dev
```

---

## Referências

- [Bot Framework](../../packages/bot-framework/README.md)
- [Lockdown Bot](../../packages/lockdown-bot/README.md)
- [Events](../02-platforms/EVENTS.md)
