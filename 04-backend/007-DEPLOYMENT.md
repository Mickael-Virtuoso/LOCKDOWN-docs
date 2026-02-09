# Backend - Deployment

## Build Local

```bash
pnpm build:backend

# Gera:
# packages/backend/dist/server.js
# packages/backend/dist/...
```

---

## Start Production

```bash
# Compilado
pnpm start:backend

# Com PM2 (VPS)
pnpm start:backend:pm2
```

---

## PM2 Configuration

`ecosystem.config.cjs`

```javascript
module.exports = {
  apps: [
    {
      name: 'lockdown-backend',
      script: 'packages/backend/dist/server.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 3000,
      },
    },
  ],
};
```

### Comandos PM2

```bash
# Iniciar
pm2 start ecosystem.config.cjs

# Listar processos
pm2 list

# Logs
pm2 logs lockdown-backend

# Reiniciar
pm2 restart lockdown-backend

# Parar
pm2 stop lockdown-backend

# Monitoramento
pm2 monit
```

---

## Docker

### Dockerfile

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY packages/backend/package.json ./packages/backend/
COPY packages/shared/package.json ./packages/shared/

RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile

COPY packages/backend ./packages/backend
COPY packages/shared ./packages/shared

RUN pnpm build:backend

FROM node:18-alpine AS runner

WORKDIR /app
COPY --from=builder /app/packages/backend/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

### Docker Compose

```yaml
services:
  backend:
    build:
      context: .
      dockerfile: packages/backend/dockerfile.backend
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - REDIS_URL=redis://redis:6379
      - DATABASE_URL=/data/lockdown.db
    volumes:
      - ./data:/data
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

---

## Health Check

```bash
curl http://localhost:3000/api/v1/health
```

### Resposta Esperada

```json
{
  "status": "healthy",
  "uptime": 12345.678,
  "timestamp": "2025-02-05T10:00:00Z",
  "services": {
    "database": "connected",
    "redis": "connected"
  }
}
```

---

## Variáveis de Ambiente (Produção)

```env
# Server
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=/data/lockdown.db

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=<strong-secret-key>
JWT_EXPIRES_IN=7d

# Logging
LOG_LEVEL=info
```

---

## Checklist de Deploy

- [ ] Build sem erros
- [ ] Variáveis de ambiente configuradas
- [ ] Database migrado
- [ ] Redis conectado
- [ ] Health check passando
- [ ] Logs configurados
- [ ] Monitoramento ativo

---

## Links Relacionados

- [Setup](./SETUP.md)
- [Deployment Geral](../01-architecture/DEPLOYMENT.md)
- [Docker Compose](../../docker-compose.yml)
