# Crewbase: Team & RBAC Management Platform

**Status:** In progress | **Stack:** Next.js 16, Prisma, PostgreSQL | **Repo:** [link](https://github.com/itsarman001/crewbase.git) | **YouTube:** [link](https://youtu.be/b05ky3LdzKU)
## Why I'm building this
I came across this project and saw it covers everything I need to start my backend journey. I'm documenting what I learn, what I do differently from the original, and the features I add myself.

## Learning checklist

**Foundations**
- [ ] Next.js 16 and its folder/file structure
- [ ] Pages and layouts
- [ ] App Router (route groups, dynamic routes)
- [ ] Server vs client components

**Backend**
- [ ] Prisma schema and client setup
- [ ] PostgreSQL
- [ ] REST API routes
- [ ] JWT auth and cookies
- [ ] Login and register flow
- [ ] Protected API routes
- [ ] RBAC (admin management and users)

**Frontend integration**
- [ ] Auth context
- [ ] Forms with `useActionState`
- [ ] API integration with the UI
- [ ] Role-based dashboard
- [ ] Server mutations

**Quality**
- [ ] App testing

## Dependencies
| Package | Why | Dev only |
|---|---|---|
| `@prisma/client` | Type-safe database client used in API routes | No |
| `prisma` | CLI for schema, migrations, client generation | Yes (usually) |
| `bcryptjs` | Hashes passwords (pure JS, no native build) | No |
| `jsonwebtoken` | Signs and verifies JWTs | No |
| `@types/bcryptjs` | TypeScript types | Yes |
| `@types/jsonwebtoken` | TypeScript types | Yes |

## Architecture
```mermaid
flowchart TD

    ROOT["Next.js Team Access Management"] --> BE["APIs / Database"]

    ROOT --> UI["User Interface"]

  

    %% ---------- Backend ----------

    BE --> MW["Middleware<br/>verify JWT → check role → allow / deny"]

    BE --> AUTH["Auth"]

    BE --> USERS["Users"]

    BE --> ACCESS["Access Management"]

    BE --> HELPER["Helper Functions"]

  

    %% Auth routes

    AUTH --> LOGIN["POST /auth/login"]

    AUTH --> REGISTER["POST /auth/register"]

    AUTH --> LOGOUT["POST /auth/logout"]

    AUTH --> ME["GET /auth/me"]

  

    %% Users routes

    USERS --> U_USERS["GET /users"]

    USERS --> U_TEAMS["GET /teams"]

    USERS --> U_ROLES["GET /roles"]

  

    %% Access management endpoints

    ACCESS --> AC1["GET /users/:id<br/>user details"]

    ACCESS --> AC2["GET /teams/:id<br/>team details"]

    ACCESS --> AC3["PATCH /users/:id/role<br/>assign role"]

    ACCESS --> AC4["POST /teams/:id/members<br/>assign team"]

    ACCESS --> AC5["DELETE /teams/:id/members/:userId<br/>remove team"]

  

    %% Middleware protects these

    MW -. protects .-> USERS

    MW -. protects .-> ACCESS

  

    %% Helpers

    HELPER --> LIB["Lib"]

    LIB --> JWT["JWT Token"]

    LIB --> COOKIES["Cookies"]

    LIB --> DBC["DB Connect"]

    DBC --> PRISMA["Prisma"]

    PRISMA --> PG[("Postgres")]

  

    %% Roles live in the schema

    PRISMA --> ROLEENUM["Role enum"]

    ROLEENUM --> ADMIN["Admin"]

    ROLEENUM --> MANAGER["Manager"]

    ROLEENUM --> USER["User"]

    ROLEENUM --> GUEST["Guest"]

    U_ROLES -. returns .-> ROLEENUM

  

    %% Shared helper usage

    AUTH -. uses .-> LIB

    MW -. uses .-> LIB

    USERS -. uses .-> DBC

    ACCESS -. uses .-> DBC

  

    %% ---------- Frontend ----------

    UI --> PAGES["Pages"]

    UI --> CTX["Auth Context"]

    UI --> DASH["Role-based Dashboard"]

    PAGES --> P1["Login page"]

    PAGES --> P2["Register page"]

    PAGES --> P3["Users page"]

    PAGES --> P4["Teams page"]

    UI -. calls .-> BE
```

## Auth and RBAC request flow
```mermaid
sequenceDiagram
  Client->>API: PATCH /users/:id/role (cookie)
  API->>Middleware: verify JWT
  Middleware->>Middleware: check role = Admin
  Middleware-->>Client: 401/403 if denied
  Middleware->>DB: update role via Prisma
  DB-->>Client: 200 updated user
```

## What I changed or added
- (e.g. Manager and Guest roles, team member endpoints)

## Challenges and fixes
- (short entries: symptom, cause, fix)

## Progress log
- 2026-10-23: started documenting
