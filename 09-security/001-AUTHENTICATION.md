# Autenticação

Sistema de autenticação do LOCKDOWN.

---

## JWT (JSON Web Token)

### Estrutura

```
Header.Payload.Signature
```

### Payload

```json
{
  "botId": "123456789",
  "iat": 1704067200,
  "exp": 1704153600
}
```

---

## Geração de Token

```typescript
import jwt from 'jsonwebtoken';

export function generateToken(botId: string): string {
  return jwt.sign(
    { botId, iat: Date.now() },
    process.env.JWT_SECRET,
    { expiresIn: '24h' }
  );
}
```

---

## Validação de Token

```typescript
import jwt from 'jsonwebtoken';

export function validateToken(token: string) {
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    return { valid: true, payload: decoded };
  } catch (error) {
    return { valid: false, error: error.message };
  }
}
```

---

## Middleware

```typescript
export async function authMiddleware(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({
      error: 'Unauthorized',
      code: 'AUTH_NO_TOKEN',
    });
  }

  const token = authHeader.split(' ')[1];
  const result = validateToken(token);

  if (!result.valid) {
    return res.status(401).json({
      error: 'Unauthorized',
      code: 'AUTH_INVALID_TOKEN',
    });
  }

  req.botId = result.payload.botId;
  next();
}
```

---

## Variáveis de Ambiente

```env
JWT_SECRET=<strong-random-secret>
JWT_EXPIRES_IN=24h
```

### Gerar Secret Seguro

```bash
openssl rand -base64 64
```

---

## Boas Práticas

1. **Nunca expor JWT_SECRET** no código
2. **Rotacionar secrets** periodicamente
3. **Tokens curtos** (24h ou menos)
4. **HTTPS sempre** em produção

---

## Links Relacionados

- [Autorização](./AUTHORIZATION.md)
- [API Auth](../08-api/AUTH.md)
