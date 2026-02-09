# Database - Backup e Restore

## SQLite (Development)

### Backup

```bash
# Copiar arquivo
cp packages/backend/dev.db packages/backend/dev.db.backup

# Com timestamp
cp packages/backend/dev.db "packages/backend/backups/dev_$(date +%Y%m%d_%H%M%S).db"
```

### Restore

```bash
cp packages/backend/dev.db.backup packages/backend/dev.db
```

---

## PostgreSQL (Production)

### Backup Completo

```bash
# Dump SQL
pg_dump -U postgres lockdown > backup.sql

# Dump comprimido
pg_dump -U postgres lockdown | gzip > backup.sql.gz

# Dump com timestamp
pg_dump -U postgres lockdown > "backup_$(date +%Y%m%d_%H%M%S).sql"
```

### Backup por Schema

```bash
# Apenas schema core
pg_dump -U postgres -n core lockdown > backup_core.sql

# Apenas schema moderation
pg_dump -U postgres -n moderation lockdown > backup_moderation.sql
```

### Restore

```bash
# Restore simples
psql -U postgres lockdown < backup.sql

# Restore comprimido
gunzip < backup.sql.gz | psql -U postgres lockdown

# Restore em novo banco
createdb -U postgres lockdown_restored
psql -U postgres lockdown_restored < backup.sql
```

---

## Automação

### Script de Backup

```bash
#!/bin/bash
# scripts/backup.sh

BACKUP_DIR="/var/backups/lockdown"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
FILENAME="lockdown_${TIMESTAMP}.sql.gz"

mkdir -p $BACKUP_DIR

pg_dump -U postgres lockdown | gzip > "$BACKUP_DIR/$FILENAME"

# Manter apenas últimos 7 dias
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete

echo "Backup created: $FILENAME"
```

### Cron Job

```bash
# Backup diário às 3h
0 3 * * * /home/deploy/lockdown/scripts/backup.sh
```

---

## Checklist de Backup

- [ ] Backup automático configurado
- [ ] Backups testados (restore funciona)
- [ ] Retenção definida (ex: 7 dias)
- [ ] Armazenamento externo (S3, etc)
- [ ] Monitoramento de falhas

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Deployment](../01-architecture/DEPLOYMENT.md)
