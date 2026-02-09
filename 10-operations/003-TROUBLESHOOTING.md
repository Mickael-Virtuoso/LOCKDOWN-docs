# Troubleshooting

Guia de resolução de problemas do LOCKDOWN.

---

## Problemas Comuns

### Backend não inicia

**Sintoma:** Erro ao executar `pnpm dev`

**Causas:**
1. Porta já em uso
2. Variáveis de ambiente faltando
3. Dependências não instaladas

**Solução:**

```bash
# 1. Verificar porta
lsof -i :3000
kill -9 <PID>

# 2. Verificar .env
cat packages/backend/.env

# 3. Reinstalar dependências
pnpm install
```

---

### Redis não conecta

**Sintoma:** `ECONNREFUSED 127.0.0.1:6379`

**Solução:**

```bash
# Verificar se Redis está rodando
redis-cli ping

# Iniciar Redis
sudo systemctl start redis

# Verificar URL no .env
REDIS_URL=redis://localhost:6379
```

---

### Database error

**Sintoma:** `SQLITE_CANTOPEN` ou connection error

**Solução:**

```bash
# Verificar se diretório existe
mkdir -p packages/backend/data

# Verificar permissões
chmod 755 packages/backend/data

# Recriar database
rm packages/backend/data/lockdown.db
pnpm db:migrate
```

---

### Bot não responde

**Sintoma:** Comandos não funcionam

**Verificações:**

```bash
# 1. Token válido
echo $DISCORD_TOKEN

# 2. Intents habilitados no Discord Developer Portal

# 3. Bot online
pm2 status

# 4. Logs
pm2 logs lockdown-bot
```

---

### Rate limit excedido

**Sintoma:** Erro 429 Too Many Requests

**Solução:**

```typescript
// Implementar backoff
async function withBackoff(fn, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status !== 429) throw error;
      await sleep(Math.pow(2, i) * 1000);
    }
  }
}
```

---

### Build falha

**Sintoma:** TypeScript errors

**Solução:**

```bash
# Limpar cache
rm -rf node_modules/.cache
rm -rf packages/*/dist

# Verificar tipos
pnpm typecheck

# Rebuild
pnpm build
```

---

### Memory leak

**Sintoma:** Uso de memória crescente

**Diagnóstico:**

```bash
# Monitorar memória
pm2 monit

# Configurar limite
pm2 start app.js --max-memory-restart 500M
```

---

## Comandos Úteis

```bash
# Reiniciar serviços
pm2 restart all

# Limpar logs
pm2 flush

# Status completo
pm2 show lockdown-backend

# Health check
curl http://localhost:3000/api/v1/health | jq
```

---

## Coleta de Informações

Ao reportar um problema, inclua:

1. **Logs:** `pm2 logs --lines 50`
2. **Versão:** `node -v && pnpm -v`
3. **Sistema:** `uname -a`
4. **Ambiente:** dev/production
5. **Passos para reproduzir**

---

## Links Relacionados

- [FAQ](../00-introduction/FAQ.md)
- [Monitoramento](./MONITORING.md)
- [Logs](./LOGS.md)
