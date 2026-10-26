# Changelog

Dated progress log for Crewbase. Reasoning lives in `03-decisions.md` and `04-challenges.md`.

---

## Oct 12, 2026, 21:39 - Project & Tools Setup
- Switched from `npm` to `pnpm` because installs were too slow. See decision 1.

## Oct 25, 2026, 17:33 - Creating Prisma Schema & Generating the Client
- Went with Prisma 7 instead of 6 and updated the schema for its changes (`output` path required, `url` moved to `prisma.config.ts`). See decision 2 and the Prisma 7 challenge.

## Oct 25, 2026, 18:53 - Connecting to the PostgreSQL DB
- Used Docker Desktop instead of a local Postgres install. Created `docker-compose.yml` and ran the migration. See decision 3.

## Oct 26, 2026, 01:30 - Creating the API
- Downgraded from Prisma 7 to Prisma 6 after it kept slowing me down. See decision 4.