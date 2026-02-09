# Variáveis de Ambiente

Guia completo de configuração de variáveis de ambiente do LOCKDOWN.

---

## Arquivo .env

Crie um arquivo `.env` na raiz do projeto baseado no `.env.example`:

```bash
cp .env.example .env
```

---

## Variáveis Obrigatórias

### Banco de Dados

```ini
# SQLite (desenvolvimento)
DATABASE_URL=file:./dev.db

# PostgreSQL (produção)
DATABASE_URL=postgresql://user:password@localhost:5432/lockdown_db
```

### Redis

```ini
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
```

### Discord

```ini
DISCORD_BOT_TOKEN=seu-token-aqui
DISCORD_CLIENT_ID=seu-client-id
DISCORD_CLIENT_SECRET=seu-client-secret
```

### JWT

```ini
JWT_SECRET=sua-chave-secreta-muito-longa-aqui
JWT_EXPIRY=24h
```

---

## Variáveis Opcionais

### Aplicação

```ini
NODE_ENV=development           # development | production | test
PORT=3000                      # Porta do backend
FRONTEND_PORT=5173             # Porta do frontend
LOG_LEVEL=debug                # trace | debug | info | warn | error
```

### CORS

```ini
FRONTEND_URL=http://localhost:5173
ALLOWED_ORIGINS=http://localhost:5173,http://localhost:3000
```

### Rate Limiting

```ini
RATE_LIMIT_WINDOW_MS=60000     # 1 minuto
RATE_LIMIT_MAX_REQUESTS=100    # Requests por janela
```

---

## Ambientes

### Desenvolvimento

```ini
NODE_ENV=development
DATABASE_URL=file:./dev.db
REDIS_HOST=localhost
LOG_LEVEL=debug
```

### Produção

```ini
NODE_ENV=production
DATABASE_URL=postgresql://user:password@db-host:5432/lockdown_db
REDIS_HOST=redis-host
LOG_LEVEL=info
```

### Testes

```ini
NODE_ENV=test
DATABASE_URL=file:./test.db
LOG_LEVEL=error
```

---

## Validação

O sistema valida as variáveis na inicialização:

```typescript
// packages/backend/src/config/environment.ts
const requiredVars = [
  'DATABASE_URL',
  'DISCORD_BOT_TOKEN',
  'JWT_SECRET'
];
```

Se alguma variável obrigatória estiver faltando, o sistema não inicializa.

---

## Segurança

### Nunca Faça
- Commitar `.env` no Git
- Usar secrets em código
- Expor variáveis no frontend

### Sempre Faça
- Usar `.env.example` como template
- Secrets diferentes por ambiente
- Rotacionar secrets periodicamente

---

## Docker

Para Docker, use docker-compose.yml com environment:

```yaml
services:
  backend:
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_HOST=redis
```

---

**Configure com segurança!**
