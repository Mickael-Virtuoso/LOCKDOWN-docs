# Monitoramento

Sistema de monitoramento do LOCKDOWN.

---

## Health Checks

### Endpoint

```bash
GET /api/v1/health

HTTP/1.1 200 OK

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

### Implementação

```typescript
app.get('/api/v1/health', async (req, res) => {
  const dbStatus = await checkDatabase();
  const redisStatus = await checkRedis();

  const healthy = dbStatus === 'connected' && redisStatus === 'connected';

  res.status(healthy ? 200 : 503).json({
    status: healthy ? 'healthy' : 'unhealthy',
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
    services: {
      database: dbStatus,
      redis: redisStatus,
    },
  });
});
```

---

## PM2 Monitoring

### Comandos

```bash
# Status dos processos
pm2 status

# Monitoramento em tempo real
pm2 monit

# Métricas
pm2 show lockdown-backend
```

### Métricas Disponíveis

| Métrica | Descrição |
|---------|-----------|
| CPU | Uso de CPU |
| Memory | Uso de memória |
| Uptime | Tempo online |
| Restarts | Número de reinícios |

---

## Alertas

### Configuração PM2

```javascript
// ecosystem.config.cjs
module.exports = {
  apps: [{
    name: 'lockdown-backend',
    script: 'dist/server.js',
    max_memory_restart: '500M',
    restart_delay: 5000,
    max_restarts: 10,
  }],
};
```

### Script de Alerta

```bash
#!/bin/bash
# scripts/alert.sh

STATUS=$(curl -s http://localhost:3000/api/v1/health | jq -r '.status')

if [ "$STATUS" != "healthy" ]; then
  echo "ALERT: Service unhealthy!"
  # Enviar notificação
fi
```

---

## Redis Monitoring

```typescript
// Verificar conexão Redis
async function checkRedis(): Promise<string> {
  try {
    await redis.ping();
    return 'connected';
  } catch {
    return 'disconnected';
  }
}

// Métricas
const info = await redis.info();
```

---

## Database Monitoring

```typescript
// Verificar conexão DB
async function checkDatabase(): Promise<string> {
  try {
    await db.execute(sql`SELECT 1`);
    return 'connected';
  } catch {
    return 'disconnected';
  }
}
```

---

## Links Relacionados

- [Logs](./LOGS.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
