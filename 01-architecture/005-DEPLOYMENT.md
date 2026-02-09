# 🚀 DEPLOYMENT - LOCKDOWN

## Deploy na VPS com Docker e PM2

---

## 📖 Índice

1. [Visão Geral](#visão-geral)
2. [VPS Setup](#vps-setup)
3. [Docker Setup](#docker-setup)
4. [PM2 Configuration](#pm2-configuration)
5. [PostgreSQL Setup](#postgresql-setup)
6. [Redis Setup](#redis-setup)
7. [Nginx Reverse Proxy](#nginx-reverse-proxy)
8. [SSL/HTTPS](#sslhttps)
9. [Monitoring](#monitoring)
10. [Backup e Restore](#backup-e-restore)

---

## 📋 Visão Geral

### Arquitetura Alvo (VPS)

```
┌─────────────────────────────────────┐
│          LOCKDOWN VPS               │
│  (2 vCPU, 8GB RAM, 100GB SSD)       │
├─────────────────────────────────────┤
│                                     │
│  ┌─────────────────────────────┐   │
│  │  Nginx (Reverse Proxy)      │   │
│  │  - HTTPS/SSL                │   │
│  │  - Load Balance             │   │
│  └────────────┬────────────────┘   │
│               │                     │
│  ┌────────────▼──────────────────┐ │
│  │  Docker Containers           │ │
│  │  ├─ Backend API (3000)       │ │
│  │  ├─ Discord Bot (process)    │ │
│  │  └─ Frontend (5173)          │ │
│  └────────────┬──────────────────┘ │
│               │                     │
│  ┌────────────▼──────────────────┐ │
│  │  Services                    │ │
│  │  ├─ PostgreSQL (5432)        │ │
│  │  ├─ Redis (6379)             │ │
│  │  └─ PM2 (Process Manager)    │ │
│  └──────────────────────────────┘ │
│                                     │
└─────────────────────────────────────┘
```

### Stack de Deploy

- **OS:** Ubuntu 22.04 LTS
- **Container:** Docker + Docker Compose
- **Process Manager:** PM2
- **Database:** PostgreSQL 15
- **Cache:** Redis 7
- **Web Server:** Nginx
- **SSL:** Let's Encrypt + Certbot
- **Monitoring:** PM2 Plus (opcional)

---

## 🔧 VPS Setup

### 1. Conectar na VPS

```bash
# SSH
ssh -i your_key.pem ubuntu@your_vps_ip

# Ou com senha
ssh ubuntu@your_vps_ip
```

### 2. Atualizar Sistema

```bash
# Atualizar packages
sudo apt update && sudo apt upgrade -y

# Instalar ferramentas essenciais
sudo apt install -y \
  curl \
  wget \
  git \
  build-essential \
  libssl-dev \
  apt-transport-https \
  ca-certificates \
  software-properties-common
```

### 3. Instalar Node.js

```bash
# Adicionar Node.js repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Instalar Node.js e npm
sudo apt install -y nodejs

# Instalar pnpm globalmente
sudo npm install -g pnpm@latest

# Verificar versões
node --version  # v20.x.x
npm --version   # 10.x.x
pnpm --version  # 10.x.x
```

### 4. Criar Usuário de Deploy

```bash
# Criar usuário
sudo useradd -m -s /bin/bash deploy

# Adicionar ao docker group (depois)
sudo usermod -aG docker deploy

# Switch para deploy user
sudo -u deploy -i
```

### 5. Clone do Repositório

```bash
# Como deploy user
cd /home/deploy

# Clone
git clone https://github.com/seu-usuario/lockdown.git
cd lockdown

# Instalar dependências
pnpm install
```

---

## 🐳 Docker Setup

### docker-compose.yml

**Localizar em raiz do projeto:**

```yaml
version: '3.9'

services:
  # =====================================================
  # BACKEND API
  # =====================================================
  backend:
    build:
      context: ./packages/backend
      dockerfile: dockerfile.backend
    container_name: lockdown-backend
    ports:
      - '3000:3000'
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgresql://lockdown_user:${DB_PASSWORD}@postgres:5432/lockdown_db
      REDIS_HOST: redis
      REDIS_PORT: 6379
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - lockdown_network
    volumes:
      - ./logs/backend:/app/logs

  # =====================================================
  # BOT (via PM2, não Docker)
  # =====================================================
  # Bot será gerenciado por PM2 diretamente

  # =====================================================
  # FRONTEND
  # =====================================================
  frontend:
    build:
      context: ./packages/frontend
      dockerfile: dockerfile.frontend
    container_name: lockdown-frontend
    ports:
      - '5173:5173'
    environment:
      VITE_API_BASE_URL: https://api.yourdomain.com/api/v1
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - lockdown_network
    volumes:
      - ./logs/frontend:/app/logs

  # =====================================================
  # POSTGRESQL
  # =====================================================
  postgres:
    image: postgres:15-alpine
    container_name: lockdown-postgres
    environment:
      POSTGRES_USER: lockdown_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: lockdown_db
    ports:
      - '5432:5432'
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U lockdown_user']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - lockdown_network
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # =====================================================
  # REDIS
  # =====================================================
  redis:
    image: redis:7-alpine
    container_name: lockdown-redis
    ports:
      - '6379:6379'
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - lockdown_network
    volumes:
      - redis_data:/data

  # =====================================================
  # NGINX (Reverse Proxy)
  # =====================================================
  nginx:
    image: nginx:latest
    container_name: lockdown-nginx
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
      - ./logs/nginx:/var/log/nginx
    depends_on:
      - backend
      - frontend
    restart: unless-stopped
    networks:
      - lockdown_network

networks:
  lockdown_network:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
```

### Dockerfiles

**packages/backend/dockerfile.backend:**

```dockerfile
# =====================================================
# BUILD STAGE
# =====================================================
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package.json pnpm-lock.yaml ./
COPY pnpm-workspace.yaml ./
COPY packages/backend ./packages/backend
COPY packages/shared ./packages/shared
COPY tsconfig.json ./

# Install dependencies
RUN npm install -g pnpm@latest && pnpm install

# Build
RUN pnpm build:backend

# =====================================================
# RUNTIME STAGE
# =====================================================
FROM node:20-alpine

WORKDIR /app

# Install only production dependencies
COPY package.json pnpm-lock.yaml ./
COPY pnpm-workspace.yaml ./
RUN npm install -g pnpm@latest && pnpm install --prod

# Copy built app
COPY --from=builder /app/packages/backend/dist ./dist

# Create logs directory
RUN mkdir -p logs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/api/v1/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

# Expose port
EXPOSE 3000

# Start app
CMD ["node", "dist/server.js"]
```

**packages/frontend/dockerfile.frontend:**

```dockerfile
# =====================================================
# BUILD STAGE
# =====================================================
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
COPY pnpm-workspace.yaml ./
COPY packages/frontend ./packages/frontend
COPY packages/shared ./packages/shared
COPY tsconfig.json ./

RUN npm install -g pnpm@latest && pnpm install

RUN pnpm build:frontend

# =====================================================
# RUNTIME STAGE (Nginx)
# =====================================================
FROM nginx:alpine

COPY --from=builder /app/packages/frontend/dist /usr/share/nginx/html

EXPOSE 5173

CMD ["nginx", "-g", "daemon off;"]
```

### Criar .env.production

**Na VPS, em `/home/deploy/lockdown/`:**

```bash
# Create .env file
cat > .env.production << 'EOF'
# =====================================================
# DATABASE
# =====================================================
DATABASE_URL=postgresql://lockdown_user:STRONG_PASSWORD_HERE@postgres:5432/lockdown_db
DB_PASSWORD=STRONG_PASSWORD_HERE

# =====================================================
# SECURITY
# =====================================================
JWT_SECRET=your_jwt_secret_key_here
API_JWT_SECRET=your_api_jwt_secret_here

# =====================================================
# DISCORD
# =====================================================
DISCORD_TOKEN=your_discord_bot_token
DISCORD_CLIENT_ID=your_client_id

# =====================================================
# REDIS
# =====================================================
REDIS_HOST=redis
REDIS_PORT=6379

# =====================================================
# NODE
# =====================================================
NODE_ENV=production
EOF
```

### Deploy com Docker Compose

```bash
# Como deploy user em /home/deploy/lockdown

# Build images
docker-compose build

# Start services
docker-compose up -d

# Check status
docker-compose ps

# Logs
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f postgres
```

---

## 🔄 PM2 Configuration

### ecosystem.config.cjs (Raiz do projeto)

```javascript
/**
 * - @file ecosystem.config.cjs
 *
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Mickael Virtuoso
 * @since 2025-02-05
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * PM2 configuration for production deployment
 *
 */

module.exports = {
  apps: [
    // =====================================================
    // DISCORD BOT
    // =====================================================
    {
      name: 'lockdown-bot',
      script: 'packages/lockdown-bot/dist/index.js',
      instances: 1,
      exec_mode: 'fork',
      env: {
        NODE_ENV: 'production',
        DISCORD_TOKEN: process.env.DISCORD_TOKEN,
        DISCORD_CLIENT_ID: process.env.DISCORD_CLIENT_ID,
        API_BASE_URL: 'http://localhost:3000/api/v1',
        REDIS_HOST: 'localhost',
        REDIS_PORT: 6379,
        LOG_LEVEL: 'info',
      },
      autorestart: true,
      max_memory_restart: '512M',
      error_file: './logs/pm2-error.log',
      out_file: './logs/pm2-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,
      watch: false,
      ignore_watch: ['node_modules', 'dist'],
      listen_timeout: 5000,
    },

    // =====================================================
    // BACKUP SERVICE (Optional)
    // =====================================================
    {
      name: 'lockdown-backup',
      script: './scripts/backup.js',
      cron: '0 2 * * *', // 2 AM daily
      instances: 1,
      exec_mode: 'fork',
      env: {
        NODE_ENV: 'production',
        DATABASE_URL: process.env.DATABASE_URL,
      },
      autorestart: false,
      error_file: './logs/backup-error.log',
      out_file: './logs/backup-out.log',
    },
  ],

  deploy: {
    production: {
      user: 'deploy',
      host: 'your_vps_ip',
      ref: 'origin/main',
      repo: 'git@github.com:seu-usuario/lockdown.git',
      path: '/home/deploy/lockdown',
      'post-deploy':
        'pnpm install && pnpm build && pm2 startOrRestart ecosystem.config.cjs --env production',
    },
  },
};
```

### Instalar PM2

```bash
sudo npm install -g pm2

# Start bot
pm2 start ecosystem.config.cjs --env production

# Monitor
pm2 monit

# Logs
pm2 logs lockdown-bot

# Restart
pm2 restart lockdown-bot

# Stop
pm2 stop lockdown-bot

# Startup (auto-restart em reboot)
pm2 startup
pm2 save
```

---

## 🗄️ PostgreSQL Setup

### Backup Automático

**`scripts/backup.js`:**

```javascript
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

const timestamp = new Date().toISOString().split('T')[0];
const backupDir = path.join(__dirname, '../backups');
const backupFile = path.join(backupDir, `lockdown_${timestamp}.sql.gz`);

if (!fs.existsSync(backupDir)) {
  fs.mkdirSync(backupDir, { recursive: true });
}

const cmd = `pg_dump -U lockdown_user -h localhost lockdown_db | gzip > ${backupFile}`;

exec(cmd, (error) => {
  if (error) {
    console.error(`Backup failed: ${error.message}`);
    process.exit(1);
  }

  console.log(`Backup created: ${backupFile}`);

  // Keep only last 30 backups
  const files = fs
    .readdirSync(backupDir)
    .map((f) => ({
      name: f,
      path: path.join(backupDir, f),
      time: fs.statSync(path.join(backupDir, f)).mtime.getTime(),
    }))
    .sort((a, b) => b.time - a.time);

  if (files.length > 30) {
    files.slice(30).forEach((f) => {
      fs.unlinkSync(f.path);
      console.log(`Deleted old backup: ${f.name}`);
    });
  }

  process.exit(0);
});
```

### Restaurar Backup

```bash
# List backups
ls -lh backups/

# Restore
gunzip < backups/lockdown_2025-02-05.sql.gz | psql -U lockdown_user -h localhost lockdown_db

# Verify
psql -U lockdown_user -h localhost lockdown_db -c "\dt"
```

---

## 🔴 Redis Setup

### Persistência

**No `docker-compose.yml`, adicionar ao Redis:**

```yaml
redis:
  volumes:
    - redis_data:/data
  command: redis-server --appendonly yes # AOF persistence
```

### Monitorar

```bash
# Conectar
redis-cli -h localhost -p 6379

# Comandos
ping              # Teste
info              # Status
dbsize            # Número de chaves
flushall          # Limpar DB (cuidado!)
SHUTDOWN          # Parar Redis
```

---

## 🌐 Nginx Reverse Proxy

### nginx.conf

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;

  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;
  client_max_body_size 20M;

  gzip on;
  gzip_disable "msie6";

  # =====================================================
  # BACKEND API
  # =====================================================
  upstream backend {
    server backend:3000;
  }

  # =====================================================
  # FRONTEND
  # =====================================================
  upstream frontend {
    server frontend:5173;
  }

  # =====================================================
  # HTTP REDIRECT TO HTTPS
  # =====================================================
  server {
    listen 80;
    server_name _;
    return 301 https://$host$request_uri;
  }

  # =====================================================
  # HTTPS SERVER
  # =====================================================
  server {
    listen 443 ssl http2;
    server_name yourdomain.com;

    # SSL certificates
    ssl_certificate /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;

    # =====================================================
    # API ROUTES
    # =====================================================
    location /api/ {
      proxy_pass http://backend;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_read_timeout 90;
    }

    # =====================================================
    # FRONTEND
    # =====================================================
    location / {
      proxy_pass http://frontend;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
    }

    # =====================================================
    # HEALTH CHECK
    # =====================================================
    location /health {
      access_log off;
      return 200 "healthy\n";
      add_header Content-Type text/plain;
    }
  }
}
```

---

## 🔒 SSL/HTTPS

### Let's Encrypt com Certbot

```bash
# Instalar Certbot
sudo apt install -y certbot python3-certbot-nginx

# Gerar certificado
sudo certbot certonly --standalone \
  -d yourdomain.com \
  -d api.yourdomain.com \
  --email admin@yourdomain.com \
  --agree-tos \
  --non-interactive

# Copiar para Docker volume
sudo cp -r /etc/letsencrypt/live/yourdomain.com/* ./certs/
sudo chown -R deploy:deploy ./certs/

# Auto-renew
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# Teste
sudo certbot renew --dry-run
```

---

## 📊 Monitoring

### PM2 Plus (Cloud Monitoring)

```bash
# Connect to PM2 Plus
pm2 link your_secret_key your_public_key

# Monitor
pm2 monit
pm2 web  # Dashboard http://localhost:9615
```

### Prometheus + Grafana (Avançado)

```bash
# Instalar node-exporter
sudo apt install -y node-exporter

# Habilitar
sudo systemctl enable node-exporter
sudo systemctl start node-exporter

# Verificar
curl http://localhost:9100/metrics
```

---

## 💾 Backup e Restore

### Full Backup Script

**`scripts/full-backup.sh`:**

```bash
#!/bin/bash

BACKUP_DIR="/home/deploy/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/lockdown_$TIMESTAMP.tar.gz"

mkdir -p $BACKUP_DIR

# Database
pg_dump -U lockdown_user -h localhost lockdown_db | gzip > "$BACKUP_DIR/db_$TIMESTAMP.sql.gz"

# Code
tar -czf "$BACKUP_FILE" \
  /home/deploy/lockdown/packages \
  /home/deploy/lockdown/.env.production \
  /home/deploy/lockdown/ecosystem.config.cjs

echo "Backup created: $BACKUP_FILE"

# Keep only last 10 backups
find $BACKUP_DIR -name "lockdown_*.tar.gz" -type f -printf '%T@ %p\n' | \
  sort -rn | tail -n +11 | cut -d' ' -f2- | xargs -r rm

echo "Cleanup complete. Keeping only 10 latest backups."
```

### Restore

```bash
# Restore database
gunzip < backups/db_20250205_100000.sql.gz | psql -U lockdown_user -h localhost lockdown_db

# Restore code
tar -xzf backups/lockdown_20250205_100000.tar.gz -C /

# Restart services
docker-compose restart backend
pm2 restart lockdown-bot
```

---

## 📚 Referências

- [Docker Docs](https://docs.docker.com)
- [PM2 Docs](https://pm2.keymetrics.io/docs)
- [Nginx Docs](https://nginx.org/en/docs/)
- [Let's Encrypt](https://letsencrypt.org)
- [SETUP.md](./SETUP.md) - Setup local

---

**Happy deploying!** 🚀
