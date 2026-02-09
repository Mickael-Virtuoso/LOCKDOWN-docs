# API - Error Handling

## Error Response Format

```json
{
  "error": "Bad Request",
  "code": "VALIDATION_ERROR",
  "message": "Guild ID is required",
  "status": 400,
  "requestId": "req_123"
}
```

---

## Status Codes

| Code | Meaning | Descrição |
|------|---------|-----------|
| 200 | OK | Request bem-sucedido |
| 201 | Created | Recurso criado |
| 400 | Bad Request | Validação falhou |
| 401 | Unauthorized | Token inválido/ausente |
| 403 | Forbidden | Sem permissão |
| 404 | Not Found | Recurso não existe |
| 409 | Conflict | Recurso já existe |
| 429 | Too Many Requests | Rate limit excedido |
| 500 | Server Error | Erro interno |

---

## Error Codes

### Autenticação

| Code | Descrição |
|------|-----------|
| `AUTH_NO_TOKEN` | Header Authorization ausente |
| `AUTH_INVALID_TOKEN` | Token malformado/inválido |
| `AUTH_TOKEN_EXPIRED` | Token expirado |

### Validação

| Code | Descrição |
|------|-----------|
| `VALIDATION_ERROR` | Erro genérico de validação |
| `INVALID_USER_ID` | User ID inválido |
| `INVALID_GUILD_ID` | Guild ID inválido |
| `MISSING_REQUIRED_FIELD` | Campo obrigatório ausente |

### Recursos

| Code | Descrição |
|------|-----------|
| `USER_NOT_FOUND` | Usuário não encontrado |
| `GUILD_NOT_FOUND` | Guild não encontrada |
| `BAN_NOT_FOUND` | Ban não encontrado |
| `MEMBER_NOT_FOUND` | Membro não encontrado |

### Conflitos

| Code | Descrição |
|------|-----------|
| `USER_ALREADY_EXISTS` | Usuário já existe |
| `USER_ALREADY_BANNED` | Usuário já banido |
| `USER_NOT_BANNED` | Usuário não está banido |

### Rate Limiting

| Code | Descrição |
|------|-----------|
| `RATE_LIMITED` | Limite de requisições excedido |

---

## Exemplos

### Validação

```json
{
  "error": "Validation Error",
  "code": "INVALID_USER_ID",
  "message": "User ID must be a valid Discord snowflake",
  "status": 400
}
```

### Não Encontrado

```json
{
  "error": "Not Found",
  "code": "BAN_NOT_FOUND",
  "message": "Ban with ID 999 not found",
  "status": 404
}
```

### Conflito

```json
{
  "error": "Conflict",
  "code": "USER_ALREADY_BANNED",
  "message": "User is already banned in this guild",
  "status": 409
}
```

### Rate Limit

```json
{
  "error": "Too Many Requests",
  "code": "RATE_LIMITED",
  "message": "Rate limit exceeded. Reset in 30 seconds",
  "status": 429,
  "retryAfter": 30
}
```

---

## Tratamento no Cliente

### JavaScript

```typescript
try {
  const response = await api.post('/moderation/bans', data);
} catch (error) {
  if (axios.isAxiosError(error)) {
    const { code, message } = error.response?.data || {};

    switch (code) {
      case 'USER_ALREADY_BANNED':
        console.log('Usuário já está banido');
        break;
      case 'RATE_LIMITED':
        const retryAfter = error.response?.data.retryAfter;
        console.log(`Aguarde ${retryAfter} segundos`);
        break;
      default:
        console.error(message);
    }
  }
}
```

---

## Links Relacionados

- [Rate Limiting](./RATELIMIT.md)
- [Visão Geral](./OVERVIEW.md)
