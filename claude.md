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

1. **Step 1: Plan with PRP Orchestrator**
   - You MUST call the `prp-orchestrator` agent first.
   - Do NOT attempt to implementation yourself.
   - The `prp-orchestrator` will analyze the requirements (PRD) and produce a structured Task Plan (PRP) where each task is assigned to a specific domain expert (e.g., Backend Developer, Frontend Developer, DB Expert).

2. **Step 2: Delegate to Specialist Agents**
   - Once the `prp-orchestrator` provides the plan, you must NOT implement the tasks yourself.
   - You MUST explicitly call the corresponding subagent for each task defined in the PRP.
   - Example: If the PRP lists a backend task, call the `backend-developer` agent. If it lists a UI task, call `frontend-developer` or `ui-designer`.
   - Pass the specific task details to the subagent using the `runSubagent` tool.

3. **Step 3: Review & iterate**
   - After subagents complete their work, review the output and ensure integration.
