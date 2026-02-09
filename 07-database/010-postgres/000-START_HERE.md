# 🐘 PostgreSQL Documentation

## Visão Geral

Documentação completa do PostgreSQL no LOCKDOWN. PostgreSQL é o banco de dados principal usado em **produção** com todas as features avançadas habilitadas.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - O que é PostgreSQL
   - Por que usamos PostgreSQL em produção
   - Features do PostgreSQL não disponíveis no SQLite
   - Driver: pg + Drizzle ORM

2. **[002-SCHEMAS.md](./002-SCHEMAS.md)**
   - Definições de todas as tabelas
   - Schemas Drizzle para PostgreSQL
   - Tipos de dados PostgreSQL
   - Constraints e validações

3. **[003-RELATIONSHIPS.md](./003-RELATIONSHIPS.md)**
   - Foreign keys e relações
   - One-to-many, many-to-many
   - Relational queries com Drizzle
   - Diagramas ER

4. **[004-MIGRATIONS.md](./004-MIGRATIONS.md)**
   - Sistema de migrations Drizzle
   - Gerar e aplicar migrations
   - Migrar de SQLite para PostgreSQL
   - Rollbacks e versionamento

5. **[005-QUERIES.md](./005-QUERIES.md)**
   - Exemplos de queries Drizzle
   - CRUD operations
   - Joins complexos
   - Query optimization

6. **[006-PERFORMANCE.md](./006-PERFORMANCE.md)**
   - Otimização de queries
   - Índices estratégicos
   - N+1 problem solutions
   - Query analysis

7. **[007-BACKUP.md](./007-BACKUP.md)**
   - Estratégias de backup
   - pg_dump e pg_restore
   - Automated backups
   - Disaster recovery

8. **[008-DATABASE.md](./008-DATABASE.md)**
   - Índice completo
   - Referência rápida
   - Comandos úteis PostgreSQL

---

## 🎯 PostgreSQL no LOCKDOWN

### Por Que PostgreSQL?

```
✅ Features SQL completas (JOINs complexos, subqueries, CTEs)
✅ JSONB nativo (queries em JSON)
✅ Full-text search
✅ Triggers e stored procedures
✅ Array types
✅ Transações ACID completas
✅ Row-level security
✅ Particionamento de tabelas
✅ Replicação e alta disponibilidade
```

### Development vs Production

```yaml
Development:
  Database: SQLite (file:./dev.db)
  Features: Limitadas
  Performance: Suficiente para dev
  Migration: Não necessária

Production:
  Database: PostgreSQL (hosted)
  Features: Completas
  Performance: Escalável
  Migration: Deploy automatizado
```

---

## ⚡ Quick Start

```bash
# Docker local
docker run -d \
  --name lockdown-postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=lockdown \
  -p 5432:5432 \
  postgres:16-alpine

# Configurar Drizzle
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/lockdown
DATABASE_PROVIDER=postgres

# Run migrations
pnpm db:migrate

# Open Drizzle Studio
pnpm db:studio
```

---

## 🔗 Documentação Relacionada

- **[../011-sqlite/](../011-sqlite/)** - SQLite (development)
- **[../../04-backend/](../../04-backend/)** - Backend integration

---

**Banco de dados robusto para produção!** 🐘
