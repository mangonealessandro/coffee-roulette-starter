# Coffee Roulette – Web

## Overview

Frontend web application per gestire match casuali tra colleghi per pause caffè.
Permette login, visualizzazione match settimanali/on-demand, e richiesta caffè in tempo reale.

**Problema risolto:** Interfaccia user-friendly per favorire connessioni spontanee tra colleghi remoti/ibridi.

## Stack

| Tech         | Version | Note                                                                 |
| ------------ | ------- | -------------------------------------------------------------------- |
| Next.js      | 14+     | App Router (non Pages Router), scelto per SSR e file-based routing   |
| React        | 18+     | Server Components by default, Client Components solo dove necessario |
| Tailwind CSS | 3+      | Utility-first CSS, configurato per design system aziendale           |
| shadcn/ui    | Latest  | Componenti headless + Radix UI, personalizzabili al 100%             |
| TypeScript   | 5+      | Strict mode abilitato per type safety massima                        |

## Getting Started

### Prerequisites

- Node.js 18+
- npm 9+
- Backend API running su `http://localhost:3001` (vedi `api/claude.md`)

### Setup

```bash
# Installa dipendenze
npm install

# Copia env vars
cp .env.example .env.local
# Edita NEXT_PUBLIC_API_URL se API non è su localhost:3001

# Installa componenti shadcn base
npx shadcn-ui@latest init

# Avvia dev server
npm run dev  # Porta 3000, hot reload abilitato
```

### Running

```bash
npm run dev          # Dev server con hot reload (porta 3000)
npm run build        # Build ottimizzato per produzione
npm run start        # Serve build di produzione
npm run lint         # ESLint + type checking
npm run lint:fix     # Auto-fix problemi linting
npx shadcn-ui add [component]  # Aggiungi componenti shadcn
```

**Env vars necessarie:**

- `NEXT_PUBLIC_API_URL` - Base URL del backend API (default: `http://localhost:3001/api`)
- `NEXT_PUBLIC_APP_NAME` - Nome app per meta tags (default: `Coffee Roulette`)

## Architecture

```
web/
├── src/
│   ├── app/                    # Next.js 14 App Router
│   │   ├── (auth)/             # Route group: autenticazione (no layout dashboard)
│   │   │   ├── login/
│   │   │   │   └── page.tsx    # Form login
│   │   │   └── register/
│   │   │       └── page.tsx    # Form registrazione
│   │   ├── (dashboard)/        # Route group: area protetta (con sidebar)
│   │   │   ├── layout.tsx      # Layout con sidebar + auth check
│   │   │   ├── matches/
│   │   │   │   └── page.tsx    # Lista match
│   │   │   └── profile/
│   │   │       └── page.tsx    # Profilo utente
│   │   ├── layout.tsx          # Root layout (providers, fonts)
│   │   ├── page.tsx            # Landing page / redirect logic
│   │   └── globals.css         # Tailwind imports + custom CSS vars
│   ├── components/
│   │   ├── ui/                 # shadcn components (auto-generated)
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   └── ...
│   │   └── features/           # Business logic components
│   │       ├── MatchCard.tsx
│   │       ├── CoffeeButton.tsx
│   │       ├── UserAvatar.tsx
│   │       └── AvailabilityToggle.tsx
│   ├── lib/
│   │   ├── api.ts              # API client con fetch wrapper + error handling
│   │   ├── auth.ts             # JWT token management + refresh logic
│   │   └── utils.ts            # Helper functions (cn, formatDate, etc)
│   ├── hooks/
│   │   ├── useAuth.ts          # Hook per login/logout/user state
│   │   ├── useMatches.ts       # Hook per fetch/update matches
│   │   └── useToast.ts         # shadcn toast wrapper
│   └── types/
│       ├── api.ts              # Response types dal backend
│       └── index.ts            # Shared types (User, Match, etc)
├── public/
│   ├── avatars/                # Avatar placeholder images
│   └── favicon.ico
└── tailwind.config.ts          # Tailwind + shadcn theme config
```

**Pattern architetturale:** Feature-based structure con separazione UI/Logic

**Data flow:**

1. Page (Server Component) → pre-fetch data se possibile
2. Client Component → useHook → API client → Backend
3. API client → gestisce errors + refresh token → ritorna data
4. Hook → aggiorna state → UI re-render

## Pages & Routes

| Route       | Type               | Descrizione                        | Auth Required |
| ----------- | ------------------ | ---------------------------------- | ------------- |
| `/`         | Server Component   | Landing page con redirect logic    | No            |
| `/login`    | Client Component   | Form login con validazione         | No            |
| `/register` | Client Component   | Form registrazione                 | No            |
| `/matches`  | Server Component\* | Lista match (pending + completati) | Sì            |
| `/profile`  | Client Component   | Profilo + toggle disponibilità     | Sì            |

\*Server Component con Client Components annidati per interattività

**Auth flow:**

- Routes in `(auth)` → accessibili solo se non loggato (redirect a `/matches` se loggato)
- Routes in `(dashboard)` → protected, redirect a `/login` se non autenticato
- Auth check in `(dashboard)/layout.tsx` via middleware o server-side check

## Key Components

### Features Components

| Component            | Path                                     | Descrizione                                               |
| -------------------- | ---------------------------------------- | --------------------------------------------------------- |
| `CoffeeButton`       | `components/features/CoffeeButton`       | CTA "Voglio un caffè!" → POST `/matches/on-demand`        |
| `MatchCard`          | `components/features/MatchCard`          | Card match con avatar, nome, data, actions                |
| `AvailabilityToggle` | `components/features/AvailabilityToggle` | Switch per disponibilità → PATCH `/users/me/availability` |
| `UserAvatar`         | `components/features/UserAvatar`         | Avatar con fallback iniziali nome                         |
| `MatchList`          | `components/features/MatchList`          | Container lista match con filter pending/completed        |

### shadcn UI Components (già installati)

- `Button` - Variants: default, outline, ghost, destructive
- `Card` - Con CardHeader, CardContent, CardFooter
- `Avatar` - Con AvatarImage, AvatarFallback
- `Badge` - Per status match (pending, completed, skipped)
- `Input`, `Label` - Form controls
- `Switch` - Toggle disponibilità
- `Toast` - Notifiche real-time (nuovo match, errori)
- `Skeleton` - Loading states

## API Integration

### API Client Setup

```typescript
// lib/api.ts
const API_BASE = process.env.NEXT_PUBLIC_API_URL || "http://localhost:3001/api";

export class ApiError extends Error {
  constructor(
    public status: number,
    message?: string,
  ) {
    super(message || `HTTP Error ${status}`);
  }
}

export async function apiClient<T>(
  endpoint: string,
  options?: RequestInit,
): Promise<T> {
  const token = getAccessToken(); // Da lib/auth.ts

  const res = await fetch(`${API_BASE}${endpoint}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token && { Authorization: `Bearer ${token}` }),
      ...options?.headers,
    },
  });

  // Handle 401 → refresh token
  if (res.status === 401) {
    const refreshed = await refreshAccessToken();
    if (refreshed) {
      // Retry request con nuovo token
      return apiClient<T>(endpoint, options);
    }
    // Refresh fallito → redirect a login
    window.location.href = "/login";
    throw new ApiError(401, "Session expired");
  }

  if (!res.ok) {
    const error = await res.json().catch(() => ({}));
    throw new ApiError(res.status, error.message);
  }

  return res.json();
}
```

### Auth Token Management

```typescript
// lib/auth.ts
const ACCESS_TOKEN_KEY = "access_token";
const REFRESH_TOKEN_KEY = "refresh_token";

// ⚠️ Per prod: usa httpOnly cookies invece di localStorage
export function getAccessToken(): string | null {
  if (typeof window === "undefined") return null;
  return localStorage.getItem(ACCESS_TOKEN_KEY);
}

export function setTokens(access: string, refresh: string): void {
  localStorage.setItem(ACCESS_TOKEN_KEY, access);
  localStorage.setItem(REFRESH_TOKEN_KEY, refresh);
}

export async function refreshAccessToken(): Promise<boolean> {
  const refreshToken = localStorage.getItem(REFRESH_TOKEN_KEY);
  if (!refreshToken) return false;

  try {
    const res = await fetch(`${API_BASE}/auth/refresh`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ refresh_token: refreshToken }),
    });

    if (!res.ok) return false;

    const { access_token } = await res.json();
    localStorage.setItem(ACCESS_TOKEN_KEY, access_token);
    return true;
  } catch {
    return false;
  }
}
```

## Conventions

### File & Component Naming

- **Components:** PascalCase file e nome → `MatchCard.tsx`, `export function MatchCard()`
- **Hooks:** camelCase con `use` prefix → `useAuth.ts`, `export function useAuth()`
- **Utils:** camelCase → `formatDate.ts`, `export function formatDate()`
- **Types:** PascalCase interfaces → `export interface Match { ... }`
- **Pages:** kebab-case folders → `app/profile/page.tsx`

### React Best Practices

- **Server Components by default:** Solo "use client" quando hai interattività (onClick, useState, useEffect)
- **Props types:** Inline per componenti semplici (<3 props), interface per complessi
- **Async Server Components:** Fetch data direttamente nei page.tsx quando possibile
- **Error boundaries:** Usa `error.tsx` in ogni route folder per gestione errori
- **Loading states:** Usa `loading.tsx` per suspense boundaries automatici

### TypeScript

- **Strict mode:** Sempre abilitato, no `any` type
- **Infer types:** Preferisci type inference dove chiaro
- **API types:** Mantieni allineati con backend DTOs (considera shared types package)
- **Generic components:** Usa generics per componenti riusabili (es: `Table<T>`)

### Styling

- **Tailwind only:** No CSS modules, no styled-components
- **Custom classes:** In `globals.css` solo per design tokens (colors, spacing)
- **Responsive:** Mobile-first, usa breakpoints Tailwind (`sm:`, `md:`, `lg:`)
- **Dark mode:** Preparato con `dark:` classes, toggle in futuro

## Component Examples

### Feature Component Template

```typescript
// components/features/MatchCard.tsx
'use client';

import { Card, CardHeader, CardTitle, CardContent, CardFooter } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { UserAvatar } from './UserAvatar';
import type { Match } from '@/types';

interface MatchCardProps {
  match: Match;
  onComplete: (id: number) => Promise<void>;
  onSkip: (id: number) => Promise<void>;
}

export function MatchCard({ match, onComplete, onSkip }: MatchCardProps) {
  const isPending = match.status === 'pending';

  return (
    <Card className="hover:shadow-lg transition-shadow">
      <CardHeader className="flex flex-row items-center gap-4">
        <UserAvatar
          name={match.partner.name}
          avatarUrl={match.partner.avatar_url}
        />
        <div>
          <CardTitle>{match.partner.name}</CardTitle>
          <p className="text-sm text-muted-foreground">
            {new Date(match.matched_at).toLocaleDateString()}
          </p>
        </div>
        <Badge variant={isPending ? 'default' : 'secondary'}>
          {match.status}
        </Badge>
      </CardHeader>

      {isPending && (
        <CardFooter className="gap-2">
          <Button
            onClick={() => onComplete(match.id)}
            className="flex-1"
          >
            ☕ Caffè fatto!
          </Button>
          <Button
            variant="outline"
            onClick={() => onSkip(match.id)}
          >
            Salta
          </Button>
        </CardFooter>
      )}
    </Card>
  );
}
```

### Custom Hook Template

```typescript
// hooks/useMatches.ts
"use client";

import { useState, useEffect } from "react";
import { apiClient, ApiError } from "@/lib/api";
import type { Match } from "@/types";

export function useMatches() {
  const [matches, setMatches] = useState<Match[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchMatches();
  }, []);

  async function fetchMatches() {
    try {
      setLoading(true);
      const data = await apiClient<Match[]>("/matches");
      setMatches(data);
    } catch (err) {
      if (err instanceof ApiError) {
        setError(err.message);
      }
    } finally {
      setLoading(false);
    }
  }

  async function completeMatch(id: number) {
    await apiClient(`/matches/${id}/complete`, { method: "PATCH" });
    await fetchMatches(); // Refresh lista
  }

  async function requestOnDemand() {
    const newMatch = await apiClient<Match>("/matches/on-demand", {
      method: "POST",
    });
    setMatches((prev) => [newMatch, ...prev]);
    return newMatch;
  }

  return {
    matches,
    loading,
    error,
    completeMatch,
    requestOnDemand,
    refetch: fetchMatches,
  };
}
```

## Boundaries

### ✅ Always (fai autonomamente)

- Usare TypeScript strict mode, no `any`
- Componenti shadcn per UI standard (Button, Card, Input, etc)
- Tailwind classes per styling (no CSS inline, no styled-components)
- Error handling con try/catch e toast notifications
- Loading states con Skeleton o Spinner
- Responsive design mobile-first

### ⚠️ Ask First (discuti con team)

- Aggiungere nuove dipendenze npm (valuta bundle size)
- Modificare `tailwind.config.ts` (colori, spacing custom)
- Cambiare strategia auth (localStorage → httpOnly cookies)
- Aggiungere route esterne al (auth)/(dashboard) pattern
- Integrare librerie di state management (Zustand, Redux)

### 🚫 Never (vietato)

- CSS inline o styled-components (usa solo Tailwind)
- `any` type in TypeScript (usa `unknown` se necessario)
- Token JWT in localStorage in produzione (migra a httpOnly cookies)
- Fetch diretto senza `apiClient` wrapper (salta error handling)
- Hardcoded API URL nel codice (usa `NEXT_PUBLIC_API_URL`)
- Componenti Client quando Server Component è sufficiente
- Import da `components/ui/` per modificare (copia e personalizza invece)

## Troubleshooting

### `Error: fetch failed` durante API calls

**Causa:** Backend API non running o CORS non configurato

**Fix:**

1. Verifica backend su `http://localhost:3001` con `curl http://localhost:3001/api/health`
2. Assicurati che backend abbia CORS abilitato per `http://localhost:3000`
3. Controlla `NEXT_PUBLIC_API_URL` in `.env.local`

### `401 Unauthorized` su route protette

**Causa:** Access token scaduto (15min) e refresh fallito

**Fix:**

1. Verifica refresh token in localStorage (deve essere valido <7 giorni)
2. Check `refreshAccessToken()` in `lib/auth.ts` con breakpoint
3. Se persistente, logout e re-login per ottenere nuovi token

### shadcn components non trovati dopo `npx shadcn-ui add`

**Causa:** Path alias `@/` non configurato in `tsconfig.json`

**Fix:** Aggiungi in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Layout shift durante caricamento avatar

**Causa:** Immagini senza dimensioni fisse causano reflow

**Fix:** Usa `width` e `height` su `<Image>` o fixed size wrapper:

```tsx
<div className="w-12 h-12 rounded-full overflow-hidden">
  <UserAvatar ... />
</div>
```

### Hydration mismatch con date/timestamp

**Causa:** Server renderizza data in un formato, client in un altro (timezone)

**Fix:** Usa `suppressHydrationWarning` o formatta date solo client-side:

```tsx
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
if (!mounted) return <Skeleton />;
return <time>{formatDate(match.matched_at)}</time>;
```

## Production Checklist

Prima di deploy in produzione:

- [ ] Migra auth da localStorage a httpOnly cookies (vedi `/lib/auth.ts`)
- [ ] Abilita rate limiting su API calls (usa `swr` o `react-query`)
- [ ] Configura error tracking (Sentry/Bugsnag)
- [ ] Setup analytics (Plausible/GA4)
- [ ] Ottimizza bundle size con `npm run analyze` (installa `@next/bundle-analyzer`)
- [ ] Test responsive su mobile (Chrome DevTools device mode)
- [ ] Audit accessibility con Lighthouse (target: score >90)
- [ ] Setup environment variables su hosting (Vercel/Netlify)
- [ ] Test end-to-end con Playwright (login → match → complete flow)

## Future Improvements

Features in roadmap (non implementare senza discussione):

- **Real-time notifications:** WebSocket per nuovi match senza polling
- **Dark mode:** Toggle tema con persistenza preferenza
- **PWA:** Service worker per uso offline
- **Internazionalizzazione:** i18n per multi-language support
- **Analytics dashboard:** Stats personali (caffè bevuti, persone incontrate)

## Resources

- [Next.js 14 Docs](https://nextjs.org/docs) - App Router reference
- [shadcn/ui Docs](https://ui.shadcn.com) - Component library
- [Tailwind CSS Docs](https://tailwindcss.com/docs) - Utility classes
- [React Server Components](https://react.dev/reference/react/use-server) - RSC patterns
