# 🔧 TROUBLESHOOTING - LOCKDOWN

## Problemas Comuns e Soluções

---

## 📖 Índice

1. [Setup & Installation](#setup--installation)
2. [Node & Dependencies](#node--dependencies)
3. [Database & Redis](#database--redis)
4. [API & Backend](#api--backend)
5. [Bot](#bot)
6. [Docker & Deployment](#docker--deployment)
7. [Performance](#performance)
8. [Getting More Help](#getting-more-help)

---

## 🚀 Setup & Installation

### Problema: `pnpm: command not found`

**Solução:**

```bash
# Instalar pnpm globalmente
npm install -g pnpm@latest

# Ou verificar versão
pnpm --version

# Se usar nvm, pode ser issue de PATH
source ~/.nvm/nvm.sh
npm install -g pnpm@latest
```

---

### Problema: `node_modules` muito grande

**Solução:**

```bash
# Usar hoisted mode (padrão em nosso projeto)
# Verifica .npmrc

cat .npmrc
# Deve ter:
# node-linker=hoisted
# hoist-pattern=*
# shamefully-hoist=true

# Limpar e reinstalar
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

---

### Problema: `permission denied` ao instalar

**Solução:**

```bash
# Usar sudo (não recomendado)
sudo pnpm install

# Melhor: Corrigir permissões
sudo chown -R $USER ~/.npm
sudo chown -R $USER /usr/local/lib/node_modules

# Ou usar NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
pnpm install
```

---

## 📦 Node & Dependencies

### Problema: `EACCES: permission denied` ao executar scripts

**Solução:**

```bash
# Adicionar permissão execute
chmod +x scripts/*.sh

# Ou rodar via pnpm
pnpm run dev  # Melhor que npm run dev
```

---

### Problema: Diferentes versões de Node entre máquinas

**Solução:**

```bash
# Usar .nvmrc para padronizar versão
echo "20.10.0" > .nvmrc

# Quando entrar no diretório:
nvm use
# Output: Now using node v20.10.0

# Ou usar nodenv
nodenv local 20.10.0
```

---

### Problema: `TypeScript compilation failed`

**Solução:**

```bash
# Limpar cache TypeScript
rm -rf packages/*/dist

# Recompilar
pnpm build

# Ou verificar tsconfig.json
pnpm type-check --noEmit

# Se ainda falhar, verificar imports
grep -r "from '.'" packages/backend/src/
# Imports sem extensão .ts podem falhar
```

---

## 🗄️ Database & Redis

### Problema: `PostgreSQL connection refused`

**Solução:**

```bash
# Verificar se PostgreSQL está rodando
sudo systemctl status postgresql

# Ou se usando Docker
docker ps | grep postgres

# Iniciar PostgreSQL
sudo systemctl start postgresql

# Ou Docker Compose
docker-compose up -d postgres

# Testar conexão
psql -U lockdown_user -h localhost -d lockdown_db
```

---

### Problema: `Database URL invalid`

**Solução:**

```bash
# Verificar .env
cat .env | grep DATABASE_URL

# Formato correto:
# postgresql://user:password@host:5432/dbname

# Testar com psql
psql "postgresql://lockdown_user:password@localhost:5432/lockdown_db"

# Se tiver caracteres especiais, URL-encode
# @ = %40, # = %23, etc
# postgresql://lockdown_user:pass%40word@localhost/lockdown_db
```

---

### Problema: `Redis connection timeout`

**Solução:**

```bash
# Verificar se Redis está rodando
redis-cli ping
# Output: PONG

# Se falhar:
sudo systemctl start redis-server

# Ou Docker:
docker-compose up -d redis

# Testar conexão
redis-cli -h localhost -p 6379 ping

# Se tiver erro de port:
lsof -i :6379  # Ver qual processo usa port 6379
```

---

### Problema: `Migration failed`

**Solução:**

```bash
# Verificar status de migrações
pnpm db:migrate:status

# Ver logs
docker-compose logs postgres

# Rollback última migração (se suportado)
pnpm db:migrate:rollback

# Recriar DB (perigoso!)
dropdb -U lockdown_user lockdown_db
createdb -U lockdown_user lockdown_db
pnpm db:migrate
```

---

## 🌐 API & Backend

### Problema: `Port 3000 already in use`

**Solução:**

```bash
# Encontrar processo usando port 3000
lsof -i :3000

# Matar processo
kill -9 <PID>

# Ou usar porta diferente
PORT=3001 pnpm dev

# Ou no .env
echo "PORT=3001" >> .env
```

---

### Problema: `CORS error` ao fazer requisição

**Solução:**

```bash
# Verificar CORS middleware no backend
# packages/backend/src/middleware/cors.ts

// Deve ter:
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:5173',
  credentials: true
}));

# Verificar .env
cat .env | grep FRONTEND_URL
```

---

### Problema: `JWT token expired`

**Solução:**

```bash
# Tokens JWT têm expiração
# No backend: JWT_SECRET e expiração configurada

# Gerar novo token
curl -X POST http://localhost:3000/api/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{"botId":"seu-bot-id"}'

# Token geralmente válido por 24h
```

---

### Problema: `API returns 500 error`

**Solução:**

```bash
# Ver logs detalhados
docker-compose logs -f backend

# Ou se rodando localmente:
pnpm dev  # Mostra erro direto no terminal

# Verificar database connection
curl http://localhost:3000/api/v1/health

# Resultado esperado:
# {"status":"healthy","redis":true,"postgres":true}
```

---

## 🤖 Bot

### Problema: Bot não responde a comandos

**Solução:**

```bash
# Verificar se bot está online
# No Discord, bot deve ter status online

# Verificar logs
pm2 logs lockdown-bot
# Ou
docker-compose logs -f lockdown-bot

# Verificar token Discord
cat .env | grep DISCORD_TOKEN

# Token correto: começa com "NTk0..." ou "MjU..."

# Reiniciar bot
pm2 restart lockdown-bot
```

---

### Problema: `Bot missing permissions`

**Solução:**

```bash
# Verificar permissões do bot
# No Discord Server → Server Settings → Roles
# Role do bot deve ter:
# ✅ Send Messages
# ✅ Embed Links
# ✅ Ban Members
# ✅ Kick Members
# ✅ Manage Roles

# Atualizar invite link com permissões corretas
# https://discord.com/oauth2/authorize?client_id=YOUR_ID&scope=bot&permissions=8

# Ou permissões específicas:
# Permissions:
# - 2048 = SEND_MESSAGES
# - 536870912 = ADMINISTRATOR
```

---

### Problema: Bot não publica eventos

**Solução:**

```bash
# Verificar Redis connection
redis-cli ping

# Verificar se subscribers estão escutando
redis-cli PUBSUB CHANNELS

# Deve listar canais: ban:created, kick:executed, etc

# Publicar evento teste
redis-cli PUBLISH ban:created '{"id":"test"}'

# Verificar logs do bot
docker-compose logs -f lockdown-bot
```

---

## 🐳 Docker & Deployment

### Problema: `Docker daemon not running`

**Solução:**

```bash
# Iniciar Docker
sudo systemctl start docker

# Ou no Mac
open /Applications/Docker.app

# Verificar status
docker ps
```

---

### Problema: `docker-compose: command not found`

**Solução:**

```bash
# Instalar Docker Compose v2
sudo apt-get install docker-compose

# Ou usar versão integrada (Docker Desktop)
docker compose version

# Se tiver Docker Desktop instalado:
docker-compose up  # Deve funcionar via alias
```

---

### Problema: Containers usando muita memória

**Solução:**

```bash
# Ver uso de memória
docker stats

# Limitar memória em docker-compose.yml
services:
  backend:
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

# Limpar images não usadas
docker image prune

# Limpar volumes não usados
docker volume prune
```

---

### Problema: `SSL certificate error` após Let's Encrypt

**Solução:**

```bash
# Verificar certificado
sudo certbot certificates

# Renovar certificado
sudo certbot renew --dry-run

# Se tiver erro:
sudo certbot revoke --cert-path /etc/letsencrypt/live/domain/fullchain.pem

# Recriar
sudo certbot certonly --standalone -d yourdomain.com

# Copiar para Docker volume
sudo cp -r /etc/letsencrypt/live/yourdomain.com/* ./certs/
sudo chown -R deploy:deploy ./certs/
```

---

## ⚡ Performance

### Problema: Queries lentas

**Solução:**

```bash
# Usar EXPLAIN no PostgreSQL
psql -U lockdown_user -d lockdown_db

\d bans  # Ver estrutura da tabela

# Verificar índices
SELECT * FROM pg_indexes WHERE tablename = 'bans';

# Adicionar índice se necessário
CREATE INDEX idx_bans_guild ON moderation.bans(guild_id);

# Analisar query
EXPLAIN ANALYZE SELECT * FROM bans WHERE guild_id = '123';
```

---

### Problema: Redis memory growing

**Solução:**

```bash
# Verificar tamanho do Redis
redis-cli INFO memory

# Limpar antigas chaves
redis-cli KEYS "old:*" | xargs redis-cli DEL

# Ou resetar tudo (perigoso!)
redis-cli FLUSHDB

# Configurar max memory policy
redis-cli CONFIG SET maxmemory-policy allkeys-lru

# Persistência
# AOF (Append-Only File) - mais seguro
# RDB (Snapshot) - mais rápido

# Em redis.conf:
# appendonly yes
# save 900 1  # Save a cada 900s se houver 1 mudança
```

---

### Problema: Build muito lento

**Solução:**

```bash
# Usar Turborepo cache
pnpm turbo run build  # Usa cache automaticamente

# Limpar cache
rm -rf node_modules/.turbo

# Parallelizar builds
pnpm turbo run build --parallel

# Skip unchanged
pnpm turbo run build --filter='!shared'
```

---

## 📚 Getting More Help

### Preciso de mais ajuda?

**Comunidade:**

- 📧 Email: admin@lockdown.app
- 💬 Discord: [Convite]
- 🐛 GitHub Issues: [Issues](https://github.com/seu-usuario/lockdown/issues)

**Documentação:**

- 📖 [SETUP.md](./SETUP.md) - Setup inicial
- 🏗️ [ARCHITECTURE.md](./ARCHITECTURE.md) - Design
- 🚀 [DEPLOYMENT.md](./DEPLOYMENT.md) - Deploy
- 🔧 [BACKEND.md](./BACKEND.md) - Backend
- 💾 [DATABASE.md](./DATABASE.md) - Database

**Recursos Externos:**

- [Stack Overflow](https://stackoverflow.com/questions/tagged/express.js)
- [Node.js Documentation](https://nodejs.org/docs/)
- [Discord.js Help Server](https://discord.gg/djs)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

## 🎯 Checklist de Diagnóstico

Quando tiver problema, siga este checklist:

- [ ] Node.js versão ≥ 20?

  ```bash
  node --version
  ```

- [ ] pnpm versão ≥ 10?

  ```bash
  pnpm --version
  ```

- [ ] PostgreSQL rodando?

  ```bash
  psql -U lockdown_user -d lockdown_db -c "\q"
  ```

- [ ] Redis rodando?

  ```bash
  redis-cli ping
  ```

- [ ] .env configurado?

  ```bash
  cat .env | grep -E "DATABASE|REDIS|DISCORD"
  ```

- [ ] Dependências instaladas?

  ```bash
  ls node_modules/@lockdown/shared
  ```

- [ ] Type-check passa?

  ```bash
  pnpm type-check
  ```

- [ ] Lint passa?

  ```bash
  pnpm lint
  ```

- [ ] Backend rodando?

  ```bash
  curl http://localhost:3000/api/v1/health
  ```

- [ ] Bot online?
  ```bash
  pm2 list | grep lockdown-bot
  ```

---

## 💡 Pro Tips

### Quando tudo falha

```bash
# Nuclear option - limpar tudo
rm -rf node_modules pnpm-lock.yaml .turbo dist

# Reinstalar
pnpm install

# Validar
pnpm validate

# Se ainda falhar, criar issue com:
pnpm diagnostic > diagnostic.txt
# Enviar diagnostic.txt na issue
```

### Logs úteis

```bash
# Backend logs
docker-compose logs -f backend

# Bot logs
pm2 logs lockdown-bot

# PostgreSQL logs
docker-compose logs -f postgres

# Redis logs
docker-compose logs -f redis

# Nginx (se deployado)
docker-compose logs -f nginx
```

### Debugging

```bash
# Node debugger
node --inspect packages/backend/dist/index.js

# Acessar em Chrome: chrome://inspect

# Ou usar VS Code debugger
# .vscode/launch.json (incluído no projeto)
```

---

## 📞 Report a Bug

Se encontrou um bug:

1. **Reproduza** o problema com dados específicos
2. **Colete** logs (veja seção "Logs úteis" acima)
3. **Abra uma Issue** com:
   - Descrição clara do problema
   - Steps to reproduce
   - Output esperado vs atual
   - Logs relevantes
   - Versão Node/pnpm
4. **Ou envie PR** com a solução!

---

**Good luck!** 🍀
