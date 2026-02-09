# API - Core Endpoints

## Users

### GET /core/users

Listar todos os usuários.

```bash
GET /api/v1/core/users?page=1&limit=50

HTTP/1.1 200 OK

{
  "data": [
    {
      "id": 1,
      "userId": "123456789",
      "username": "john_doe",
      "avatar": "https://cdn.discordapp.com/...",
      "isBot": false,
      "createdAt": "2025-02-05T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 100,
    "pages": 2
  }
}
```

**Query Parameters:**
- `page` (number, default=1)
- `limit` (number, default=50)
- `search` (string) - Buscar por username

---

### GET /core/users/:userId

Obter usuário por Discord ID.

```bash
GET /api/v1/core/users/123456789

HTTP/1.1 200 OK

{
  "data": {
    "id": 1,
    "userId": "123456789",
    "username": "john_doe",
    "createdAt": "2025-02-05T10:00:00Z"
  }
}
```

---

### POST /core/users

Criar novo usuário.

```bash
POST /api/v1/core/users
Content-Type: application/json

{
  "userId": "123456789",
  "username": "john_doe",
  "avatar": "https://cdn.discordapp.com/..."
}

HTTP/1.1 201 Created
```

**Request Body:**
- `userId` (string, required) - Discord ID
- `username` (string, required)
- `avatar` (string, optional)
- `isBot` (boolean, optional)

---

## Guilds

### GET /core/guilds

Listar todas as guilds.

```bash
GET /api/v1/core/guilds?page=1&limit=50

HTTP/1.1 200 OK

{
  "data": [
    {
      "id": 1,
      "guildId": "987654321",
      "name": "My Server",
      "memberCount": 100,
      "createdAt": "2025-02-01T10:00:00Z"
    }
  ]
}
```

---

### GET /core/guilds/:guildId

Obter guild específica.

```bash
GET /api/v1/core/guilds/987654321

HTTP/1.1 200 OK

{
  "data": {
    "guildId": "987654321",
    "name": "My Server",
    "memberCount": 100
  }
}
```

---

### POST /core/guilds

Criar nova guild.

```bash
POST /api/v1/core/guilds
Content-Type: application/json

{
  "guildId": "987654321",
  "name": "My Server",
  "ownerId": "123456789"
}

HTTP/1.1 201 Created
```

---

## Members

### GET /core/guilds/:guildId/members

Listar membros de uma guild.

```bash
GET /api/v1/core/guilds/987654321/members?page=1

HTTP/1.1 200 OK

{
  "data": [
    {
      "id": 1,
      "guildId": "987654321",
      "userId": "123456789",
      "nickname": "John",
      "roles": ["role1", "role2"],
      "joinedAt": "2025-02-01T10:00:00Z"
    }
  ]
}
```

---

### POST /core/guilds/:guildId/members

Adicionar membro.

```bash
POST /api/v1/core/guilds/987654321/members
Content-Type: application/json

{
  "userId": "123456789",
  "nickname": "John",
  "roles": ["role1"]
}

HTTP/1.1 201 Created
```

---

## Links Relacionados

- [Moderation Endpoints](./MODERATION.md)
- [Visão Geral](./OVERVIEW.md)
