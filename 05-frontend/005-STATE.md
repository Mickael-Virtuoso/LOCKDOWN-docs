# Frontend - State Management

## Zustand

Usamos Zustand para gerenciamento de estado global.

---

## Estrutura de Stores

```
store/
├─ authStore.ts      # Autenticação
├─ bansStore.ts      # Banimentos
├─ guildsStore.ts    # Guilds
└─ uiStore.ts        # UI state
```

---

## Bans Store

```typescript
// store/bansStore.ts
import { create } from 'zustand';
import { bansApi } from '@api/bans.api';
import type { SelectBan } from '@types/api.types';

interface BansState {
  bans: SelectBan[];
  isLoading: boolean;
  error: string | null;
  fetchBans: (guildId: string) => Promise<void>;
  createBan: (data: any) => Promise<void>;
  removeBan: (banId: string) => Promise<void>;
}

export const useBansStore = create<BansState>((set) => ({
  bans: [],
  isLoading: false,
  error: null,

  fetchBans: async (guildId: string) => {
    set({ isLoading: true, error: null });
    try {
      const response = await bansApi.listByGuild(guildId);
      set({ bans: response.data, isLoading: false });
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false,
      });
    }
  },

  createBan: async (data) => {
    set({ isLoading: true, error: null });
    try {
      const newBan = await bansApi.create(data);
      set((state) => ({
        bans: [newBan, ...state.bans],
        isLoading: false,
      }));
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false,
      });
      throw error;
    }
  },

  removeBan: async (banId: string) => {
    set({ isLoading: true, error: null });
    try {
      await bansApi.remove(banId);
      set((state) => ({
        bans: state.bans.filter((b) => b.id.toString() !== banId),
        isLoading: false,
      }));
    } catch (error) {
      set({
        error: error instanceof Error ? error.message : 'Unknown error',
        isLoading: false,
      });
      throw error;
    }
  },
}));
```

---

## Auth Store

```typescript
// store/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  username: string;
  avatar: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (user: User, token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,

      login: (user, token) => {
        localStorage.setItem('auth_token', token);
        set({ user, token, isAuthenticated: true });
      },

      logout: () => {
        localStorage.removeItem('auth_token');
        set({ user: null, token: null, isAuthenticated: false });
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ user: state.user, token: state.token }),
    }
  )
);
```

---

## UI Store

```typescript
// store/uiStore.ts
import { create } from 'zustand';

interface UIState {
  sidebarOpen: boolean;
  theme: 'light' | 'dark';
  toggleSidebar: () => void;
  setTheme: (theme: 'light' | 'dark') => void;
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  theme: 'dark',

  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),

  setTheme: (theme) => set({ theme }),
}));
```

---

## Padrões

### 1. Selectors

```typescript
// Selecionar apenas o que precisa
const bans = useBansStore((state) => state.bans);
const isLoading = useBansStore((state) => state.isLoading);

// Evitar re-renders desnecessários
const { bans, isLoading } = useBansStore(
  useShallow((state) => ({ bans: state.bans, isLoading: state.isLoading }))
);
```

### 2. Actions separadas

```typescript
// Chamar actions diretamente
const fetchBans = useBansStore((state) => state.fetchBans);

// Usar em useEffect
useEffect(() => {
  fetchBans(guildId);
}, [guildId, fetchBans]);
```

### 3. Estado derivado

```typescript
// Computar dentro do componente
const activeBans = bans.filter((ban) => ban.isActive);
const totalBans = bans.length;
```

---

## Links Relacionados

- [API Integration](./API.md)
- [Componentes](./COMPONENTS.md)
- [Estrutura](./STRUCTURE.md)
