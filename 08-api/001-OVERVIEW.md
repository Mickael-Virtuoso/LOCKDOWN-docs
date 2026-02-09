# API - Visão Geral

## Base URL

```
Development: http://localhost:3000/api/v1
Production:  https://api.lockdown.app/api/v1
```

---

## Headers

```http
Content-Type: application/json
Authorization: Bearer <JWT_TOKEN>
X-Request-ID: <UUID> (opcional)
```

---

## Response Format

### Sucesso

```json
{
  "data": { ... },
  "meta": {
    "timestamp": "2025-02-05T10:00:00Z",
    "requestId": "req_123"
  }
}
```

### Paginação

```json
{
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 100,
    "pages": 2
  }
}
```

### Erro

```json
{
  "error": "Bad Request",
  "code": "VALIDATION_ERROR",
  "message": "Guild ID is required",
  "status": 400
}
```

---

## Versionamento

A API usa versionamento na URL:

```
/api/v1/...  → Versão atual
/api/v2/...  → Versão futura
```

---

## Content Types

| Método | Content-Type |
|--------|--------------|
| GET | - |
| POST | application/json |
| PATCH | application/json |
| DELETE | - |

---

## Links Relacionados

- [Autenticação](./AUTH.md)
- [Core Endpoints](./CORE.md)
- [Moderation Endpoints](./MODERATION.md)
- [Errors](./ERRORS.md)
