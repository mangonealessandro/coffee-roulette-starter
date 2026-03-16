# Coffee Roulette – Web

## Overview

Frontend web application to manage random coffee matches between colleagues.
Allows login, viewing weekly/on-demand matches, and requesting coffee in real-time.

**Problem solved:** User-friendly interface to foster spontaneous connections between remote/hybrid colleagues.

## Stack

| Tech         | Version | Notes                                                                |
| ------------ | ------- | -------------------------------------------------------------------- |
| Next.js      | 14+     | App Router (not Pages Router), chosen for SSR and file-based routing |
| React        | 18+     | Server Components by default, Client Components only where necessary |
| Tailwind CSS | 3+      | Utility-first CSS, configured for company design system              |
| shadcn/ui    | Latest  | Headless components + Radix UI, 100% customizable                    |
| TypeScript   | 5+      | Strict mode enabled for maximum type safety                          |

## Getting Started

### Prerequisites

- Node.js 18+
- npm 9+
- Backend API running on `http://localhost:3001` (see `api/claude.md`)

### Setup

```bash
# Install dependencies
npm install

# Copy env vars
cp .env.example .env.local
# Edit NEXT_PUBLIC_API_URL if API is not on localhost:3001

# Install shadcn base components
npx shadcn-ui@latest init

# Start dev server
npm run dev  # Port 3000, hot reload enabled
```

### Running

```bash
npm run dev          # Dev server with hot reload (port 3000)
npm run build        # Optimized production build
npm run start        # Serve production build
npm run lint         # ESLint + type checking
npm run lint:fix     # Auto-fix linting issues
npx shadcn-ui add [component]  # Add shadcn components
```

**Required Env vars:**

- `NEXT_PUBLIC_API_URL` - Backend API Base URL (default: `http://localhost:3001/api`)
- `NEXT_PUBLIC_APP_NAME` - App name for meta tags (default: `Coffee Roulette`)

## Architecture

```
web/
├── src/
│   ├── app/                    # Next.js 14 App Router
│   │   ├── (auth)/             # Route group: authentication (no dashboard layout)
│   │   │   ├── login/
│   │   │   │   └── page.tsx    # Login form
│   │   │   └── register/
│   │   │       └── page.tsx    # Registration form
│   │   ├── (dashboard)/        # Route group: protected area (with sidebar)
│   │   │   ├── layout.tsx      # Layout with sidebar + auth check
│   │   │   ├── matches/
│   │   │   │   └── page.tsx    # Match list
│   │   │   └── profile/
│   │   │       └── page.tsx    # User profile
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
│   │   ├── api.ts              # API client with fetch wrapper + error handling
│   │   ├── auth.ts             # JWT token management + refresh logic
│   │   └── utils.ts            # Helper functions (cn, formatDate, etc)
│   ├── hooks/
│   │   ├── useAuth.ts          # Hook for login/logout/user state
│   │   ├── useMatches.ts       # Hook for fetch/update matches
│   │   └── useToast.ts         # shadcn toast wrapper
│   └── types/
│       ├── api.ts              # Response types from backend
│       └── index.ts            # Shared types (User, Match, etc)
├── public/
│   ├── avatars/                # Avatar placeholder images
│   └── favicon.ico
└── tailwind.config.ts          # Tailwind + shadcn theme config
```

**Architectural Pattern:** Feature-based structure with UI/Logic separation

**Data flow:**

1. Page (Server Component) → pre-fetch data if possible
2. Client Component → useHook → API client → Backend
3. API client → handles errors + refresh token → returns data
4. Hook → updates state → UI re-render

## Pages & Routes

| Route       | Type               | Description                      | Auth Required |
| ----------- | ------------------ | -------------------------------- | ------------- |
| `/`         | Server Component   | Landing page with redirect logic | No            |
| `/login`    | Client Component   | Login form with validation       | No            |
| `/register` | Client Component   | Registration form                | No            |
| `/matches`  | Server Component\* | Match list (pending + completed) | Yes           |
| `/profile`  | Client Component   | Profile + availability toggle    | Yes           |

\*Server Component with nested Client Components for interactivity

**Auth flow:**

- Routes in `(auth)` → accessible only if not logged in (redirect to `/matches` if logged in)
- Routes in `(dashboard)` → protected, redirect to `/login` if not authenticated
- Auth check in `(dashboard)/layout.tsx` via middleware or server-side check

## Key Components

### Features Components

| Component            | Path                                     | Description                                              |
| -------------------- | ---------------------------------------- | -------------------------------------------------------- |
| `CoffeeButton`       | `components/features/CoffeeButton`       | CTA "I want a coffee!" → POST `/matches/on-demand`       |
| `MatchCard`          | `components/features/MatchCard`          | Match card with avatar, name, date, actions              |
| `AvailabilityToggle` | `components/features/AvailabilityToggle` | Switch for availability → PATCH `/users/me/availability` |
| `UserAvatar`         | `components/features/UserAvatar`         | Avatar with name initials fallback                       |
| `MatchList`          | `components/features/MatchList`          | Match list container with pending/completed filter       |

### shadcn UI Components (already installed)

- `Button` - Variants: default, outline, ghost, destructive
- `Card` - With CardHeader, CardContent, CardFooter
- `Avatar` - With AvatarImage, AvatarFallback
- `Badge` - For match status (pending, completed, skipped)
- `Input`, `Label` - Form controls
- `Switch` - Availability toggle
- `Toast` - Real-time notifications (new match, errors)
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
  const token = getAccessToken(); // From lib/auth.ts

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
      // Retry request with new token
      return apiClient<T>(endpoint, options);
    }
    // Refresh failed → redirect to login
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

// ⚠️ For prod: use httpOnly cookies instead of localStorage
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

- **Components:** PascalCase file and name → `MatchCard.tsx`, `export function MatchCard()`
- **Hooks:** camelCase with `use` prefix → `useAuth.ts`, `export function useAuth()`
- **Utils:** camelCase → `formatDate.ts`, `export function formatDate()`
- **Types:** PascalCase interfaces → `export interface Match { ... }`
- **Pages:** kebab-case folders → `app/profile/page.tsx`

### React Best Practices

- **Server Components by default:** Only "use client" when you have interactivity (onClick, useState, useEffect)
- **Props types:** Inline for simple components (<3 props), interface for complex ones
- **Async Server Components:** Fetch data directly in page.tsx when possible
- **Error boundaries:** Use `error.tsx` in every route folder for error handling
- **Loading states:** Use `loading.tsx` for automatic suspense boundaries

### TypeScript

- **Strict mode:** Always enabled, no `any` type
- **Infer types:** Prefer type inference where clear
- **API types:** Keep aligned with backend DTOs (consider shared types package)
- **Generic components:** Use generics for reusable components (e.g., `Table<T>`)

### Styling

- **Tailwind only:** No CSS modules, no styled-components
- **Custom classes:** In `globals.css` only for design tokens (colors, spacing)
- **Responsive:** Mobile-first, use Tailwind breakpoints (`sm:`, `md:`, `lg:`)
- **Dark mode:** Prepared with `dark:` classes, toggle in future

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
            ☕ Coffee done!
          </Button>
          <Button
            variant="outline"
            onClick={() => onSkip(match.id)}
          >
            Skip
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
    await fetchMatches(); // Refresh list
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

### ✅ Always (do autonomously)

- Use TypeScript strict mode, no `any`
- shadcn components for standard UI (Button, Card, Input, etc)
- Tailwind classes for styling (no inline CSS, no styled-components)
- Error handling with try/catch and toast notifications
- Loading states with Skeleton or Spinner
- Mobile-first responsive design

### ⚠️ Ask First (discuss with team)

- Add new npm dependencies (evaluate bundle size)
- Modify `tailwind.config.ts` (custom colors, spacing)
- Change auth strategy (localStorage → httpOnly cookies)
- Add routes outside the (auth)/(dashboard) pattern
- Integrate state management libraries (Zustand, Redux)

### 🚫 Never (forbidden)

- Inline CSS or styled-components (use only Tailwind)
- `any` type in TypeScript (use `unknown` if necessary)
- JWT tokens in localStorage in production (migrate to httpOnly cookies)
- Direct fetch without `apiClient` wrapper (skips error handling)
- Hardcoded API URL in code (use `NEXT_PUBLIC_API_URL`)
- Client Components when Server Component is sufficient
- Import from `components/ui/` to modify (copy and customize instead)

## Troubleshooting

### `Error: fetch failed` during API calls

**Cause:** Backend API not running or CORS not configured

**Fix:**

1. Verify backend on `http://localhost:3001` with `curl http://localhost:3001/api/health`
2. Ensure backend has CORS enabled for `http://localhost:3000`
3. Check `NEXT_PUBLIC_API_URL` in `.env.local`

### `401 Unauthorized` on protected routes

**Cause:** Access token expired (15min) and refresh failed

**Fix:**

1. Verify refresh token in localStorage (must be valid <7 days)
2. Check `refreshAccessToken()` in `lib/auth.ts` with breakpoint
3. If persistent, logout and re-login to get new tokens

### shadcn components not found after `npx shadcn-ui add`

**Cause:** Path alias `@/` not configured in `tsconfig.json`

**Fix:** Add to `tsconfig.json`:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Layout shift during avatar loading

**Cause:** Images without fixed dimensions cause reflow

**Fix:** Use `width` and `height` on `<Image>` or fixed size wrapper:

```tsx
<div className="w-12 h-12 rounded-full overflow-hidden">
  <UserAvatar ... />
</div>
```

### Hydration mismatch with date/timestamp

**Cause:** Server renders date in one format, client in another (timezone)

**Fix:** Use `suppressHydrationWarning` or format dates client-side only:

```tsx
const [mounted, setMounted] = useState(false);
useEffect(() => setMounted(true), []);
if (!mounted) return <Skeleton />;
return <time>{formatDate(match.matched_at)}</time>;
```

## Production Checklist

Before production deploy:

- [ ] Migrate auth from localStorage to httpOnly cookies (see `/lib/auth.ts`)
- [ ] Enable rate limiting on API calls (use `swr` or `react-query`)
- [ ] Configure error tracking (Sentry/Bugsnag)
- [ ] Setup analytics (Plausible/GA4)
- [ ] Optimize bundle size with `npm run analyze` (install `@next/bundle-analyzer`)
- [ ] Test responsive on mobile (Chrome DevTools device mode)
- [ ] Audit accessibility with Lighthouse (target: score >90)
- [ ] Setup environment variables on hosting (Vercel/Netlify)
- [ ] Test end-to-end with Playwright (login → match → complete flow)

## Future Improvements

Features in roadmap (do not implement without discussion):

- **Real-time notifications:** WebSocket for new matches without polling
- **Dark mode:** Theme toggle with preference persistence
- **PWA:** Service worker for offline use
- **Internationalization:** i18n for multi-language support
- **Analytics dashboard:** Personal stats (coffees drunk, people met)

## Resources

- [Next.js 14 Docs](https://nextjs.org/docs) - App Router reference
- [shadcn/ui Docs](https://ui.shadcn.com) - Component library
- [Tailwind CSS Docs](https://tailwindcss.com/docs) - Utility classes
- [React Server Components](https://react.dev/reference/react/use-server) - RSC patterns
