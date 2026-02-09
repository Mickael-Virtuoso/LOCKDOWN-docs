# Backend - Estrutura de Pastas

## Organização Completa

```
packages/backend/src/
├─ api/
│  ├─ controllers/          # HTTP request handlers
│  │  ├─ core/
│  │  │  ├─ UserController.ts
│  │  │  ├─ MemberController.ts
│  │  │  └─ GuildController.ts
│  │  ├─ moderation/
│  │  │  ├─ BanController.ts
│  │  │  ├─ KickController.ts
│  │  │  ├─ MuteController.ts
│  │  │  └─ WarningController.ts
│  │  ├─ config/
│  │  │  ├─ AutoRoleController.ts
│  │  │  └─ GuildConfigController.ts
│  │  └─ audit/
│  │     └─ AuditLogController.ts
│  │
│  ├─ routes/               # Endpoint definitions
│  │  └─ v1/
│  │     ├─ core/
│  │     ├─ moderation/
│  │     ├─ config/
│  │     └─ index.routes.ts (agregador)
│  │
│  ├─ middlewares/          # Express middlewares
│  │  ├─ authentication.middleware.ts
│  │  ├─ authorization.middleware.ts
│  │  ├─ validation.middleware.ts
│  │  ├─ errorHandler.middleware.ts
│  │  └─ logger.middleware.ts
│  │
│  └─ app.ts               # Express instance
│
├─ config/                  # Configuration
│  ├─ environment.ts        # Env var validation
│  ├─ database.ts           # Drizzle setup
│  ├─ redis.ts             # Redis connection
│  └─ logger.ts            # Pino setup
│
├─ core/                    # Bootstrap
│  ├─ bootstrap.ts         # App initialization
│  ├─ shutdown.ts          # Graceful shutdown
│  └─ index.ts             # Main entry
│
├─ database/               # Data layer
│  ├─ schemas/             # Table definitions
│  │  ├─ core.ts           # Users, Members, Guilds
│  │  ├─ moderation.ts     # Bans, Kicks, Mutes
│  │  ├─ config.ts         # Auto-roles, etc
│  │  └─ audit.ts          # Audit logs
│  │
│  ├─ repositories/        # Data access (CRUD)
│  │  ├─ core/
│  │  │  ├─ UserRepository.ts
│  │  │  └─ GuildRepository.ts
│  │  └─ moderation/
│  │     ├─ BanRepository.ts
│  │     └─ KickRepository.ts
│  │
│  ├─ services/            # Business logic
│  │  ├─ core/
│  │  │  ├─ UserService.ts
│  │  │  └─ GuildService.ts
│  │  └─ moderation/
│  │     ├─ BanService.ts
│  │     └─ KickService.ts
│  │
│  ├─ migrations/          # Database migrations
│  │  ├─ 0001_initial_schema.sql
│  │  ├─ 0002_moderation_tables.sql
│  │  └─ ...
│  │
│  └─ client.ts            # Drizzle ORM instance
│
├─ events/                  # Pub/Sub handlers
│  ├─ publishers/          # Event publishers
│  │  ├─ BanPublisher.ts
│  │  └─ KickPublisher.ts
│  ├─ subscribers/         # Event subscribers
│  └─ EventManager.ts      # Central manager
│
├─ services/               # Global services
│  ├─ CacheService.ts      # Redis cache wrapper
│  ├─ HealthCheckService.ts
│  ├─ LoggerService.ts
│  └─ NetworkManager.ts    # Inter-service communication
│
├─ types/                  # TypeScript types
│  ├─ api.types.ts         # DTOs, responses
│  ├─ database.types.ts    # Model types
│  ├─ events.types.ts      # Event schemas
│  └─ express.d.ts         # Express extensions
│
├─ utils/                  # Utilities
│  ├─ validators.ts        # Validation functions
│  ├─ formatters.ts        # Data formatting
│  ├─ helpers.ts           # Helper functions
│  ├─ errors.ts            # Custom error classes
│  └─ constants.ts         # Constants
│
└─ server.ts              # Entry point (main)
```

---

## Descrição das Pastas

### `/api`
Camada HTTP - controllers, routes e middlewares.

### `/config`
Configurações centralizadas - ambiente, banco, redis, logger.

### `/core`
Bootstrap da aplicação - inicialização e shutdown.

### `/database`
Camada de dados - schemas, repositories, services, migrations.

### `/events`
Sistema de eventos Redis Pub/Sub.

### `/services`
Serviços globais compartilhados.

### `/types`
Definições TypeScript.

### `/utils`
Funções utilitárias.

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Desenvolvimento](./DEVELOPMENT.md)
- [Arquitetura](../01-architecture/ARCHITECTURE.md)
