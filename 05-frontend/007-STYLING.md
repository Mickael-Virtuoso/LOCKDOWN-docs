# Frontend - Styling

## Tailwind CSS

Usamos Tailwind CSS para estilização.

---

## Configuração

### tailwind.config.ts

```typescript
import type { Config } from 'tailwindcss';

export default {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
        },
        secondary: '#1F2937',
        background: {
          light: '#F9FAFB',
          dark: '#111827',
        },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
    },
  },
  plugins: [],
} satisfies Config;
```

---

## Estilos Globais

### globals.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    @apply bg-background-dark text-white antialiased;
  }
}

@layer components {
  .btn {
    @apply px-4 py-2 rounded font-medium transition-colors;
  }

  .btn-primary {
    @apply btn bg-primary-600 hover:bg-primary-700 text-white;
  }

  .btn-secondary {
    @apply btn bg-gray-600 hover:bg-gray-700 text-white;
  }

  .btn-danger {
    @apply btn bg-red-600 hover:bg-red-700 text-white;
  }

  .card {
    @apply bg-gray-800 rounded-lg shadow-lg p-6;
  }

  .input {
    @apply w-full px-4 py-2 bg-gray-700 border border-gray-600 rounded focus:outline-none focus:ring-2 focus:ring-primary-500;
  }
}
```

---

## Componentes de Estilo

### Botões

```tsx
// Variantes de botões
<button className="btn-primary">Primary</button>
<button className="btn-secondary">Secondary</button>
<button className="btn-danger">Danger</button>

// Com ícone
<button className="btn-primary flex items-center gap-2">
  <PlusIcon className="h-5 w-5" />
  Add New
</button>
```

### Cards

```tsx
<div className="card">
  <h3 className="text-lg font-semibold mb-4">Card Title</h3>
  <p className="text-gray-400">Card content goes here.</p>
</div>
```

### Inputs

```tsx
<input
  type="text"
  className="input"
  placeholder="Enter value..."
/>

<select className="input">
  <option>Option 1</option>
  <option>Option 2</option>
</select>
```

---

## Padrões de Layout

### Container

```tsx
<div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
  {/* Content */}
</div>
```

### Grid

```tsx
// Responsive grid
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map((item) => (
    <Card key={item.id} />
  ))}
</div>
```

### Flexbox

```tsx
// Space between
<div className="flex justify-between items-center">
  <h1>Title</h1>
  <button>Action</button>
</div>

// Center
<div className="flex justify-center items-center h-screen">
  <Spinner />
</div>
```

---

## Responsividade

### Breakpoints

| Breakpoint | Min-width | CSS |
|------------|-----------|-----|
| sm | 640px | `sm:` |
| md | 768px | `md:` |
| lg | 1024px | `lg:` |
| xl | 1280px | `xl:` |
| 2xl | 1536px | `2xl:` |

### Exemplo

```tsx
<div className="
  p-4
  sm:p-6
  md:p-8
  lg:p-10
">
  {/* Padding aumenta conforme tela */}
</div>
```

---

## Dark Mode

```tsx
// Tailwind dark mode
<div className="bg-white dark:bg-gray-800 text-black dark:text-white">
  Content
</div>
```

---

## Links Relacionados

- [Componentes](./COMPONENTS.md)
- [Estrutura](./STRUCTURE.md)
- [TailwindCSS Docs](https://tailwindcss.com/docs)
