# API - Exemplos

## cURL

### Criar Ban

```bash
curl -X POST http://localhost:3000/api/v1/moderation/bans \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "guildId": "987654321",
    "userId": "123456789",
    "reason": "Spam",
    "moderatorId": "111222333"
  }'
```

### Listar Bans

```bash
curl -X GET "http://localhost:3000/api/v1/moderation/bans?guildId=987654321&isActive=true" \
  -H "Authorization: Bearer $TOKEN"
```

### Remover Ban

```bash
curl -X DELETE http://localhost:3000/api/v1/moderation/bans/1 \
  -H "Authorization: Bearer $TOKEN"
```

---

## JavaScript/Node.js

### Setup

```typescript
import axios from 'axios';

const API_BASE = 'http://localhost:3000/api/v1';
const TOKEN = process.env.API_TOKEN;

const api = axios.create({
  baseURL: API_BASE,
  headers: {
    Authorization: `Bearer ${TOKEN}`,
    'Content-Type': 'application/json',
  },
});
```

### CRUD Bans

```typescript
// Criar ban
const createBan = async (data: {
  guildId: string;
  userId: string;
  reason: string;
  moderatorId: string;
}) => {
  const response = await api.post('/moderation/bans', data);
  return response.data.data;
};

// Listar bans
const listBans = async (guildId: string) => {
  const response = await api.get('/moderation/bans', {
    params: { guildId, isActive: true },
  });
  return response.data.data;
};

// Remover ban
const removeBan = async (banId: string) => {
  await api.delete(`/moderation/bans/${banId}`);
};

// Uso
const ban = await createBan({
  guildId: '987654321',
  userId: '123456789',
  reason: 'Spam',
  moderatorId: '111222333',
});

console.log('Ban created:', ban.id);
```

---

## Python

### Setup

```python
import requests
import os

API_BASE = 'http://localhost:3000/api/v1'
TOKEN = os.environ['API_TOKEN']

headers = {
    'Authorization': f'Bearer {TOKEN}',
    'Content-Type': 'application/json'
}
```

### CRUD Bans

```python
# Criar ban
def create_ban(guild_id, user_id, reason, moderator_id):
    response = requests.post(
        f'{API_BASE}/moderation/bans',
        headers=headers,
        json={
            'guildId': guild_id,
            'userId': user_id,
            'reason': reason,
            'moderatorId': moderator_id
        }
    )
    return response.json()['data']

# Listar bans
def list_bans(guild_id):
    response = requests.get(
        f'{API_BASE}/moderation/bans',
        headers=headers,
        params={'guildId': guild_id, 'isActive': True}
    )
    return response.json()['data']

# Remover ban
def remove_ban(ban_id):
    requests.delete(
        f'{API_BASE}/moderation/bans/{ban_id}',
        headers=headers
    )

# Uso
ban = create_ban('987654321', '123456789', 'Spam', '111222333')
print(f"Ban created: {ban['id']}")
```

---

## Discord.js Bot

```typescript
import { Client, Events } from 'discord.js';
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.API_URL,
  headers: { Authorization: `Bearer ${process.env.API_TOKEN}` },
});

client.on(Events.GuildBanAdd, async (ban) => {
  await api.post('/moderation/bans', {
    guildId: ban.guild.id,
    userId: ban.user.id,
    reason: ban.reason || 'No reason provided',
    moderatorId: client.user!.id,
  });
});

client.on(Events.GuildBanRemove, async (ban) => {
  const response = await api.get('/moderation/bans', {
    params: { guildId: ban.guild.id, userId: ban.user.id },
  });

  const activeBan = response.data.data[0];
  if (activeBan) {
    await api.delete(`/moderation/bans/${activeBan.id}`);
  }
});
```

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Autenticação](./AUTH.md)
- [Moderation Endpoints](./MODERATION.md)
