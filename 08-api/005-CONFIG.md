# API - Config Endpoints

## Guild Configuration

### GET /config/guilds/:guildId

Obter configuração da guild.

```bash
GET /api/v1/config/guilds/987654321

HTTP/1.1 200 OK

{
  "data": {
    "id": 1,
    "guildId": "987654321",
    "logChannelId": "111222333",
    "moderationRoleId": "444555666",
    "muteRoleId": "777888999",
    "autoModEnabled": true,
    "createdAt": "2025-02-01T10:00:00Z"
  }
}
```

---

### PATCH /config/guilds/:guildId

Atualizar configuração.

```bash
PATCH /api/v1/config/guilds/987654321
Content-Type: application/json

{
  "logChannelId": "111222333",
  "autoModEnabled": true
}

HTTP/1.1 200 OK

{
  "data": { ... }
}
```

**Campos Atualizáveis:**
- `logChannelId` (string)
- `moderationRoleId` (string)
- `muteRoleId` (string)
- `autoModEnabled` (boolean)

---

## Auto Roles

### GET /config/guilds/:guildId/auto-roles

Listar roles automáticas.

```bash
GET /api/v1/config/guilds/987654321/auto-roles

HTTP/1.1 200 OK

{
  "data": [
    {
      "id": 1,
      "guildId": "987654321",
      "roleId": "role_123",
      "createdAt": "2025-02-01T10:00:00Z"
    }
  ]
}
```

---

### POST /config/guilds/:guildId/auto-roles

Adicionar role automática.

```bash
POST /api/v1/config/guilds/987654321/auto-roles
Content-Type: application/json

{
  "roleId": "role_123"
}

HTTP/1.1 201 Created
```

---

### DELETE /config/guilds/:guildId/auto-roles/:roleId

Remover role automática.

```bash
DELETE /api/v1/config/guilds/987654321/auto-roles/role_123

HTTP/1.1 200 OK
```

---

## Restrict Roles

### GET /config/guilds/:guildId/restrict-roles

Listar roles restritas.

```bash
GET /api/v1/config/guilds/987654321/restrict-roles

HTTP/1.1 200 OK
```

### POST /config/guilds/:guildId/restrict-roles

Adicionar role restrita.

```bash
POST /api/v1/config/guilds/987654321/restrict-roles
Content-Type: application/json

{
  "roleId": "role_456",
  "restrictions": ["send_messages", "add_reactions"]
}

HTTP/1.1 201 Created
```

---

## Links Relacionados

- [Moderation Endpoints](./MODERATION.md)
- [Audit Endpoints](./AUDIT.md)
