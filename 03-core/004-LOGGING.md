# Sistema de Logging

Guia do sistema de logging centralizado do LOCKDOWN usando Pino.

---

## Visão Geral

O LOCKDOWN usa Pino como logger por sua performance e estrutura JSON.

```typescript
import { getLogger } from '@lockdown/shared/logger';

const logger = getLogger('MeuServico');
logger.info('Mensagem de log');
```

---

## Níveis de Log

| Nível | Valor | Uso |
|-------|-------|-----|
| `trace` | 10 | Debug detalhado |
| `debug` | 20 | Informações de debug |
| `info` | 30 | Operações normais |
| `warn` | 40 | Alertas |
| `error` | 50 | Erros |
| `fatal` | 60 | Erros críticos |

---

## Uso Básico

### Criando Logger

```typescript
import { getLogger } from '@lockdown/shared/logger';

const logger = getLogger('BanService');
```

### Métodos de Log

```typescript
logger.trace('Detalhes de execução');
logger.debug('Informação de debug');
logger.info('Operação completada');
logger.warn('Algo pode estar errado');
logger.error('Erro na operação');
logger.fatal('Sistema comprometido');
```

---

## Formato de Saída

### Desenvolvimento (Pretty)

```
[03/02/2025 10:30:45 UTC] info: [BanService] "✅ Ban created successfully"
[03/02/2025 10:30:46 UTC] warn: [BanService] "⚠️ User already banned"
[03/02/2025 10:30:47 UTC] error: [BanService] "❌ Failed to create ban"
```

### Produção (JSON)

```json
{"level":30,"time":1706958645000,"name":"BanService","msg":"Ban created successfully"}
```

---

## Convenções

### Prefixo de Classe

```typescript
// ✅ Correto
logger.info(`[${this.constructor.name}] "✅ operation completed"`);

// ❌ Incorreto
logger.info('operation completed');
```

### Emojis por Tipo

```typescript
logger.info(`"✅ success"`);   // Sucesso
logger.warn(`"⚠️ warning"`);   // Aviso
logger.error(`"❌ error"`);    // Erro
logger.debug(`"🔍 debug"`);    // Debug
```

---

## Contexto Adicional

```typescript
// Com objeto de contexto
logger.info({ userId, guildId }, 'Ban created');

// Com erro
logger.error({ err: error }, 'Failed to create ban');
```

---

## Configuração

### Variáveis de Ambiente

```ini
LOG_LEVEL=debug    # Nível mínimo de log
NODE_ENV=development  # Ativa pretty print
```

### Arquivo de Configuração

```typescript
// packages/shared/src/logger/config/levels.ts
export const levels = {
  trace: 10,
  debug: 20,
  info: 30,
  warn: 40,
  error: 50,
  fatal: 60
};
```

---

## HTTP Request Logging

```typescript
import { httpLogger } from '@lockdown/shared/logger';

app.use(httpLogger);

// Output:
// [03/02/2025 10:30:45 UTC] info: [HTTP] "POST /api/v1/bans 201 45ms"
```

---

## Boas Práticas

### Faça
- Use níveis apropriados
- Inclua contexto relevante
- Use prefixos de classe
- Log erros com stack trace

### Não Faça
- Log dados sensíveis (senhas, tokens)
- Log em excesso em produção
- Usar console.log
- Ignorar erros

---

## Exemplo Completo

```typescript
import { getLogger } from '@lockdown/shared/logger';

const logger = getLogger('BanService');

class BanService {
  async createBan(data: CreateBanDTO): Promise<Ban> {
    logger.debug({ data }, `[BanService] "🔍 Creating ban"`);
    
    try {
      const ban = await this.repository.create(data);
      logger.info({ banId: ban.id }, `[BanService] "✅ Ban created"`);
      return ban;
    } catch (error) {
      logger.error({ err: error }, `[BanService] "❌ Failed to create ban"`);
      throw error;
    }
  }
}
```

---

**Logs bem estruturados facilitam debugging!**
