Design e Arquitetura do Sistema
📊 Visão Geral
O Que é LOCKDOWN?

LOCKDOWN é uma plataforma central de governança para comunidades Discord, projetada para sustentar um ecossistema distribuído de bots especializados.

Ele atua como uma autoridade central benevolente, responsável por políticas, coerência e integridade, enquanto delega execução a clientes especializados (bots satélites).

🧠 Princípios Arquiteturais

Autoridade central clara

Execução distribuída

Políticas imutáveis e auditáveis

Comunicação orientada a eventos

Escalabilidade horizontal desde a base

📦 Stack Tecnológico
Camada Tecnologia Papel Arquitetural
Backend Core Express + TypeScript + Drizzle Autoridade e políticas
Bots Discord.js + TypeScript Clientes executores
Frontend React + Vite + TypeScript Interface administrativa
Database SQLite (dev) / PostgreSQL (prod) Fonte única da verdade
Cache/Eventos Redis + ioredis Backbone de comunicação
Logging Pino Observabilidade sistêmica
Monorepo pnpm + Turbo Orquestração e consistência
🎯 Arquitetura de Alto Nível
┌──────────────────────────────────────────────────────────────┐
│ LOCKDOWN CORE │
│ (Policy Engine + Backend API) │
└──────────────────────────────────────────────────────────────┘
↑ ↑
(REST + JWT) (Pub/Sub Redis)
│ │
┌───────────────────────┐ ┌──────────────────────────┐
│ Bots Satélites │ │ Serviços de Suporte │
│ (Auth, Mod, Security, │ │ Cache • Audit • Eventos │
│ Tickets, etc) │ └──────────────────────────┘
└───────────────────────┘
│
(Discord API)
│
Comunidades Discord
🧩 Camadas Internas do Backend
Controllers

Responsáveis apenas por:

Parsing de requests

Validação de entrada

Serialização de respostas

Services

Responsáveis por:

Regras de negócio

Orquestração

Interação com Policy Engine

Publicação de eventos

Repositories

Responsáveis por:

Acesso a dados

Queries SQL explícitas

Performance e integridade

Schemas

Responsáveis por:

Definições de tabelas

Relações

Tipagem forte

🔄 Arquitetura Orientada a Eventos

Redis atua como sistema nervoso do ecossistema.

Backend publica eventos

Bots satélites consomem eventos

Nenhuma dependência direta entre instâncias

Eventos são fatos, não comandos.

🛰️ Relação com Bots Satélites

Satélites são clientes do LOCKDOWN

Não possuem autoridade própria

Executam ações conforme políticas recebidas

Compartilham vocabulário e contratos

LOCKDOWN não depende de satélites para operar.

🗄️ Persistência e Consistência

PostgreSQL é a fonte definitiva de dados

SQLite é restrito a desenvolvimento local

Redis não é fonte de verdade

Toda decisão relevante é persistida.

📈 Escalabilidade

A arquitetura suporta explicitamente:

Clustering de bots

Sharding de workloads

Multi-region backend

Execução paralela de consumidores

Nada depende de estado local.

🔐 Segurança e Confiança

Autenticação via JWT

Políticas centralizadas

Audit logs imutáveis e internos

Separação clara entre decisão e execução

🧭 Governança

LOCKDOWN existe para garantir:

Coerência entre módulos

Evolução sem fragmentação

Confiança a longo prazo

Não é um bot.

É a fundação que permite que outros existam.
