# Challenges

## Prisma 7 breaking changes
- **Date:** Oct 25-26, 2026
- **Problem:** A newer major version of Prisma was installed than the tutorial uses. Prisma 7 removed `url` support from the datasource block in `schema.prisma`, so the connection URL now goes in `prisma.config.ts`. A `generator` `output` path is also required.
- **Impact:** Simple tasks took much longer than following the video would have, and progress stalled.
- **Resolution:** Downgraded to Prisma 6 (see decision 4).
