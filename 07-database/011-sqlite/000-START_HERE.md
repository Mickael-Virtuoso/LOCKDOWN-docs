# 📁 SQLite Documentation

## Visão Geral

Documentação completa do SQLite no LOCKDOWN. SQLite é usado em **development** para desenvolvimento local rápido e simples.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - O que é SQLite
   - Por que usamos SQLite em development
   - Vantagens e limitações
   - Driver: better-sqlite3 + Drizzle ORM

2. **[002-SETUP.md](./002-SETUP.md)**
   - Setup local
   - Configuração do Drizzle para SQLite
   - Desenvolvimento com SQLite
   - Drizzle Studio

3. **[003-LIMITATIONS.md](./003-LIMITATIONS.md)**
   - Limitações do SQLite vs PostgreSQL
   - Features não disponíveis
   - Quando migrar para PostgreSQL
   - Workarounds

4. **[004-DEVELOPMENT.md](./004-DEVELOPMENT.md)**
   - Workflow de desenvolvimento
   - Migrations em dev
   - Testes com SQLite
   - Seed data

---

## 🎯 SQLite no LOCKDOWN

### Por Que SQLite em Development?

```
✅ Zero configuração (arquivo único)
✅ Sem servidor externo necessário
✅ Setup instantâneo
✅ Perfeito para dev local
✅ Fast enough para desenvolvimento
✅ Fácil de resetar (delete file)
```

### Development Only

```yaml
Development:
  Database: SQLite (file:./dev.db)
  Setup: Automático
  Migrations: Drizzle Kit
  Reset: rm dev.db && pnpm db:migrate

Production:
  Database: PostgreSQL ✅
  SQLite: ❌ NÃO USAR
```

---

## ⚡ Quick Start

```bash
# Já está configurado!
# Apenas rode:

pnpm db:migrate      # Cria dev.db
pnpm db:studio       # Visualizar dados
pnpm dev             # Rodar app
```

---

**Desenvolvimento rápido com SQLite!** 📁
