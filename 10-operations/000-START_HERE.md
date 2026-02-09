# 📊 Operations & Monitoring

## Visão Geral

Documentação de operações, monitoramento e troubleshooting para manter o LOCKDOWN rodando em produção.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-MONITORING.md](./001-MONITORING.md)** ⭐ **COMECE AQUI!**
   - Sistema de monitoramento
   - Métricas e dashboards
   - Alertas e notificações
   - Health checks

2. **[002-LOGS.md](./002-LOGS.md)**
   - Sistema de logs centralizado
   - Log aggregation
   - Log levels e filtering
   - Log analysis

3. **[003-TROUBLESHOOTING.md](./003-TROUBLESHOOTING.md)**
   - Problemas comuns em produção
   - Debugging procedures
   - Emergency procedures
   - Runbooks

4. **[004-OPERATIONS.md](./004-OPERATIONS.md)**
   - Índice completo
   - SOP (Standard Operating Procedures)
   - On-call procedures
   - Escalation matrix

---

## 🎯 Pilares de Operations

```
├── Monitoring   - Observar o sistema
├── Logging      - Registrar eventos
├── Alerting     - Notificar problemas
└── Debugging    - Resolver issues
```

---

## 📈 Métricas Principais

```
├── Uptime               - 99.9% SLA
├── Response Time        - < 200ms p95
├── Error Rate           - < 0.1%
├── CPU/Memory Usage     - < 70%
└── Database Connections - < 80% pool
```

---

## 🚨 Alertas Críticos

Alertas que requerem ação imediata:
- 🔴 **API Down** - Sistema indisponível
- 🔴 **Database Down** - Banco inacessível
- 🟠 **High Error Rate** - > 1% errors
- 🟠 **High Latency** - > 1s p95
- 🟡 **Disk Space Low** - < 10% free

---

## 📊 Dashboards

```
├── Application Dashboard
│   ├── Request rate
│   ├── Response time
│   ├── Error rate
│   └── Active users
│
├── Infrastructure Dashboard
│   ├── CPU usage
│   ├── Memory usage
│   ├── Disk I/O
│   └── Network traffic
│
└── Database Dashboard
    ├── Query performance
    ├── Connection pool
    ├── Slow queries
    └── Lock waits
```

---

## 🔍 Log Levels

```
TRACE  - Informação muito detalhada
DEBUG  - Debugging information
INFO   - Informações gerais
WARN   - Avisos (não crítico)
ERROR  - Erros que precisam atenção
FATAL  - Sistema vai parar
```

---

## 💡 Quick Troubleshooting

### API não responde

```bash
# 1. Verificar se está rodando
pm2 status

# 2. Ver logs
pm2 logs backend --lines 100

# 3. Verificar health
curl http://localhost:3000/health

# 4. Restart se necessário
pm2 restart backend
```

### Database slow

```bash
# Ver queries ativas
pnpm db:studio

# Ver locks
# (PostgreSQL)
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

---

## 🔧 Ferramentas

```
├── pm2          - Process manager
├── Grafana      - Dashboards
├── Prometheus   - Metrics collection
├── Pino         - Logging
└── Drizzle      - Database monitoring
```

---

## 🔗 Documentação Relacionada

- **[../04-backend/](../04-backend/)** - Backend architecture
- **[../07-database/](../07-database/)** - Database performance
- **[../09-security/](../09-security/)** - Security monitoring

---

## 📞 Contatos de Emergência

```
On-Call: +55 XX XXXX-XXXX
Email: ops@lockdown.dev
Slack: #ops-alerts
```

---

**Mantenha o sistema saudável!** 📊
