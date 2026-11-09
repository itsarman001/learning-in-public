# Changelog

Dated progress log for Crewbase. Reasoning lives in `decisions.md` and `challenges.md`.

---

## Oct 12, 2026, 21:39 - Project & Tools Setup

- Switched from `npm` to `pnpm` because installs were too slow. See decision 1.

## Oct 25, 2026, 17:33 - Creating Prisma Schema & Generating the Client

- Went with Prisma 7 instead of 6 and updated the schema for its changes (`output` path required, `url` moved to `prisma.config.ts`). See decision 2 and the Prisma 7 challenge.

## Oct 25, 2026, 18:53 - Connecting to the PostgreSQL DB

- Used Docker Desktop instead of a local Postgres install. Created `docker-compose.yml` and ran the migration. See decision 3.

## Oct 26, 2026, 01:30 - Creating the API

- Downgraded from Prisma 7 to Prisma 6 after it kept slowing me down. See decision 4.

## Oct 27, 2026 - Auth Helpers

- Implemented the authentication helpers:
  - Password hashing
  - Token generation
  - User permission check
- Found and fixed a typo in `generateToken` while debugging why the register API was failing. See the challenge in `challenges.md`.

## Oct 28, 2026 - Register API

- Fixed a casing error in the Prisma user model: `Team` -> `team`.
- Implemented user registration with validation, role assignment, and token generation.
- Debugged and fixed minor bugs in the `generateToken` helper and the register API.

## Oct 29, 2026 - Login API

- Implemented login with user validation and token generation.

## Oct 30, 2026 - Logout, Me & User Management APIs

**Auth**

- Implemented logout with token invalidation.
- Cleaned up the login API by removing unnecessary returns.
- Fixed a typo in the authentication error messages.

**Current user**

- Implemented a GET endpoint to retrieve the current user's information, with error handling.

**User management**

- Implemented a PATCH endpoint for assigning a user to a team, with authorization checks.
- Implemented a PATCH endpoint for updating user roles, with authorization checks.

**Database**

- Implemented database seeding for teams and users with hashed passwords.

## Nov 1, 2026 - Team & Role Assignment Routes
- Merged PR #4 (`feat/team-user-api-enhance`) adding team and role assignment routes for users.
- Renamed the route folder `[userid]` -> `[userId]` so params resolve correctly.
- **Team assignment:** accepts `Team.id`, verifies the team exists, and allows `null` to unassign.
- **Role update:** validates against `MANAGER`/`USER`, blocks self-change, and blocks changing other admins.
- Returned proper status codes: 401 unauthenticated, 403 unauthorized, 400 invalid JSON or body, 404 missing user or team.
- Used Prisma's `P2025` error code for not-found handling instead of matching the error message.
- Selected only safe user fields so the password hash is never returned.

## Nov 2, 2026 - UI Planning & Foundation (plan.md Steps 1-4)

**Setup**
- Added `AGENTS.md` with project guidelines and `pnpm` commands (tech stack, code rules, Next.js 16 notes, Git practices).
- Added `mcp_config.json` for the shadcn MCP server.
- Added `plan.md` to define the scope, phases, and execution order for building the UI with shadcn/ui.
- Added `components.json` and the initial dependencies (`@base-ui/react`, `class-variance-authority`, `cn`, `lucide-react`, `shadcn`, `tw-animate-css`), plus a `cn` utility and a `Button` component with variants and sizes.

**Step 1 - Component library & theme**
- Added the UI component library and updated theme styling.

**Step 2 - Providers & cleanup**
- Added theme variables and layout providers.
- Refactored the auth route: removed an unused `request` parameter and sanitized the user object without destructuring.

**Step 3 - Dashboard**
- Added the dashboard overview page, navigation sidebar, and mock data infrastructure.

**Step 4 - Authentication UI**
- Implemented the authentication layout and the login and register pages with forms.

## Nov 3, 2026 - Remaining Phase 1 Screens (plan.md Step 5)
- Completed Phase 1 of `plan.md` by implementing the users, teams, roles, profile, and system states screens.
- Refactored the UI to use direct named imports for React hooks and types.
