# Sistema de Logs

Sistema de logging do LOCKDOWN usando Pino.

---

## Configuração

```typescript
// config/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true,
      translateTime: 'SYS:standard',
    },
  },
});
```

---

## Níveis de Log

| Nível | Valor | Uso |
|-------|-------|-----|
| `trace` | 10 | Debug detalhado |
| `debug` | 20 | Desenvolvimento |
| `info` | 30 | Operações normais |
| `warn` | 40 | Situações anômalas |
| `error` | 50 | Erros |
| `fatal` | 60 | Erros críticos |

---

## Padrão de Log

```typescript
// ✅ Com contexto
logger.info(`[${this.constructor.name}] "✅ operation completed"`);

// ✅ Com dados estruturados
logger.info({
  action: 'ban_created',
  guildId: '123',
  userId: '456',
}, 'Ban created successfully');
```

---

## Visualização

### PM2 Logs

```bash
# Todos os logs
pm2 logs

# Logs específicos
pm2 logs lockdown-backend

# Últimas N linhas
pm2 logs --lines 100
```

### Filtrar Logs

```bash
# Apenas erros
pm2 logs | grep ERROR

# Por timestamp
pm2 logs --format
```

---

## Rotação de Logs

### PM2 Log Rotate

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

---

## Logs Estruturados

### Request Logging

```typescript
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
  }, 'Incoming request');

  next();
});
```

### Error Logging

```typescript
app.use((err, req, res, next) => {
  logger.error({
    error: err.message,
    stack: err.stack,
    method: req.method,
    url: req.url,
  }, 'Request error');

  res.status(500).json({ error: 'Internal Server Error' });
});
```

---

## Boas Práticas

1. **Não logar dados sensíveis** (senhas, tokens)
2. **Usar níveis apropriados** (não info para debug)
3. **Estruturar logs** para análise posterior
4. **Rotacionar logs** para evitar disco cheio

---

## Links Relacionados

- [Monitoramento](./MONITORING.md)
- [Logging Core](../03-core/LOGGING.md)
