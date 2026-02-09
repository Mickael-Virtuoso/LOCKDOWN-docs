# 🌐 REST API Documentation

## Visão Geral

Documentação completa da API REST do LOCKDOWN. Referência de todos os endpoints, autenticação e exemplos de uso.

---

## 📖 Ordem de Leitura Recomendada

1. **[001-OVERVIEW.md](./001-OVERVIEW.md)** ⭐ **COMECE AQUI!**
   - Visão geral da API
   - Base URL e versionamento
   - Formato de requests/responses
   - Status codes

2. **[002-AUTH.md](./002-AUTH.md)**
   - Autenticação e autorização
   - JWT tokens
   - OAuth2 flow
   - API keys

3. **[003-CORE.md](./003-CORE.md)**
   - Core endpoints
   - `/users`, `/guilds`, `/health`
   - CRUD operations
   - Endpoints essenciais

4. **[004-MODERATION.md](./004-MODERATION.md)**
   - Moderation endpoints
   - `/moderation/*`
   - Punishments, warnings, bans
   - Moderation logs

5. **[005-CONFIG.md](./005-CONFIG.md)**
   - Configuration endpoints
   - `/config/*`
   - Guild settings
   - Policy configuration

6. **[006-AUDIT.md](./006-AUDIT.md)**
   - Audit endpoints
   - `/audit/*`
   - Activity logs
   - Audit trail

7. **[007-ERRORS.md](./007-ERRORS.md)**
   - Error handling
   - Error format
   - Error codes
   - Troubleshooting

8. **[008-RATELIMIT.md](./008-RATELIMIT.md)**
   - Rate limiting
   - Limites por endpoint
   - Headers de rate limit
   - Como lidar com 429

9. **[009-EXAMPLES.md](./009-EXAMPLES.md)**
   - Exemplos práticos
   - SDKs e clients
   - Code samples
   - Postman collection

10. **[010-API.md](./010-API.md)**
    - Índice completo
    - Referência rápida
    - OpenAPI/Swagger

---

## 🎯 API Base

```
Base URL: https://api.lockdown.dev/v1
Development: http://localhost:3000/api/v1
```

---

## 🔑 Autenticação

```bash
# Bearer token
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  https://api.lockdown.dev/v1/users/me
```

---

## 📊 Endpoints Principais

```
GET    /health           - Health check
GET    /users/me         - User atual
GET    /guilds           - Lista de guilds
POST   /moderation/warn  - Avisar usuário
GET    /audit/logs       - Logs de auditoria
```

---

## 💡 Quick Start

```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: 'http://localhost:3000/api/v1',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

// Get current user
const { data } = await api.get('/users/me');
console.log(data);
```

---

## 📝 Formato de Response

```json
{
  "success": true,
  "data": {
    "id": "123",
    "name": "John Doe"
  },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

---

## ⚠️ Error Response

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid token",
    "details": {}
  }
}
```

---

## 🔒 Rate Limiting

```
Headers:
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1640000000
```

---

## 🔗 Documentação Relacionada

- **[../04-backend/](../04-backend/)** - Backend implementation
- **[../09-security/](../09-security/)** - Security best practices
- **[../07-database/](../07-database/)** - Database schemas

---

## 📚 Recursos

- Swagger UI: `/api/docs`
- Postman Collection: `./postman/`
- OpenAPI Spec: `/api/openapi.json`

---

**APIs RESTful bem documentadas!** 🌐
