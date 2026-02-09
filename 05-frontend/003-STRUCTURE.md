# Frontend - Estrutura de Pastas

## Organização Completa

```
packages/frontend/src/
├─ components/
│  ├─ common/
│  │  ├─ Header.tsx
│  │  ├─ Sidebar.tsx
│  │  ├─ Footer.tsx
│  │  └─ LoadingSpinner.tsx
│  │
│  ├─ bans/
│  │  ├─ BansList.tsx
│  │  ├─ BanCard.tsx
│  │  ├─ CreateBanModal.tsx
│  │  └─ BanFilters.tsx
│  │
│  ├─ guilds/
│  │  ├─ GuildsList.tsx
│  │  ├─ GuildStats.tsx
│  │  └─ GuildConfig.tsx
│  │
│  └─ dashboard/
│     ├─ DashboardHome.tsx
│     ├─ StatsCard.tsx
│     └─ ActivityFeed.tsx
│
├─ pages/
│  ├─ Home.tsx
│  ├─ Bans.tsx
│  ├─ Guilds.tsx
│  ├─ Settings.tsx
│  ├─ Audit.tsx
│  └─ NotFound.tsx
│
├─ api/
│  ├─ client.ts
│  ├─ bans.api.ts
│  ├─ guilds.api.ts
│  ├─ users.api.ts
│  └─ types.ts
│
├─ store/
│  ├─ authStore.ts
│  ├─ bansStore.ts
│  ├─ guildsStore.ts
│  └─ uiStore.ts
│
├─ hooks/
│  ├─ useAuth.ts
│  ├─ useBans.ts
│  ├─ useGuilds.ts
│  └─ useApi.ts
│
├─ styles/
│  ├─ globals.css
│  ├─ tailwind.css
│  └─ variables.css
│
├─ types/
│  ├─ api.types.ts
│  ├─ components.types.ts
│  └─ store.types.ts
│
├─ utils/
│  ├─ api.utils.ts
│  ├─ format.utils.ts
│  ├─ date.utils.ts
│  └─ validation.utils.ts
│
├─ App.tsx
├─ main.tsx
└─ vite-env.d.ts
```

---

## Descrição das Pastas

### `/components`
Componentes React organizados por feature.

### `/pages`
Páginas da aplicação (rotas).

### `/api`
Cliente HTTP e funções de API.

### `/store`
Stores Zustand para estado global.

### `/hooks`
Custom hooks reutilizáveis.

### `/styles`
Arquivos CSS e Tailwind.

### `/types`
Definições TypeScript.

### `/utils`
Funções utilitárias.

---

## Links Relacionados

- [Visão Geral](./OVERVIEW.md)
- [Componentes](./COMPONENTS.md)
- [State Management](./STATE.md)
