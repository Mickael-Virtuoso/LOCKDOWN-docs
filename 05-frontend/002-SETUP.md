# Frontend - Setup

## Pré-requisitos

- Node.js 18+
- pnpm

---

## Instalação

```bash
cd packages/frontend
pnpm install
```

---

## Variáveis de Ambiente

Crie um arquivo `.env` em `packages/frontend/`:

```env
VITE_API_BASE_URL=http://localhost:3000/api/v1
VITE_BOT_INVITE_URL=https://discord.com/oauth2/authorize?...
VITE_SUPPORT_SERVER=https://discord.gg/...
```

---

## Scripts Disponíveis

```bash
# Desenvolvimento
pnpm dev              # Inicia em http://localhost:5173

# Build
pnpm build            # Gera dist/

# Preview
pnpm preview          # Preview do build em http://localhost:4173

# Lint
pnpm lint             # Verifica código
pnpm lint:fix         # Corrige automaticamente

# Type check
pnpm typecheck        # Verifica tipos
```

---

## Rodar em Desenvolvimento

```bash
pnpm dev

# Esperado:
# VITE v5.0.0  ready in XXX ms
# ➜  Local:   http://localhost:5173/
```

---

## Troubleshooting

### Porta já em uso

```bash
# Especificar outra porta
pnpm dev -- --port 3001
```

### API não conecta

Verifique se:
1. Backend está rodando
2. `VITE_API_BASE_URL` está correto
3. CORS está configurado no backend

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Estrutura](./STRUCTURE.md)
- [Variáveis de Ambiente](../00-introduction/ENVIRONMENT.md)
