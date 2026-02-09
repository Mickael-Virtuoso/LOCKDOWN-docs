# 🔴 Redis Documentation

## Visão Geral

Documentação completa do Redis no LOCKDOWN. Redis é usado para cache, Pub/Sub de eventos e sessões tanto em desenvolvimento quanto em produção.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - O que é Redis
   - Por que usamos Redis
   - Casos de uso no LOCKDOWN
   - Driver: ioredis

2. **[002-SETUP.md](./002-SETUP.md)**
   - Instalação local (Docker)
   - Configuração do ioredis
   - Variáveis de ambiente
   - Conexão e health check

3. **[003-PUBSUB.md](./003-PUBSUB.md)**
   - Sistema Pub/Sub de eventos
   - Event-driven architecture
   - Comunicação entre bots e backend
   - Exemplos práticos

4. **[004-CACHING.md](./004-CACHING.md)**
   - Estratégias de cache
   - Cache-aside pattern
   - TTL e invalidação
   - Cache de queries e APIs

5. **[005-PRODUCTION.md](./005-PRODUCTION.md)**
   - Redis em produção
   - Persistência (RDB + AOF)
   - Redis Cluster
   - Monitoring e troubleshooting

---

## 🎯 Redis no LOCKDOWN

### Em Development
```
- Docker container local
- Sem persistência (dados perdidos ao reiniciar)
- Usado para Pub/Sub e cache
- Facilita desenvolvimento local
```

### Em Production
```
- Redis gerenciado (AWS ElastiCache, Railway, etc)
- Persistência habilitada (RDB + AOF)
- High availability (replication)
- Monitoring e alertas
```

---

## ⚡ Quick Start

```bash
# Docker local
docker run -d --name redis -p 6379:6379 redis:alpine

# Testar conexão
redis-cli ping
# PONG

# No código
import Redis from 'ioredis';
const redis = new Redis({
  host: 'localhost',
  port: 6379
});

await redis.set('key', 'value');
const value = await redis.get('key');
```

---

## 🔗 Documentação Relacionada

- **[../../02-platforms/002-EVENTS.md](../../02-platforms/002-EVENTS.md)** - Sistema de eventos
- **[../../04-backend/](../../04-backend/)** - Integração no backend

---

**Cache rápido e eventos em tempo real!** 🔴
