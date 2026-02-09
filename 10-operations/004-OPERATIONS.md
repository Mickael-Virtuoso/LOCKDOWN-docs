# Operations - LOCKDOWN

Documentação de operações e monitoramento da plataforma LOCKDOWN.

---

## Índice

| Documento | Descrição |
|-----------|-----------|
| [Monitoramento](./MONITORING.md) | Health checks e métricas |
| [Logs](./LOGS.md) | Sistema de logging |
| [Troubleshooting](./TROUBLESHOOTING.md) | Resolução de problemas |

---

## Stack de Operações

```
Operations Stack
├─ PM2           # Process Manager
├─ Pino          # Logging
├─ Health Checks # Endpoints de saúde
├─ Redis         # Cache monitoring
└─ Docker        # Containerização
```

---

## Quick Reference

### Verificar Status

```bash
# PM2 status
pm2 status

# Health check
curl http://localhost:3000/api/v1/health

# Logs em tempo real
pm2 logs lockdown-backend
```

---

## Referências

- [Deployment](../01-architecture/DEPLOYMENT.md)
- [Logging](../03-core/LOGGING.md)
