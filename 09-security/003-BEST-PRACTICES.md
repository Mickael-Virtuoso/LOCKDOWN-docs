# Boas Práticas de Segurança

Guidelines de segurança para o LOCKDOWN.

---

## Variáveis de Ambiente

### Nunca no Código

```typescript
// ❌ NUNCA
const secret = 'minha-senha-secreta';

// ✅ SEMPRE
const secret = process.env.JWT_SECRET;
```

### .env no .gitignore

```gitignore
.env
.env.local
.env.production
```

---

## Validação de Input

### Sanitizar Dados

```typescript
import { z } from 'zod';

const BanSchema = z.object({
  guildId: z.string().regex(/^\d{17,19}$/),
  userId: z.string().regex(/^\d{17,19}$/),
  reason: z.string().max(500).optional(),
});

// Uso
const data = BanSchema.parse(req.body);
```

### SQL Injection

```typescript
// ❌ NUNCA
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// ✅ SEMPRE (Drizzle ORM)
const user = await db.select().from(usersTable)
  .where(eq(usersTable.userId, userId));
```

---

## Rate Limiting

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minuto
  max: 100,
  message: { error: 'Too many requests' },
});

app.use('/api/', limiter);
```

---

## CORS

```typescript
import cors from 'cors';

app.use(cors({
  origin: ['https://lockdown.app'],
  methods: ['GET', 'POST', 'PATCH', 'DELETE'],
  credentials: true,
}));
```

---

## HTTPS

```typescript
// Redirecionar HTTP para HTTPS
app.use((req, res, next) => {
  if (req.headers['x-forwarded-proto'] !== 'https') {
    return res.redirect(`https://${req.hostname}${req.url}`);
  }
  next();
});
```

---

## Headers de Segurança

```typescript
import helmet from 'helmet';

app.use(helmet());
// Adiciona:
// - X-Content-Type-Options
// - X-Frame-Options
// - Content-Security-Policy
// - etc
```

---

## Logging Seguro

```typescript
// ❌ NUNCA logar dados sensíveis
logger.info(`User login: ${email}, password: ${password}`);

// ✅ SEMPRE sanitizar
logger.info(`User login: ${email}`);
```

---

## Senhas e Secrets

### Hashing

```typescript
import bcrypt from 'bcrypt';

// Hash
const hash = await bcrypt.hash(password, 12);

// Verificar
const match = await bcrypt.compare(password, hash);
```

### Rotação de Secrets

1. Gere novo secret
2. Atualize em produção
3. Invalide tokens antigos (gradualmente)
4. Remova secret antigo

---

## Checklist

- [ ] Variáveis sensíveis em .env
- [ ] Input validado e sanitizado
- [ ] Rate limiting ativo
- [ ] CORS configurado
- [ ] HTTPS em produção
- [ ] Headers de segurança
- [ ] Logs sem dados sensíveis
- [ ] Secrets rotacionados

---

## Links Relacionados

- [Autenticação](./AUTHENTICATION.md)
- [Autorização](./AUTHORIZATION.md)
- [SECURITY.md](../../SECURITY.md)
