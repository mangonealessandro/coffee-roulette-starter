# Coffee Roulette

## Objective

Internal app to randomly match colleagues for a coffee. Automatic weekly match + on-demand instant match capability.

## Monorepo Structure

```
coffee-roulette/
├── api/          # NestJS Backend
└── web/          # Next.js Frontend
```

Each folder has its own `claude.md` with specific rules. ALWAYS read the `claude.md` of the folder you are working in.

## Global Stack

| Layer    | Tech                 | Version          |
| -------- | -------------------- | ---------------- |
| Backend  | NestJS               | 10+              |
| Frontend | Next.js              | 14+              |
| Database | SQLite               | 3                |
| Auth     | JWT                  | Access + Refresh |
| UI       | Tailwind + shadcn/ui | Latest           |

## Global Conventions

- Package manager: `npm` (never yarn or pnpm)
- DB column naming: `snake_case`
- Function naming: `camelCase`
- Class/component naming: `PascalCase`
- Code language: English
- Comment language: English

## Global Boundaries

- ✅ **Always:** Follow naming conventions, write in English
- ⚠️ **Ask first:** Cross-project changes (api ↔ web)
- 🚫 **Never:** Commit secrets, modify .env files, install different package managers

## API ↔ Web Communication

- API Base URL: `http://localhost:3001/api`
- Frontend dev: `http://localhost:3000`
- All API endpoints are prefixed with `/api`

## Agent Workflow

When the user explicitly asks to "start with the implementation" or "implement the PRP doc", or similar phrases related to starting implementation from requirements:

- You MUST call the `prp-orchestrator` agent.
- Do not attempt to implement the full feature set yourself without the orchestrator's plan.
- The `prp-orchestrator` will analyze the requirements (PRD), create a task plan (PRP), and coordinate execution.
