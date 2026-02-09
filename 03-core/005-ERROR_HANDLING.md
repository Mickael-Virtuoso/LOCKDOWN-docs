# Tratamento de Erros

Guia de tratamento de erros no LOCKDOWN.

---

## Hierarquia de Erros

```
AppError (base)
├── ValidationError (400)
├── AuthenticationError (401)
├── AuthorizationError (403)
├── NotFoundError (404)
├── ConflictError (409)
├── RateLimitError (429)
└── InternalError (500)
```

---

## Classes de Erro

### AppError (Base)

```typescript
import { AppError } from '@lockdown/shared/errors';

class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
  }
}
```

### ValidationError

```typescript
throw new ValidationError('User ID is required');
// HTTP 400: { error: 'User ID is required', code: 'VALIDATION_ERROR' }
```

### NotFoundError

```typescript
throw new NotFoundError('User not found');
// HTTP 404: { error: 'User not found', code: 'NOT_FOUND' }
```

---

## Middleware de Erro

```typescript
// packages/backend/src/api/middlewares/errorHandler.middleware.ts

export function errorHandler(err, req, res, next) {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  logger.error(`[ErrorHandler] ${message}`, { stack: err.stack });
  
  res.status(statusCode).json({
    error: message,
    code: err.code,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
}
```

---

## Padrões de Uso

### Controllers

```typescript
async createUser(req, res, next) {
  try {
    const user = await userService.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    next(error); // Passa para o error handler
  }
}
```

### Services

```typescript
async findById(id: string): Promise<User> {
  if (!id) {
    throw new ValidationError('ID is required');
  }
  
  const user = await userRepository.findById(id);
  
  if (!user) {
    throw new NotFoundError('User not found');
  }
  
  return user;
}
```

---

## Códigos de Erro

| Código | HTTP | Descrição |
|--------|------|-----------|
| `VALIDATION_ERROR` | 400 | Input inválido |
| `AUTHENTICATION_ERROR` | 401 | Não autenticado |
| `AUTHORIZATION_ERROR` | 403 | Sem permissão |
| `NOT_FOUND` | 404 | Recurso não encontrado |
| `CONFLICT` | 409 | Recurso já existe |
| `RATE_LIMIT` | 429 | Muitas requisições |
| `INTERNAL_ERROR` | 500 | Erro interno |

---

## Logging de Erros

```typescript
// Erro esperado (não loga stack)
logger.warn(`[Service] User not found: ${id}`);

// Erro inesperado (loga stack completo)
logger.error(`[Service] Database error`, { error, stack: error.stack });
```

---

## Resposta de Erro

### Formato Padrão

```json
{
  "error": "Mensagem de erro",
  "code": "ERROR_CODE",
  "details": {} // Opcional, apenas em desenvolvimento
}
```

### Exemplo

```json
{
  "error": "User not found",
  "code": "NOT_FOUND"
}
```

---

**Trate erros com clareza!**
