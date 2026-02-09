# 🤖 Plataformas Suportadas

## Visão Geral

Esta pasta documenta as plataformas integradas ao LOCKDOWN e como elas se comunicam através do sistema de eventos.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-BOT.md](./001-BOT.md)** ⭐ **COMECE AQUI!**
   - Conceito de bots no LOCKDOWN
   - Plataformas suportadas (Discord, Telegram, etc)
   - Arquitetura de bots
   - Como adicionar novas plataformas

2. **[002-EVENTS.md](./002-EVENTS.md)**
   - Sistema de eventos (Redis Pub/Sub)
   - Event-driven architecture
   - Tipos de eventos
   - Subscribers e publishers

3. **[003-EVENT_PROTOCOL.md](./003-EVENT_PROTOCOL.md)**
   - Protocolo detalhado de eventos
   - Estrutura de mensagens
   - Payload schemas
   - Versionamento de eventos

---

## 🎯 Plataformas Atuais

- ✅ **Discord** - Totalmente implementado
- 🚧 **Telegram** - Em desenvolvimento
- 📋 **WhatsApp** - Planejado
- 📋 **Slack** - Planejado

---

## 🔄 Arquitetura Event-Driven

```
Bot Platform → Event Publisher → Redis Pub/Sub → Event Subscribers → Actions
```

Todos os bots publicam eventos padronizados que são consumidos por diferentes partes do sistema.

---

## 💡 Para Desenvolvedores de Bots

Se você vai criar um bot para uma nova plataforma:
1. Leia [001-BOT.md](./001-BOT.md) para entender a estrutura
2. Estude [003-EVENT_PROTOCOL.md](./003-EVENT_PROTOCOL.md) para implementar os eventos corretamente
3. Consulte [../06-bots/](../06-bots/) para detalhes específicos do Discord

---

**Uma arquitetura, múltiplas plataformas!** 🤖
