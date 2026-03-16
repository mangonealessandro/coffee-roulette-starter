# Coffee Roulette – API

## Overview

Backend REST API to match random colleagues for coffee breaks.
Handles automatic weekly matches and real-time on-demand requests.

**Problem solved:** Encourages spontaneous connections between colleagues in remote/hybrid companies.

## Stack

| Tech    | Version | Notes                                                                              |
| ------- | ------- | ---------------------------------------------------------------------------------- |
| NestJS  | 10+     | Main framework, chosen for modular architecture and native TypeScript support      |
| SQLite  | 3       | Local database for dev (prod will use PostgreSQL)                                  |
| TypeORM | 0.3+    | ORM for entities. SQLite = no advanced features (full-text search, JSON operators) |
| JWT     | -       | Access token 15min (security), Refresh 7d (UX)                                     |
| Jest    | -       | Testing framework, configured for unit + integration                               |

## Getting Started

### Prerequisites

- Node.js 18+
- npm 9+

### Setup

```bash
# Install dependencies
npm install

# Create database and tables
npm run db:migrate

# Populate with fake users for dev
npm run db:seed

# Start in dev mode
npm run start:dev  # Port 3001
```

### Running

```bash
npm run start:dev    # Watch mode on port 3001
npm run test         # Run all tests
npm run test:watch   # Test in watch mode
npm run build        # Production build
```

**Required Env vars:**

- `JWT_SECRET` - Secret for signing tokens (generate with `openssl rand -base64 32`)
- `JWT_REFRESH_SECRET` - Secret for refresh tokens
- `DATABASE_PATH` - Path to SQLite file (default: `./db.sqlite`)

## Architecture

```
api/src/
├── auth/              # Authentication: login, register, JWT strategy
│   ├── dto/           # LoginDto, RegisterDto
│   ├── guards/        # JwtAuthGuard
│   └── strategies/    # JWT Passport strategy
├── users/             # User and profile management
│   ├── dto/           # UpdateProfileDto, ToggleAvailabilityDto
│   └── users.service.ts
├── matches/           # Core business logic
│   ├── dto/           # CreateMatchDto, CompleteMatchDto
│   ├── matches.service.ts  # Matching algorithm
│   └── matches.cron.ts     # Automatic weekly job
├── common/            # Shared utilities
│   ├── guards/        # AuthGuard, RolesGuard
│   ├── decorators/    # @CurrentUser(), @Public()
│   └── filters/       # GlobalExceptionFilter
└── database/
    ├── entities/      # TypeORM entities (User, Match)
    └── seeds/         # Seed data for dev/test
```

**Architectural Pattern:** Modular monolith with domain separation

**Main Data Flow:**

1. Request → Guard (JWT validation) → Controller
2. Controller → Service (business logic) → Repository (TypeORM)
3. Service → Response DTO → Client

## Database Schema

### users

| Column        | Type    | Notes                          |
| ------------- | ------- | ------------------------------ |
| id            | INTEGER | PK, autoincrement              |
| email         | TEXT    | Unique, index for fast lookups |
| password_hash | TEXT    | Bcrypt with salt rounds = 10   |
| name          | TEXT    | Display name                   |
| avatar_url    | TEXT    | Nullable, external URL         |
| is_available  | INTEGER | 0/1, flag for on-demand match  |
| created_at    | TEXT    | ISO 8601 timestamp             |

### matches

| Column       | Type    | Notes                                   |
| ------------ | ------- | --------------------------------------- |
| id           | INTEGER | PK                                      |
| user_one_id  | INTEGER | FK → users, part of composite index     |
| user_two_id  | INTEGER | FK → users, part of composite index     |
| type         | TEXT    | 'weekly' (cron) or 'on_demand' (manual) |
| status       | TEXT    | 'pending' → 'completed' or 'skipped'    |
| matched_at   | TEXT    | ISO 8601, when created                  |
| completed_at | TEXT    | Nullable, when marked completed         |

**Indexes:** Composite on `(user_one_id, user_two_id)` to prevent duplicates and speed up queries.

## API Reference

### Auth

| Method | Endpoint             | Body                        | Description                    |
| ------ | -------------------- | --------------------------- | ------------------------------ |
| POST   | `/api/auth/register` | `{ email, password, name }` | New user registration          |
| POST   | `/api/auth/login`    | `{ email, password }`       | Login → access + refresh token |
| POST   | `/api/auth/refresh`  | `{ refresh_token }`         | Renew access token             |

### Users

| Method | Endpoint                     | Body                    | Description                |
| ------ | ---------------------------- | ----------------------- | -------------------------- |
| GET    | `/api/users/me`              | -                       | Current user profile       |
| PATCH  | `/api/users/me`              | `{ name?, avatar_url?}` | Update profile             |
| PATCH  | `/api/users/me/availability` | `{ is_available }`      | Toggle coffee availability |

### Matches

| Method | Endpoint                    | Body | Description                           |
| ------ | --------------------------- | ---- | ------------------------------------- |
| GET    | `/api/matches`              | -    | User match list (pending + history)   |
| POST   | `/api/matches/on-demand`    | -    | Create instant match with random user |
| PATCH  | `/api/matches/:id/complete` | -    | Mark match as completed               |
| PATCH  | `/api/matches/:id/skip`     | -    | Skip match (not interested)           |

**Auth:** All endpoints require JWT except `/auth/register` and `/auth/login`.

## Conventions

### Naming

- **Entities:** Singular PascalCase → `User`, `Match`
- **DTOs:** Action + Entity + Dto → `CreateUserDto`, `UpdateMatchDto`
- **Files:** kebab-case → `users.service.ts`, `auth.controller.ts`
- **DB columns:** snake_case → `user_id`, `created_at`
- **Endpoints:** kebab-case → `/users/me`, `/matches/on-demand`

### Code Style

- **1 module per feature:** `auth.module.ts` imports `UsersModule` if user lookup is needed
- **Mandatory DTOs:** Every input/output must have DTO with class-validator
- **Entities in `/database/entities/`:** Never import directly from service, always via Repository
- **Guards everywhere:** Use `@UseGuards(JwtAuthGuard)` or global guard with `@Public()` decorator for exceptions

### Testing

- **Unit test:** Mock dependencies, test isolated business logic
- **Integration test:** In-memory database for E2E tests of complete flows
- **Minimum coverage:** 80% on services, 60% on controllers

### Example Complete DTO

```typescript
// src/auth/dto/register.dto.ts
import { IsEmail, IsString, MinLength } from "class-validator";

export class RegisterDto {
  @IsEmail({}, { message: "Invalid email" })
  email: string;

  @IsString()
  @MinLength(8, { message: "Password must be at least 8 characters" })
  password: string;

  @IsString()
  @MinLength(2, { message: "Name must be at least 2 characters" })
  name: string;
}
```

### Example Service Pattern

```typescript
// src/matches/matches.service.ts
@Injectable()
export class MatchesService {
  constructor(
    @InjectRepository(Match) private matchRepo: Repository<Match>,
    @InjectRepository(User) private userRepo: Repository<User>,
  ) {}

  async createOnDemandMatch(currentUserId: number): Promise<Match> {
    // 1. Find available users (excluding current user)
    // 2. Filter users already matched this week
    // 3. Random pick
    // 4. Create match with type='on_demand'
  }
}
```

## Boundaries

### ✅ Always (do autonomously)

- Add DTOs with class-validator for new endpoints
- Create unit tests for new services
- Use snake_case for database columns
- Wrap logic in transactions for multi-step operations
- Log errors with NestJS `Logger`

### ⚠️ Ask First (discuss with team)

- Add new entities or modify existing schema
- Change JWT strategy (expiration, refresh logic)
- Introduce new external dependencies
- Modify matching algorithm
- Create new cron jobs

### 🚫 Never (forbidden)

- Plaintext passwords or custom hash (always use bcrypt)
- Hardcoded JWT secrets in code (env vars only)
- Raw SQL queries (always use TypeORM query builder)
- Disable validation on DTOs
- Commit `db.sqlite` file (it is in `.gitignore`)
- Expose stack traces in API response (use exception filters)

## Troubleshooting

### `SQLITE_BUSY: database is locked`

**Cause:** Multiple processes accessing DB simultaneously (e.g., test + dev server)

**Fix:** Close other processes or use `DATABASE_PATH=./test.db` for tests

### `UnauthorizedException: jwt expired`

**Cause:** Access token expired (15min lifetime)

**Fix:** Frontend must call `/auth/refresh` with refresh token

### `QueryFailedError: UNIQUE constraint failed: users.email`

**Cause:** Attempt to register already existing email

**Fix:** Handled by `GlobalExceptionFilter`, returns 409 Conflict. Frontend must show appropriate error.

### TypeORM migration fails with SQLite

**Cause:** SQLite does not support complex ALTER TABLE (add/drop constraints)

**Fix:** Create new table → copy data → drop old → rename. See `migrations/XXXXXX-example.ts`

### Match algorithm does not find available users

**Cause:** All users have `is_available = 0` or are already matched

**Fix:** Algorithm returns `null`, controller handles with 404 "No users available at the moment"

## Migration from SQLite to PostgreSQL (Prod)

When moving to prod:

1. Change driver in `app.module.ts`: `type: 'postgres'`
2. Update env vars: `DATABASE_URL=postgresql://...`
3. Re-run migrations: `npm run migration:run`
4. Enable PostgreSQL features if needed (full-text search, JSON operators)

**Key differences:**

- PostgreSQL has native `BOOLEAN` (SQLite uses INTEGER 0/1)
- PostgreSQL supports `ARRAY` columns (useful for future features)
- Better timestamp handling in PostgreSQL

## Resources

- [NestJS Docs](https://docs.nestjs.com) - Framework reference
- [TypeORM Docs](https://typeorm.io) - ORM and migrations
- [SQLite Limitations](https://www.sqlite.org/limits.html) - What to avoid
- [JWT Best Practices](https://tools.ietf.org/html/rfc8725) - Security considerations
