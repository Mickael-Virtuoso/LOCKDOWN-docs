# 🏗️ Arquitetura do LOCKDOWN

## Visão Geral

Esta pasta contém a documentação completa sobre a arquitetura e estrutura do sistema LOCKDOWN.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-ARCHITECTURE.md](./001-ARCHITECTURE.md)** ⭐ **COMECE AQUI!**
   - Visão geral da arquitetura
   - Design patterns utilizados
   - Fluxo de dados
   - Componentes principais

2. **[002-MONOREPO.md](./002-MONOREPO.md)**
   - Estrutura do monorepo
   - pnpm workspaces
   - Turborepo configuration
   - Gerenciamento de dependências

3. **[003-DIRECTORIES.md](./003-DIRECTORIES.md)**
   - Estrutura de diretórios completa
   - Organização de arquivos
   - Convenções de nomenclatura
   - Localização de recursos

4. **[004-SERVICES.md](./004-SERVICES.md)**
   - Camada de serviços
   - Service layer architecture
   - Comunicação entre serviços
   - Event-driven architecture

5. **[005-DEPLOYMENT.md](./005-DEPLOYMENT.md)**
   - Estratégias de deploy
   - Ambientes (dev, staging, prod)
   - CI/CD pipeline
   - Docker e containerização

---

## 🎯 O Que Você Vai Aprender

- ✅ Como o sistema é estruturado
- ✅ Decisões arquiteturais e trade-offs
- ✅ Padrões de design implementados
- ✅ Comunicação entre componentes
- ✅ Estratégias de deploy

---

## 🧩 Componentes Principais

```
LOCKDOWN
├── Backend (Fastify + Drizzle)
├── Frontend (React + Vite)
├── Bots (Discord.js)
├── Shared (Types & Utils)
└── Tests (E2E & Integration)
```

---

## 📊 Diagramas

Os arquivos nesta pasta contêm diagramas Mermaid mostrando:
- Fluxo de dados entre componentes
- Arquitetura de microserviços
- Event-driven communication
- Database relationships

---

## 💡 Dica para Arquitetos

Se você está revisando a arquitetura ou planejando mudanças significativas, leia todos os documentos nesta pasta para entender o contexto completo das decisões atuais.

---

**Entenda a base antes de construir!** 🏗️
