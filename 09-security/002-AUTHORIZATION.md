# Autorização

Sistema de permissões e roles do LOCKDOWN.

---

## Role-Based Access Control (RBAC)

### Níveis

| Role | Permissões |
|------|------------|
| Owner | Acesso total |
| Admin | Gerenciar configurações |
| Moderator | Ações de moderação |
| Member | Acesso básico |

---

## Permissões Discord

```typescript
const MODERATION_PERMISSIONS = [
  'BAN_MEMBERS',
  'KICK_MEMBERS',
  'MANAGE_MESSAGES',
  'MUTE_MEMBERS',
];

const ADMIN_PERMISSIONS = [
  'MANAGE_GUILD',
  'MANAGE_ROLES',
  'MANAGE_CHANNELS',
];
```

---

## Guard de Autorização

```typescript
export async function authorizationMiddleware(req, res, next) {
  const { botId } = req;
  const { guildId } = req.params;

  // Verificar se bot tem acesso à guild
  const hasAccess = await checkBotGuildAccess(botId, guildId);

  if (!hasAccess) {
    return res.status(403).json({
      error: 'Forbidden',
      code: 'NO_GUILD_ACCESS',
    });
  }

  next();
}
```

---

## Verificação de Permissão

```typescript
export function checkPermission(
  member: GuildMember,
  permission: PermissionResolvable
): boolean {
  return member.permissions.has(permission);
}

// Uso
if (!checkPermission(member, 'BAN_MEMBERS')) {
  throw new Error('Missing BAN_MEMBERS permission');
}
```

---

## Hierarquia de Roles

```typescript
export function canModerate(
  moderator: GuildMember,
  target: GuildMember
): boolean {
  // Não pode moderar a si mesmo
  if (moderator.id === target.id) return false;

  // Não pode moderar o dono
  if (target.id === target.guild.ownerId) return false;

  // Verificar hierarquia de roles
  return moderator.roles.highest.position > target.roles.highest.position;
}
```

---

## Whitelist e Blacklist

### Whitelist

Usuários na whitelist ignoram certas restrições.

```typescript
async function isWhitelisted(guildId: string, userId: string): Promise<boolean> {
  const entry = await db.select().from(whitelistTable)
    .where(and(
      eq(whitelistTable.guildId, guildId),
      eq(whitelistTable.userId, userId)
    ));

  return entry.length > 0;
}
```

### Blacklist

Usuários na blacklist são automaticamente bloqueados.

```typescript
async function isBlacklisted(guildId: string, userId: string): Promise<boolean> {
  const entry = await db.select().from(blacklistTable)
    .where(and(
      eq(blacklistTable.guildId, guildId),
      eq(blacklistTable.userId, userId)
    ));

  return entry.length > 0;
}
```

---

## Links Relacionados

- [Autenticação](./AUTHENTICATION.md)
- [Boas Práticas](./BEST-PRACTICES.md)
