# 🚀 SETUP - LOCKDOWN

## Guia Completo de Instalação e Configuração

---

## 📋 Pré-requisitos

### Sistema Operacional

- **macOS**, **Linux** ou **Windows** (WSL2 recomendado)
- **Node.js**: `>=18.17.0` (recomendado 20+)
- **pnpm**: `>=8.0.0` (gerenciador de pacotes)
- **Git**: `>=2.30.0`

### Verificar Versões

```bash
node --version      # v20.x.x ou superior
npm --version       # 10.x.x ou superior
git --version       # 2.40+
```

### Instalar pnpm

```bash
npm install -g pnpm@latest

# Verificar
pnpm --version      # 10.x.x ou superior
```

---

## 🛠️ Setup Inicial (Primeira Vez)

### 1. Clonar Repositório

```bash
git clone https://github.com/seu-usuario/lockdown.git
cd lockdown
```

### 2. Instalar Dependências

```bash
# Instala todas as dependências do monorepo
pnpm install

# Aguarde ~2-3 minutos (primeira instalação é lenta)
```

### 3. Verificar Instalação

```bash
# Ver workspaces reconhecidos
pnpm ls --depth=-1

# Esperado:
# lockdown-monorepo@0.0.1
# ├── @lockdown/backend
# ├── @lockdown/bot
# ├── @lockdown/frontend
# ├── @lockdown/shared
# └── @lockdown/tests
```

### 4. Validar Stack

```bash
# Type-check completo
pnpm type-check

# Lint completo
pnpm lint

# Format check
pnpm format:check

# Esperado:
# ✅ Tudo passou!
```

---

## ⚙️ Variáveis de Ambiente

### Backend (.env)

Criar arquivo **`packages/backend/.env`**:

```ini
# =====================================================
# SERVER CONFIGURATION
# =====================================================
NODE_ENV=development
PORT=3000
HOST=localhost

# =====================================================
# DATABASE
# =====================================================
DATABASE_URL=file:./dev.db
DATABASE_PROVIDER=better-sqlite3

# Para PostgreSQL (futuro):
# DATABASE_URL=postgresql://user:password@localhost:5432/lockdown
# DATABASE_PROVIDER=postgres

# =====================================================
# REDIS
# =====================================================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_PASSWORD=

# =====================================================
# JWT & SECURITY
# =====================================================
JWT_SECRET=your-super-secret-key-change-me-in-production
JWT_EXPIRY=24h

# =====================================================
# LOGGING
# =====================================================
LOG_LEVEL=info
LOG_FORMAT=pretty

# =====================================================
# CORS
# =====================================================
CORS_ORIGIN=*
CORS_CREDENTIALS=true
```

### Bot (.env)

Criar arquivo **`packages/lockdown-bot/.env`**:

```ini
# =====================================================
# DISCORD BOT
# =====================================================
DISCORD_TOKEN=your-discord-bot-token-here
DISCORD_CLIENT_ID=your-client-id-here

# =====================================================
# API BACKEND
# =====================================================
API_BASE_URL=http://localhost:3000/api/v1
API_TIMEOUT=5000

# =====================================================
# LOGGING
# =====================================================
LOG_LEVEL=info
```

### Frontend (.env)

Criar arquivo **`packages/frontend/.env`**:

```ini
# =====================================================
# API
# =====================================================
VITE_API_BASE_URL=http://localhost:3000/api/v1

# =====================================================
# ENVIRONMENT
# =====================================================
VITE_ENV=development
```

---

## 🚀 Desenvolvimento Local

### Rodar Tudo em Paralelo

```bash
# Dev mode (todos os packages em paralelo)
pnpm dev

# Esperado:
# @lockdown/backend running on http://localhost:3000
# @lockdown/bot connected to Discord
# @lockdown/frontend running on http://5173
```

### Rodar Específico

```bash
# Apenas backend
pnpm dev:backend

# Apenas bot
pnpm dev:bot

# Apenas frontend
pnpm dev:frontend
```

### Parar Serviços

```bash
# Ctrl + C em cada terminal, ou:
pkill -f "tsx\|vite\|node"
```

---

## 📦 Build para Produção

### Build Completo

```bash
# Build todos os packages
pnpm build:prod

# Gera:
# packages/backend/dist/
# packages/lockdown-bot/dist/
# packages/frontend/dist/
# packages/shared/dist/
```

### Build Isolado

```bash
pnpm build:backend
pnpm build:bot
pnpm build:frontend
```

### Verificar Build

```bash
# Ver artifacts gerados
ls -la packages/backend/dist/
ls -la packages/lockdown-bot/dist/
ls -la packages/frontend/dist/
```

---

## 🗄️ Banco de Dados

### Setup SQLite (Development)

```bash
# Migrations automáticas (via Drizzle)
pnpm db:migrate

# Seed com dados de teste (opcional)
pnpm db:seed

# Abrir Drizzle Studio (GUI)
pnpm db:studio
# Abre em http://local.drizzle.studio
```

### Reset Database

```bash
# ⚠️ CUIDADO: Deleta tudo!
pnpm db:reset

# Depois:
pnpm db:migrate
pnpm db:seed
```

### Visualizar Dados

```bash
# Drizzle Studio
pnpm db:studio

# Ou SQLite CLI
sqlite3 packages/backend/dev.db
# > .tables
# > SELECT * FROM users;
# > .quit
```

---

## 🧪 Testes

### Rodar Todos os Testes

```bash
pnpm test

# Esperado:
# PASS  tests/unit/...
# PASS  tests/integration/...
```

### Testes Específicos

```bash
# Testes unitários
pnpm test:unit

# Testes integração
pnpm test:integration

# Testes E2E
pnpm test:e2e

# Coverage
pnpm test:coverage
```

### Watch Mode (Desenvolvimento)

```bash
# Roda testes ao modificar arquivos
pnpm test:watch
```

---

## ✅ Validação Antes de Commit

### Checklist Completo

```bash
# Executar tudo:
pnpm validate

# Se tiver problemas:
pnpm validate:fix

# Ou manual:
pnpm type-check    # Type-check
pnpm lint           # ESLint
pnpm format:check   # Prettier
pnpm test           # Testes
```

### Antes de Push

```bash
# Type-check + lint + format
pnpm before-push

# Se passar, pode fazer git push
git push
```

---

## 🐛 Troubleshooting

### Problema: "Cannot find module"

```bash
# Solução:
pnpm install
pnpm clean
pnpm install
```

### Problema: Type-check falha

```bash
# Verificar tipos
pnpm type-check

# Ver detalhes
pnpm type-check 2>&1 | head -50
```

### Problema: ESLint/Prettier conflita

```bash
# Arrumar formatação
pnpm format

# Depois lint:fix
pnpm lint:fix

# Testar novamente
pnpm lint
```

### Problema: Port 3000 já em uso

```bash
# Mudar PORT em .env backend
PORT=3001

# Ou matar processo:
lsof -i :3000
kill -9 <PID>
```

### Problema: node_modules corrompido

```bash
# Reset completo
pnpm clean:hard

# Reinstalar
pnpm install
```

---

## 🔄 Workflow Recomendado

### Dia a Dia

```bash
# 1. Morning - pull latest
git pull origin main

# 2. Install new deps (se houver)
pnpm install

# 3. Type-check
pnpm type-check

# 4. Development
pnpm dev

# 5. Before commit
pnpm before-commit

# 6. Commit & push
git add .
git commit -m "feat: something"
git push

# 7. Before final push
pnpm before-push
```

---

## 📊 Verificar Saúde do Projeto

```bash
# Health check completo
pnpm validate

# Ver estrutura
tree -L 2 -I 'node_modules|dist'

# Ver tamanho
du -sh .
du -sh node_modules/
du -sh packages/*/dist

# Ver packages
pnpm ls --depth=-1
```

---

## 🎓 Próximos Passos

- Leia **ARCHITECTURE.md** para entender design
- Leia **BACKEND.md** para iniciar desenvolvimento
- Leia **API.md** para endpoints disponíveis
- Leia **DEPLOYMENT.md** para deploy na VPS

---

## 📞 Suporte

Se tiver problemas:

1. Verifica **TROUBLESHOOTING.md**
2. Roda `pnpm validate:fix`
3. Limpa tudo com `pnpm clean:hard`
4. Instala novamente: `pnpm install`

---

**Happy coding!** 🚀
