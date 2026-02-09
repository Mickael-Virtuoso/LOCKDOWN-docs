# Backend - Padrões de Código

## Logging

```typescript
// ✅ CORRETO: Com [ClassName]
logger.info(`[${this.constructor.name}] "✅ ban created"`);
logger.error(`[${this.constructor.name}] "❌ error occurred"`);
logger.debug(`[${this.constructor.name}] "🔍 fetching data"`);

// ❌ ERRADO: Sem prefixo
logger.info('something happened');
```

### Níveis de Log

| Nível | Uso |
|-------|-----|
| `debug` | Informações de desenvolvimento |
| `info` | Operações normais |
| `warn` | Situações não críticas |
| `error` | Erros que precisam de atenção |
| `fatal` | Erros críticos que param a aplicação |

---

## Error Handling

```typescript
// ✅ CORRETO: Throw AppError
throw new AppError('Ban not found', 404);

// ❌ ERRADO: Throw generic Error
throw new Error('Ban not found');
```

### Classe AppError

```typescript
export class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number = 500,
    public code?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}
```

### Códigos HTTP Comuns

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 500 | Internal Server Error |

---

## Types

```typescript
// ✅ CORRETO: Tipos explícitos
async findById(id: string): Promise<Ban | null> {
  // ...
}

// ❌ ERRADO: Usar any
async findById(id: any): Promise<any> {
  // ...
}
```

### Convenções de Tipos

- Use `interface` para objetos
- Use `type` para unions e intersections
- Sempre defina tipos de retorno
- Evite `any` - use `unknown` se necessário

---

## Validações

```typescript
// ✅ CORRETO: Validar na service
if (!userId) {
  throw new AppError('User ID is required', 400);
}

// ❌ ERRADO: Nenhuma validação
const ban = await repo.create(data);
```

### Onde Validar

| Camada | O que validar |
|--------|---------------|
| Controller | Formato de entrada (query params, body) |
| Service | Regras de negócio |
| Repository | Integridade de dados |

---

## Nomenclatura

### Arquivos

```
UserController.ts     # PascalCase para classes
user.routes.ts        # kebab-case para routes
api.types.ts          # kebab-case para types
```

### Classes e Interfaces

```typescript
class UserController {}      // PascalCase
interface CreateUserDTO {}   // PascalCase
type UserId = string;        // PascalCase
```

### Variáveis e Funções

```typescript
const userId = '123';        // camelCase
function getUserById() {}    // camelCase
```

---

## Estrutura de Arquivo

```typescript
/**
 * - @file NomeDoArquivo.ts
 *
 * =====================================================================
 * ===================== FILE METADATA ================================
 * =====================================================================
 *
 * @author Seu Nome
 * @since YYYY-MM-DD
 *
 * =====================================================================
 * ====================== GENERAL DESCRIPTION ==========================
 * =====================================================================
 *
 * @description
 * Descrição do arquivo
 *
 */

// Imports
import { ... } from '...';

// Types/Interfaces
interface ... {}

// Class/Functions
export default class ... {}
```

---

## Injeção de Dependência

```typescript
// ✅ CORRETO: Injetar via constructor
class BanService {
  constructor(private banRepository: BanRepository) {}
}

// ❌ ERRADO: Criar dentro do método
class BanService {
  async listBans() {
    const repo = new BanRepository(); // Não!
  }
}
```

---

## Links Relacionados

- [Error Handling](../03-core/ERROR_HANDLING.md)
- [Logging](../03-core/LOGGING.md)
- [Testing](./TESTING.md)
