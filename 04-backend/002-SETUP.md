# Backend - Setup

## Pré-requisitos

- Node.js 18+
- pnpm
- Redis (para cache e eventos)

---

## Abrir Backend em Dev Mode

```bash
# Terminal 1: Iniciar backend
cd packages/backend
pnpm dev

# Esperado:
# [backend] 🚀 Server running on http://localhost:3000
```

---

## Variáveis de Ambiente

Crie um arquivo `.env` em `packages/backend/`:

```env
# Server
PORT=3000
NODE_ENV=development

# Database
DATABASE_URL=./data/lockdown.db

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=7d

# Logging
LOG_LEVEL=debug
```

---

## Verificar Saúde

```bash
# Terminal 2: Health check
curl http://localhost:3000/api/v1/health

# Resposta esperada:
# {
#   "status": "healthy",
#   "uptime": 12.345,
#   "timestamp": "2025-02-05T10:00:00Z"
# }
```

---

## Scripts Disponíveis

```bash
# Desenvolvimento
pnpm dev              # Inicia com hot-reload

# Build
pnpm build            # Compila TypeScript

# Produção
pnpm start            # Inicia compilado
pnpm start:pm2        # Inicia com PM2

# Testes
pnpm test             # Todos os testes
pnpm test:unit        # Testes unitários
pnpm test:integration # Testes de integração

# Lint
pnpm lint             # Verifica código
pnpm lint:fix         # Corrige automaticamente
```

---

## Troubleshooting

### Porta já em uso

```bash
# Encontrar processo na porta 3000
lsof -i :3000

# Matar processo
kill -9 <PID>
```

### Redis não conecta

```bash
# Verificar se Redis está rodando
redis-cli ping

# Iniciar Redis
sudo systemctl start redis
```

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Estrutura](./STRUCTURE.md)
- [Variáveis de Ambiente](../00-introduction/ENVIRONMENT.md)
