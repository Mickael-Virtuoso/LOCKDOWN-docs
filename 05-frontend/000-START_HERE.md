# 🎨 Frontend Development Guide

## Visão Geral

Documentação completa para desenvolvimento frontend no LOCKDOWN usando React, Vite e Tailwind CSS.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - Visão geral do frontend
   - Stack tecnológica (React, Vite, Tailwind)
   - Arquitetura de componentes
   - Responsabilidades do frontend

2. **[002-SETUP.md](./002-SETUP.md)**
   - Configuração do ambiente
   - Instalação e dependências
   - Variáveis de ambiente
   - Dev server

3. **[003-STRUCTURE.md](./003-STRUCTURE.md)**
   - Estrutura de pastas
   - Organização de componentes
   - Convenções de nomenclatura
   - Atomic Design (atoms, molecules, organisms)

4. **[004-COMPONENTS.md](./004-COMPONENTS.md)**
   - Como criar componentes React
   - Props e TypeScript
   - Hooks customizados
   - Component composition

5. **[005-STATE.md](./005-STATE.md)**
   - State management
   - Context API
   - Zustand (se aplicável)
   - Data fetching

6. **[006-API.md](./006-API.md)**
   - Integração com backend
   - Axios/Fetch configuration
   - Error handling
   - Authentication headers

7. **[007-STYLING.md](./007-STYLING.md)**
   - Tailwind CSS
   - Design system
   - Temas e cores
   - Responsive design

8. **[008-DEPLOYMENT.md](./008-DEPLOYMENT.md)**
   - Build para produção
   - Otimizações
   - Deploy (Vercel, Netlify, etc)
   - Environment variables

9. **[009-FRONTEND.md](./009-FRONTEND.md)**
   - Índice completo
   - Referência rápida
   - Recursos externos

---

## 🎯 Stack Tecnológica

```
├── React 18      - UI Library
├── TypeScript    - Type safety
├── Vite          - Build tool
├── Tailwind CSS  - Styling
├── React Router  - Routing
└── Axios         - HTTP client
```

---

## 🚀 Quick Start

```bash
# Setup frontend
cd packages/frontend
pnpm install

# Configure .env
cp .env.example .env

# Start dev server
pnpm dev

# Frontend running on http://localhost:5173
```

---

## 📁 Estrutura Resumida

```
packages/frontend/
├── src/
│   ├── components/   - React components
│   ├── pages/        - Page components
│   ├── hooks/        - Custom hooks
│   ├── services/     - API services
│   ├── styles/       - Global styles
│   └── utils/        - Utilities
└── public/           - Static assets
```

---

## 💡 Para Novos Frontend Developers

1. Leia [001-OVERVIEW.md](./001-OVERVIEW.md) para entender a stack
2. Configure com [002-SETUP.md](./002-SETUP.md)
3. Entenda a estrutura em [003-STRUCTURE.md](./003-STRUCTURE.md)
4. Crie componentes seguindo [004-COMPONENTS.md](./004-COMPONENTS.md)
5. Estilize com [007-STYLING.md](./007-STYLING.md)

---

## 🎨 Design System

O projeto utiliza um design system baseado em Tailwind CSS com:
- Cores customizadas
- Componentes reutilizáveis
- Temas (light/dark)
- Responsive breakpoints

---

## 🔗 Documentação Relacionada

- **[../08-api/](../08-api/)** - Endpoints disponíveis
- **[../09-security/](../09-security/)** - Auth no frontend

---

**Crie interfaces incríveis!** 🎨
