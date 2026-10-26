# Decisions

## 1. Package manager: pnpm over npm
- **Date:** Oct 12, 2026
- **Why:** `npm` was taking far too long to install packages.

## 2. Prisma 7 instead of Prisma 6 (later reversed)
- **Date:** Oct 25, 2026
- **Why:** Following Prisma 6 felt like blindly copying the tutorial. Prisma 7 seemed better for my learning.
- **Status:** Superseded by decision 4.

## 3. Docker instead of a local PostgreSQL install
- **Date:** Oct 25, 2026
- **Why:** The tutorial installs PostgreSQL locally on a Mac, but I didn't think that was a good idea for my setup.
- **How:** Used Docker Desktop. Asked Claude to create a `docker-compose.yml`, then ran the migration.

## 4. Downgrade from Prisma 7 to Prisma 6
- **Date:** Oct 26, 2026
- **Why:** Prisma 7 kept slowing me down on simple tasks I could finish in minutes by following the video. I was struggling instead of learning.

|                | Prisma 7              | Prisma 6                     |
| -------------- | --------------------- | ---------------------------- |
| Learning value | Higher in theory      | Steady, matches the tutorial |
| Friction       | High, slowed progress | Low                          |
| Outcome        | Dropped               | Chosen                       |