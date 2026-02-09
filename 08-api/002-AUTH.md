# API - Autenticação

## JWT Token

A API usa JWT (JSON Web Token) para autenticação.

---

## Gerar Token

O bot envia um token JWT pré-assinado com o `JWT_SECRET`.

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

## Usar em Requests

### Header

```http
Authorization: Bearer <JWT_TOKEN>
```

### Exemplo cURL

```bash
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl -H "Authorization: Bearer $TOKEN" \
  http://localhost:3000/api/v1/core/users
```

### Exemplo JavaScript

```typescript
const api = axios.create({
  baseURL: 'http://localhost:3000/api/v1',
  headers: {
    Authorization: `Bearer ${TOKEN}`,
  },
});
```

---

## Erros de Autenticação

### Token Inválido

```json
{
  "error": "Unauthorized",
  "code": "AUTH_INVALID_TOKEN",
  "message": "Invalid or malformed token",
  "status": 401
}
```

### Token Expirado

```json
{
  "error": "Unauthorized",
  "code": "AUTH_TOKEN_EXPIRED",
  "message": "Token has expired",
  "status": 401
}
```

### Token Ausente

```json
{
  "error": "Unauthorized",
  "code": "AUTH_NO_TOKEN",
  "message": "Authorization header required",
  "status": 401
}
```

---

## Renovar Token

Tokens expiram após 24h. Gere um novo token antes da expiração.

```typescript
// Verificar expiração
const decoded = jwt.decode(token);
const expiresAt = decoded.exp * 1000;

if (Date.now() > expiresAt - 3600000) {
  // Renovar se expira em menos de 1h
  token = generateToken(botId);
}
```

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Errors](./ERRORS.md)
