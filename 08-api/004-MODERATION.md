# API - Moderation Endpoints

## Bans

### GET /moderation/bans

Listar bans.

```bash
GET /api/v1/moderation/bans?guildId=987654321&isActive=true

HTTP/1.1 200 OK

{
  "data": [
    {
      "id": 1,
      "guildId": "987654321",
      "userId": "123456789",
      "reason": "Spam",
      "moderatorId": "111222333",
      "expiresAt": "2025-03-05T10:00:00Z",
      "isActive": true,
      "createdAt": "2025-02-05T10:00:00Z"
    }
  ]
}
```

**Query Parameters:**
- `guildId` (string, required)
- `isActive` (boolean, optional)
- `userId` (string, optional)
- `page`, `limit`

---

### POST /moderation/bans

Criar ban.

```bash
POST /api/v1/moderation/bans
Content-Type: application/json

{
  "guildId": "987654321",
  "userId": "123456789",
  "reason": "Spam",
  "moderatorId": "111222333",
  "expiresAt": "2025-03-05T10:00:00Z"
}

HTTP/1.1 201 Created
```

**Request Body:**
- `guildId` (string, required)
- `userId` (string, required)
- `reason` (string, optional)
- `moderatorId` (string, required)
- `expiresAt` (ISO string, optional) - null = permanente

---

### DELETE /moderation/bans/:banId

Remover ban.

```bash
DELETE /api/v1/moderation/bans/1

HTTP/1.1 200 OK

{
  "data": { "success": true }
}
```

---

## Kicks

### POST /moderation/kicks

Criar kick.

```bash
POST /api/v1/moderation/kicks
Content-Type: application/json

{
  "guildId": "987654321",
  "userId": "123456789",
  "reason": "Disruptive behavior",
  "moderatorId": "111222333"
}

HTTP/1.1 201 Created
```

### GET /moderation/kicks

Listar kicks.

```bash
GET /api/v1/moderation/kicks?guildId=987654321

HTTP/1.1 200 OK
```

---

## Mutes

### POST /moderation/mutes

Criar mute.

```bash
POST /api/v1/moderation/mutes
Content-Type: application/json

{
  "guildId": "987654321",
  "userId": "123456789",
  "reason": "Excessive messages",
  "moderatorId": "111222333",
  "duration": 60
}

HTTP/1.1 201 Created
```

**Campos:**
- `duration` (number) - Minutos

### DELETE /moderation/mutes/:muteId

Remover mute.

```bash
DELETE /api/v1/moderation/mutes/1

HTTP/1.1 200 OK
```

---

## Warnings

### POST /moderation/warnings

Adicionar warning.

```bash
POST /api/v1/moderation/warnings
Content-Type: application/json

{
  "guildId": "987654321",
  "userId": "123456789",
  "reason": "First warning",
  "moderatorId": "111222333"
}

HTTP/1.1 201 Created

{
  "data": {
    "id": 1,
    "userId": "123456789",
    "count": 1
  }
}
```

### GET /moderation/warnings/:guildId/:userId

Obter warnings do usuário.

```bash
GET /api/v1/moderation/warnings/987654321/123456789

HTTP/1.1 200 OK

{
  "data": {
    "userId": "123456789",
    "count": 2,
    "warnings": [...]
  }
}
```

---

## Whitelist & Blacklist

### POST /moderation/whitelist

```bash
POST /api/v1/moderation/whitelist
Content-Type: application/json

{
  "guildId": "987654321",
  "userId": "123456789",
  "reason": "Trusted member",
  "addedBy": "111222333"
}

HTTP/1.1 201 Created
```

### POST /moderation/blacklist

```bash
POST /api/v1/moderation/blacklist
Content-Type: application/json

{
  "guildId": "987654321",
  "userId": "123456789",
  "reason": "Banned from other servers",
  "addedBy": "111222333"
}

HTTP/1.1 201 Created
```

---

## Links Relacionados

- [Core Endpoints](./CORE.md)
- [Config Endpoints](./CONFIG.md)
- [Errors](./ERRORS.md)
