# Frontend - Visão Geral

## O que é o Frontend?

Dashboard React para gerenciar LOCKDOWN:

```
Users
├─ View all bans
├─ Create new ban
├─ Remove ban
└─ Filter by guild/user

Guilds
├─ View guild stats
├─ Configure settings
└─ View audit logs

Dashboard
├─ Real-time stats
├─ Activity feed
└─ Quick actions
```

---

## Stack Tecnológica

| Tecnologia | Propósito |
|------------|-----------|
| React 18 | UI library |
| Vite 5 | Build tool |
| TailwindCSS | Styling |
| TypeScript | Type safety |
| Axios | HTTP client |
| Zustand | State management |
| React Router | Navigation |

---

## Funcionalidades

### Bans Management
- Listar banimentos por guild
- Criar novos banimentos
- Remover banimentos
- Filtrar por status (active/expired)

### Guild Management
- Visualizar estatísticas
- Configurar settings
- Auto-roles
- Whitelist/Blacklist

### Dashboard
- Estatísticas em tempo real
- Feed de atividades
- Ações rápidas

### Audit Logs
- Histórico de ações
- Filtros por tipo/data
- Export de relatórios

---

## Links Relacionados

- [Setup](./SETUP.md)
- [Estrutura](./STRUCTURE.md)
- [Componentes](./COMPONENTS.md)
- [API Integration](./API.md)
