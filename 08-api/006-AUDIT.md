# API - Audit Endpoints

## Audit Logs

### GET /audit/logs

Listar audit logs.

```bash
GET /api/v1/audit/logs?guildId=987654321&action=ban&page=1

HTTP/1.1 200 OK

{
  "data": [
    {
      "id": 1,
      "guildId": "987654321",
      "action": "ban",
      "performedBy": "111222333",
      "targetId": "123456789",
      "details": {
        "reason": "Spam",
        "expiresAt": "2025-03-05T10:00:00Z"
      },
      "createdAt": "2025-02-05T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 200,
    "pages": 4
  }
}
```

**Query Parameters:**
- `guildId` (string, required)
- `action` (string, optional) - ban, kick, mute, warning, config_change
- `performedBy` (string, optional) - Moderator ID
- `targetId` (string, optional) - Target user ID
- `startDate` (ISO string, optional)
- `endDate` (ISO string, optional)
- `page` (number, default=1)
- `limit` (number, default=50, max=100)

---

### GET /audit/logs/:logId

Obter log específico.

```bash
GET /api/v1/audit/logs/1

HTTP/1.1 200 OK

{
  "data": {
    "id": 1,
    "guildId": "987654321",
    "action": "ban",
    "performedBy": "111222333",
    "targetId": "123456789",
    "details": { ... },
    "createdAt": "2025-02-05T10:00:00Z"
  }
}
```

---

## Action Types

| Action | Descrição |
|--------|-----------|
| `ban` | Usuário banido |
| `unban` | Usuário desbanido |
| `kick` | Usuário expulso |
| `mute` | Usuário mutado |
| `unmute` | Usuário desmutado |
| `warning` | Warning adicionado |
| `warning_clear` | Warnings limpos |
| `whitelist_add` | Adicionado à whitelist |
| `whitelist_remove` | Removido da whitelist |
| `blacklist_add` | Adicionado à blacklist |
| `blacklist_remove` | Removido da blacklist |
| `config_change` | Configuração alterada |
| `auto_role_add` | Auto-role adicionada |
| `auto_role_remove` | Auto-role removida |

---

## Filtros Avançados

### Por Data

```bash
GET /api/v1/audit/logs?guildId=987654321&startDate=2025-02-01&endDate=2025-02-28
```

### Por Moderador

```bash
GET /api/v1/audit/logs?guildId=987654321&performedBy=111222333
```

### Múltiplas Ações

```bash
GET /api/v1/audit/logs?guildId=987654321&action=ban,kick,mute
```

---

## Links Relacionados

- [Moderation Endpoints](./MODERATION.md)
- [Config Endpoints](./CONFIG.md)
