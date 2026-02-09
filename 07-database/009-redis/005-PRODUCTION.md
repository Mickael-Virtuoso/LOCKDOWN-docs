# Redis - Production Deployment

## Visão Geral

Guia completo para deploy e gerenciamento de Redis em produção com alta disponibilidade, persistência e monitoring.

---

## Persistência de Dados

Redis em produção deve ter persistência habilitada para não perder dados em caso de restart.

### RDB (Snapshot)

Salva snapshots do dataset em intervalos:

```conf
# redis.conf
save 900 1      # 900s (15min) se >= 1 key mudou
save 300 10     # 300s (5min) se >= 10 keys mudaram
save 60 10000   # 60s (1min) se >= 10000 keys mudaram

dbfilename dump.rdb
dir /data
```

**Prós:**
- Backup simples (um arquivo)
- Melhor performance
- Menor consumo de CPU

**Contras:**
- Pode perder dados entre snapshots
- Fork de processo pode pausar Redis

### AOF (Append Only File)

Log de todas as operações de escrita:

```conf
# redis.conf
appendonly yes
appendfilename "appendonly.aof"

# Sync strategy
appendfsync everysec   # Melhor balance (recomendado)
# appendfsync always   # Mais seguro, mais lento
# appendfsync no       # Mais rápido, menos seguro
```

**Prós:**
- Perda mínima de dados (1s)
- Log humanamente legível
- Auto-rewrite quando grande

**Contras:**
- Arquivo maior que RDB
- Mais lento que RDB
- Maior uso de I/O

### Recomendação: RDB + AOF

```conf
# Use ambos para máxima durabilidade
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec
```

---

## High Availability

### Redis Sentinel

Monitora Redis e faz failover automático.

```conf
# sentinel.conf
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

Arquitetura:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Sentinel 1  │     │  Sentinel 2  │     │  Sentinel 3  │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                    │
       └────────────────────┼────────────────────┘
                            │
              ┌─────────────┴──────────────┐
              │                            │
        ┌─────▼─────┐              ┌──────▼──────┐
        │   Master  │──────────────│   Replica   │
        └───────────┘   Replication└─────────────┘
```

### Redis Cluster

Para escalar horizontalmente:

```conf
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

Mínimo 6 nodes (3 masters + 3 replicas):

```
Master 1 (0-5460)  ──► Replica 1
Master 2 (5461-10922) ──► Replica 2
Master 3 (10923-16383) ──► Replica 3
```

---

## Configuração de Produção

### redis.conf Otimizado

```conf
# Network
bind 0.0.0.0
port 6379
timeout 0
tcp-keepalive 300

# Security
requirepass your-strong-password-here
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command CONFIG ""

# Memory
maxmemory 4gb
maxmemory-policy allkeys-lru  # Evict least recently used

# Persistence
save 900 1
save 300 10
save 60 10000
appendonly yes
appendfsync everysec

# Replication
replica-read-only yes
repl-diskless-sync yes

# Logging
loglevel notice
logfile /var/log/redis/redis.log

# Performance
tcp-backlog 511
databases 16
lazyfree-lazy-eviction yes
lazyfree-lazy-expire yes
```

---

## Deploy Options

### 1. Managed Services (Recomendado)

**AWS ElastiCache**
```yaml
Tier: cache.r6g.large (13.07 GB)
Engine: Redis 7.x
Multi-AZ: Yes
Automatic Failover: Yes
Backup: Daily snapshots
```

**Railway**
```yaml
Plan: Pro ($5/mo)
Memory: 512 MB - 8 GB
Backup: Automatic
Regions: Multiple
```

**Redis Cloud**
```yaml
Plan: Paid tiers
Memory: Customizable
Multi-region: Yes
```

### 2. Docker Compose (Self-hosted)

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis-production
    restart: unless-stopped
    ports:
      - "6379:6379"
    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD}
      --maxmemory 4gb
      --maxmemory-policy allkeys-lru
      --save 900 1
      --save 300 10
      --save 60 10000
      --appendonly yes
      --appendfsync everysec
    volumes:
      - redis-data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - lockdown-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  redis-data:

networks:
  lockdown-network:
    driver: bridge
```

### 3. Kubernetes

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
        resources:
          requests:
            memory: "4Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 20Gi
```

---

## Monitoring

### Metrics to Track

```yaml
Performance:
  - ops/sec
  - latency (p50, p95, p99)
  - hit rate (%)
  - network I/O

Memory:
  - used_memory
  - used_memory_peak
  - mem_fragmentation_ratio
  - evicted_keys

Connections:
  - connected_clients
  - blocked_clients
  - rejected_connections

Persistence:
  - rdb_last_save_time
  - aof_last_rewrite_time
  - aof_current_size
```

### Prometheus Exporter

```yaml
# docker-compose.yml
redis-exporter:
  image: oliver006/redis_exporter:latest
  ports:
    - "9121:9121"
  environment:
    REDIS_ADDR: redis://redis:6379
    REDIS_PASSWORD: ${REDIS_PASSWORD}
  depends_on:
    - redis
```

### Grafana Dashboard

Importe dashboard: https://grafana.com/grafana/dashboards/11835

---

## Backup e Recovery

### Backup Automático

```bash
#!/bin/bash
# backup-redis.sh

BACKUP_DIR="/backups/redis"
DATE=$(date +%Y%m%d_%H%M%S)

# Trigger BGSAVE
redis-cli -a $REDIS_PASSWORD BGSAVE

# Wait for save to complete
while [ $(redis-cli -a $REDIS_PASSWORD LASTSAVE) -eq $LASTSAVE ]; do
  sleep 1
done

# Copy dump file
cp /data/dump.rdb "$BACKUP_DIR/dump_$DATE.rdb"
cp /data/appendonly.aof "$BACKUP_DIR/aof_$DATE.aof"

# Compress
gzip "$BACKUP_DIR/dump_$DATE.rdb"
gzip "$BACKUP_DIR/aof_$DATE.aof"

# Delete backups older than 7 days
find $BACKUP_DIR -name "*.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

### Restore

```bash
# 1. Stop Redis
docker-compose stop redis

# 2. Copy backup
cp backup/dump_XXXXXXXX.rdb.gz /data/
gunzip /data/dump_XXXXXXXX.rdb.gz
mv /data/dump_XXXXXXXX.rdb /data/dump.rdb

# 3. Start Redis
docker-compose start redis
```

---

## Security

### Checklist

```yaml
✅ requirepass definido
✅ bind apenas IPs necessários
✅ Comandos perigosos desabilitados (FLUSHALL, CONFIG)
✅ TLS/SSL em produção
✅ Firewall configurado (apenas backend pode acessar)
✅ Logs habilitados
✅ Monitoring ativo
✅ Backups automáticos
```

### TLS Configuration

```conf
# redis.conf
tls-port 6380
port 0  # Disable non-TLS

tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt

tls-auth-clients no  # Optional client auth
```

---

## Performance Tuning

### Linux Kernel Settings

```bash
# /etc/sysctl.conf
vm.overcommit_memory = 1
net.core.somaxconn = 511
```

### Transparent Huge Pages

```bash
# Disable THP (melhora performance)
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

### Max Connections

```conf
# redis.conf
maxclients 10000
```

---

## Troubleshooting

### Alta Latência

```bash
# Ver slow log
redis-cli --latency
redis-cli SLOWLOG GET 10

# Analisar comandos lentos
redis-cli --bigkeys
```

### Memory Issues

```bash
# Ver uso de memória
redis-cli INFO memory

# Ver keys consumindo mais memória
redis-cli --memkeys

# Analisar evictions
redis-cli INFO stats | grep evicted_keys
```

### Connection Issues

```bash
# Ver clients conectados
redis-cli CLIENT LIST

# Matar client travado
redis-cli CLIENT KILL 127.0.0.1:12345
```

---

## Disaster Recovery Plan

### 1. Redis Down

```yaml
1. Verificar logs: docker logs redis
2. Tentar restart: docker-compose restart redis
3. Se persistir, restore from backup
4. Notificar equipe
```

### 2. Data Corruption

```yaml
1. Stop Redis
2. Verify backup integrity
3. Restore from latest valid backup
4. Start Redis
5. Validate data
6. Investigate root cause
```

### 3. High Memory

```yaml
1. Identify large keys: redis-cli --bigkeys
2. Set eviction policy if not set
3. Increase memory limit (if possible)
4. Clear unnecessary data
5. Optimize key TTLs
```

---

## Cost Optimization

### Memory Sizing

```typescript
// Estimate memory needed
Users: 100,000 users * 1KB/user = 100MB
Sessions: 10,000 sessions * 500B/session = 5MB
Cache: 50MB
Pub/Sub buffer: 10MB
Overhead (30%): 50MB
──────────────────────────────────────────
Total: ~215MB → Provision 512MB
```

### Compression

```typescript
import { compress, decompress } from 'lz-string';

// Before storing
const compressed = compress(JSON.stringify(largeObject));
await redis.set(key, compressed, 300);

// When retrieving
const data = await redis.get(key);
const original = JSON.parse(decompress(data));
```

---

**Redis production-ready!** 🚀🔴
