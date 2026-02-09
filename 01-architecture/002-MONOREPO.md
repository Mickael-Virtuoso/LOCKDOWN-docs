# 📦 MONOREPO - LOCKDOWN

## pnpm Workspaces + Turborepo - Estrutura Multi-Package

---

## 📖 Índice

1. [Visão Geral](#visão-geral)
2. [Estrutura](#estrutura)
3. [pnpm Workspaces](#pnpm-workspaces)
4. [Turborepo](#turborepo)
5. [Dependências Compartilhadas](#dependências-compartilhadas)
6. [Comandos](#comandos)
7. [Best Practices](#best-practices)

---

## 📋 Visão Geral

### O que é Monorepo?

Um repositório único contendo **múltiplos packages** (projetos) correlatos:

```
lockdown/ (repositório único)
├─ packages/backend/       (API Express)
├─ packages/lockdown-bot/  (Discord.js)
├─ packages/frontend/      (React)
├─ packages/shared/        (Tipos e utilities)
├─ tests/                  (Testes)
└─ docs/                   (Documentação)
```

### Benefícios

✅ Compartilhar código entre packages
✅ Versioning único
✅ Build otimizado com Turborepo
✅ Scripts reutilizáveis
✅ Facilita refactoring
✅ CI/CD simplificado

---

## 📁 Estrutura

### Root Structure

```
lockdown/
├─ packages/
│  ├─ backend/
│  │  ├─ src/
│  │  ├─ dist/
│  │  ├─ package.json
│  │  ├─ tsconfig.json
│  │  └─ vitest.config.ts
│  │
│  ├─ lockdown-bot/
│  │  ├─ src/
│  │  ├─ dist/
│  │  ├─ package.json
│  │  └─ tsconfig.json
│  │
│  ├─ frontend/
│  │  ├─ src/
│  │  ├─ dist/
│  │  ├─ package.json
│  │  ├─ vite.config.ts
│  │  └─ tsconfig.json
│  │
│  └─ shared/
│     ├─ src/
│     ├─ package.json
│     └─ tsconfig.json
│
├─ tests/
│  ├─ unit/
│  ├─ integration/
│  ├─ e2e/
│  ├─ package.json
│  └─ jest.config.js
│
├─ docs/
│  └─ *.md
│
├─ scripts/
│  ├─ setup.sh
│  ├─ deploy.sh
│  └─ backup.sh
│
├─ .github/
│  └─ workflows/
│     ├─ test.yml
│     └─ deploy.yml
│
├─ pnpm-workspace.yaml      # Define workspaces
├─ turbo.json               # Turborepo config
├─ tsconfig.json            # TS base
├─ eslint.config.js         # ESLint base
├─ prettier.config.js       # Prettier config
├─ package.json             # Root package
└─ .npmrc                   # npm config
```

---

## 🔧 pnpm Workspaces

### pnpm-workspace.yaml

```yaml
packages:
  - 'packages/*'
  - 'tests'
```

### Package.json Structure

**Root `package.json`:**

```json
{
  "name": "lockdown",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "engines": {
    "node": ">=20.0.0",
    "pnpm": ">=10.0.0"
  },
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "build:prod": "turbo run build --filter='!tests'",
    "lint": "turbo run lint",
    "lint:fix": "turbo run lint:fix",
    "format": "prettier --write .",
    "type-check": "turbo run type-check",
    "test": "turbo run test",
    "test:coverage": "turbo run test:coverage",
    "validate": "pnpm type-check && pnpm lint && pnpm format",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "turbo": "^2.8.0",
    "typescript": "^5.3.0"
  },
  "pnpm": {
    "peerDependencyRules": {
      "ignoreMissing": ["eslint"]
    }
  }
}
```

**Backend `package.json`:**

```json
{
  "name": "@lockdown/backend",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "test": "jest",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "@lockdown/shared": "workspace:*",
    "express": "^4.18.2",
    "drizzle-orm": "^0.30.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "typescript": "workspace:*"
  }
}
```

### Referências entre Packages

Use `workspace:*` para sempre usar versão local:

```json
{
  "dependencies": {
    "@lockdown/shared": "workspace:*"
  }
}
```

Ao publicar (npm publish), `workspace:*` é convertido automaticamente para versão real (ex: `^0.0.1`).

---

## 🚀 Turborepo

### turbo.json

```json
{
  "extends": ["//"],
  "globalDependencies": ["**/.env", "tsconfig.json"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"],
      "cache": true
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "outputs": [],
      "cache": true
    },
    "type-check": {
      "outputs": [],
      "cache": true
    },
    "test": {
      "outputs": ["coverage/**"],
      "cache": true
    }
  }
}
```

### Explained

```
build:
  dependsOn: ["^build"]    ← Compilar dependencies primeiro
  outputs: ["dist/**"]     ← Cache output
  cache: true              ← Usar cache

dev:
  cache: false             ← Não cachear dev
  persistent: true         ← Ficar rodando
```

---

## 🔗 Dependências Compartilhadas

### Shared Package

**`packages/shared/package.json`:**

```json
{
  "name": "@lockdown/shared",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "types": "./dist/index.d.ts"
    },
    "./types": {
      "import": "./dist/types/index.js",
      "types": "./dist/types/index.d.ts"
    }
  },
  "scripts": {
    "build": "tsc",
    "type-check": "tsc --noEmit"
  }
}
```

**`packages/shared/src/index.ts`:**

```typescript
/**
 * Shared utilities e types para LOCKDOWN
 */

export * from './types';
export * from './errors';
export * from './utils';
```

**`packages/shared/src/types/index.ts`:**

```typescript
// API Types
export type { User, Guild, Ban, Kick, Mute } from './api.types';

// Event Types
export type { Event, BanCreatedEvent } from './events.types';

// Store Types
export type { AuthState, BansState } from './store.types';
```

### Usar Shared em Outro Package

**Backend:**

```typescript
import { type User, AppError, validateEmail } from '@lockdown/shared';

// Usar tipos e utilities compartilhados
```

**Bot:**

```typescript
import { type Event, type BanCreatedEvent, formatDate } from '@lockdown/shared';
```

---

## 🎯 Comandos

### Install

```bash
# Instalar dependências todas
pnpm install

# Apenas um package
cd packages/backend
pnpm install

# Adicionar dependência compartilhada
pnpm add lodash -F @lockdown/backend

# Dependência de desenvolvimento
pnpm add -D typescript -F @lockdown/backend
```

### Run Scripts

```bash
# Rodar em todos os packages
pnpm run dev

# Rodar em package específico
pnpm -F @lockdown/backend run dev

# Rodar sequencialmente
pnpm --recursive run build

# Excluir packages
pnpm turbo run build --filter='!tests'
```

### Cleaning

```bash
# Limpar cache Turbo
pnpm turbo prune --docker

# Remover node_modules
pnpm clean

# Rebuild tudo
pnpm install
pnpm build
```

---

## 🎓 Best Practices

### ✅ DO

```typescript
// ✅ Importar shared quando reutilizar
import { type User, AppError } from '@lockdown/shared';

// ✅ Usar workspace:* em dependencies
{
  "dependencies": {
    "@lockdown/shared": "workspace:*"
  }
}

// ✅ Estrutura clara de exports
// packages/shared/src/index.ts
export * from './types';
export { User, Guild } from './types';

// ✅ Usar Turborepo para paralelizar
pnpm turbo run build  // Compila packages em paralelo
```

### ❌ DON'T

```typescript
// ❌ Circular dependencies
// packages/backend → @lockdown/shared → @lockdown/backend

// ❌ Misturar packages
// backend/src/api/frontend/components.ts

// ❌ Versão fixa em workspace
{
  "dependencies": {
    "@lockdown/shared": "^0.0.1"  // ❌ Use "workspace:*"
  }
}

// ❌ Cache em dev
{
  "dev": {
    "cache": true  // ❌ Dev não deve cachear
  }
}
```

---

## 🔄 Workflow Típico

### 1. Setup Inicial

```bash
# Clone
git clone https://github.com/seu-usuario/lockdown.git
cd lockdown

# Install
pnpm install

# Type check e lint
pnpm validate
```

### 2. Desenvolvimento

```bash
# Rodar tudo em dev
pnpm dev

# Backend: http://localhost:3000
# Frontend: http://localhost:5173
# Bot: rodando em background
```

### 3. Commit e Push

```bash
# Validar antes de commit
pnpm validate

# Commit
git add .
git commit -m "feat: add ban service"

# Push
git push
```

### 4. Build para Produção

```bash
# Build tudo (menos testes)
pnpm build:prod

# Verificar outputs
ls -la packages/backend/dist
ls -la packages/frontend/dist
```

---

## 📚 Referências

- [pnpm Docs](https://pnpm.io)
- [Turborepo Docs](https://turbo.build)
- [Monorepo Guide](https://monorepo.tools)
- [SETUP.md](./SETUP.md) - Setup inicial
- [DEPLOYMENT.md](./DEPLOYMENT.md) - Deploy

---

**Happy monorepo-ing!** 📦
