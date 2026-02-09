# Backend - Visão Geral

## O que é o Backend LOCKDOWN?

API REST **agnóstica e independente** que expõe funcionalidades de moderação Discord como serviços HTTP.

**Não conhece Discord.js** - apenas fornece dados estruturados via REST.

---

## Stack Tecnológica

| Componente     | Versão  | Propósito            |
| -------------- | ------- | -------------------- |
| Express.js     | ^4.18.2 | Web framework        |
| TypeScript     | latest  | Tipagem              |
| Drizzle ORM    | ^0.30.0 | Database abstraction |
| Better SQLite3 | ^9.2.0  | Dev database         |
| Redis          | 5.3.2   | Cache/Pub-Sub        |
| Pino           | ^8.17.0 | Logging              |

---

## Arquitetura

O backend segue uma arquitetura em camadas:

```
Request → Routes → Controllers → Services → Repositories → Database
                       ↓
                   Middlewares
```

### Camadas

1. **Routes**: Define endpoints e métodos HTTP
2. **Controllers**: Processa requests/responses HTTP
3. **Services**: Contém lógica de negócio
4. **Repositories**: Acesso a dados (CRUD)
5. **Middlewares**: Autenticação, validação, logging

---

## Princípios

- **Separação de responsabilidades**: Cada camada tem uma função específica
- **Agnóstico ao Discord**: O backend não conhece Discord.js
- **Event-driven**: Comunicação via Redis Pub/Sub
- **Type-safe**: TypeScript em todo o código

---

## Links Relacionados

- [Setup](./SETUP.md) - Configuração inicial
- [Estrutura](./STRUCTURE.md) - Organização de pastas
- [Desenvolvimento](./DEVELOPMENT.md) - Criação de endpoints
- [Padrões](./PATTERNS.md) - Convenções de código
