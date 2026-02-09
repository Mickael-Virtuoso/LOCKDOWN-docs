# Frontend - Deployment

## Build

```bash
pnpm build

# Output:
# ✓ 123 modules transformed
# dist/index.html
# dist/assets/main.abc123.js
# dist/assets/style.def456.css
```

---

## Preview Local

```bash
pnpm preview

# http://localhost:4173
```

---

## Deploy Manual

### Copiar para servidor

```bash
# Via SCP
scp -r dist/* user@server:/var/www/lockdown/frontend

# Via rsync
rsync -avz dist/ user@server:/var/www/lockdown/frontend/
```

---

## Docker

### Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build

# Production stage
FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### nginx.conf

```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
```

### docker-compose.yml

```yaml
services:
  frontend:
    build:
      context: ./packages/frontend
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - backend
```

---

## Vercel

### vercel.json

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ],
  "headers": [
    {
      "source": "/assets/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
      ]
    }
  ]
}
```

### Deploy

```bash
# Instalar CLI
npm i -g vercel

# Deploy
vercel --prod
```

---

## Netlify

### netlify.toml

```toml
[build]
  command = "pnpm build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

---

## Variáveis de Ambiente

### Produção

```env
VITE_API_BASE_URL=https://api.lockdown.app/v1
VITE_BOT_INVITE_URL=https://discord.com/oauth2/authorize?client_id=...
VITE_SUPPORT_SERVER=https://discord.gg/lockdown
```

---

## Checklist de Deploy

- [ ] Build sem erros
- [ ] Variáveis de ambiente configuradas
- [ ] API URL apontando para produção
- [ ] Assets otimizados
- [ ] HTTPS configurado
- [ ] Redirects para SPA configurados

---

## Links Relacionados

- [Setup](./SETUP.md)
- [Deployment Geral](../01-architecture/DEPLOYMENT.md)
