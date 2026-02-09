# Frontend - Componentes

## Padrões de Componentes

### Estrutura de Arquivo

```typescript
/**
 * - @file NomeDoComponente.tsx
 *
 * @author Seu Nome
 * @since YYYY-MM-DD
 *
 * @description
 * Descrição do componente
 */

import React from 'react';

interface Props {
  // Props tipadas
}

export default function NomeDoComponente({ ...props }: Props) {
  return (
    // JSX
  );
}
```

---

## Componentes Comuns

### Header

```typescript
import React from 'react';
import { Link } from 'react-router-dom';
import { useAuth } from '@hooks/useAuth';

export default function Header() {
  const { user, logout } = useAuth();

  return (
    <header className="bg-slate-900 text-white shadow-lg">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between items-center h-16">
          {/* Logo */}
          <Link to="/" className="text-2xl font-bold">
            🔒 LOCKDOWN
          </Link>

          {/* Navigation */}
          <nav className="hidden md:flex space-x-8">
            <Link to="/dashboard" className="hover:text-blue-400 transition">
              Dashboard
            </Link>
            <Link to="/bans" className="hover:text-blue-400 transition">
              Bans
            </Link>
            <Link to="/guilds" className="hover:text-blue-400 transition">
              Guilds
            </Link>
          </nav>

          {/* User Menu */}
          <div className="flex items-center space-x-4">
            {user ? (
              <>
                <span className="text-sm">{user.username}</span>
                <button
                  onClick={logout}
                  className="bg-red-600 px-4 py-2 rounded hover:bg-red-700"
                >
                  Logout
                </button>
              </>
            ) : (
              <Link
                to="/login"
                className="bg-blue-600 px-4 py-2 rounded hover:bg-blue-700"
              >
                Login
              </Link>
            )}
          </div>
        </div>
      </div>
    </header>
  );
}
```

---

## Componentes de Feature

### BansList

```typescript
import React, { useEffect, useState } from 'react';
import { useBans } from '@hooks/useBans';
import BanCard from './BanCard';
import CreateBanModal from './CreateBanModal';

interface Props {
  guildId: string;
}

export default function BansList({ guildId }: Props) {
  const { bans, isLoading, error, fetchBans, removeBan } = useBans(guildId);
  const [showModal, setShowModal] = useState(false);
  const [filter, setFilter] = useState<'all' | 'active' | 'expired'>('active');

  useEffect(() => {
    fetchBans();
  }, [guildId]);

  const filteredBans = bans.filter((ban) => {
    if (filter === 'all') return true;
    if (filter === 'active') return ban.isActive;
    if (filter === 'expired') return !ban.isActive;
    return true;
  });

  if (isLoading) {
    return (
      <div className="flex justify-center items-center h-64">
        <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600" />
      </div>
    );
  }

  if (error) {
    return (
      <div className="bg-red-50 text-red-700 p-4 rounded">
        Error: {error}
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">Bans ({filteredBans.length})</h2>
        <button
          onClick={() => setShowModal(true)}
          className="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
        >
          + New Ban
        </button>
      </div>

      {/* Filters */}
      <div className="flex space-x-2">
        {(['all', 'active', 'expired'] as const).map((f) => (
          <button
            key={f}
            onClick={() => setFilter(f)}
            className={`px-4 py-2 rounded ${
              filter === f
                ? 'bg-blue-600 text-white'
                : 'bg-gray-200 hover:bg-gray-300'
            }`}
          >
            {f.charAt(0).toUpperCase() + f.slice(1)}
          </button>
        ))}
      </div>

      {/* Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {filteredBans.map((ban) => (
          <BanCard key={ban.id} ban={ban} onRemove={() => removeBan(ban.id)} />
        ))}
      </div>

      {/* Modal */}
      {showModal && (
        <CreateBanModal
          guildId={guildId}
          onClose={() => setShowModal(false)}
          onSuccess={() => {
            setShowModal(false);
            fetchBans();
          }}
        />
      )}
    </div>
  );
}
```

---

## Boas Práticas

### 1. Props tipadas

```typescript
// ✅ CORRETO
interface Props {
  guildId: string;
  onSuccess?: () => void;
}

// ❌ ERRADO
function Component(props: any) {}
```

### 2. Componentes pequenos

```typescript
// ✅ CORRETO: Componentes focados
function BanCard({ ban, onRemove }) {}
function BanFilters({ filter, onChange }) {}
function BansList({ guildId }) {}

// ❌ ERRADO: Componente gigante
function BansPage() {
  // 500 linhas de código
}
```

### 3. Custom hooks para lógica

```typescript
// ✅ CORRETO: Extrair lógica
const { bans, isLoading, fetchBans } = useBans(guildId);

// ❌ ERRADO: Lógica no componente
const [bans, setBans] = useState([]);
const [isLoading, setIsLoading] = useState(false);
// ... muito mais estado
```

---

## Links Relacionados

- [Estrutura](./STRUCTURE.md)
- [State Management](./STATE.md)
- [Styling](./STYLING.md)
