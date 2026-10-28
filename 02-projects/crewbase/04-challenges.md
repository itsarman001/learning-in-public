# Challenges

## Prisma 7 breaking changes
- **Date:** Oct 25-26, 2026
- **Problem:** A newer major version of Prisma was installed than the tutorial uses. Prisma 7 removed `url` support from the datasource block in `schema.prisma`, so the connection URL now goes in `prisma.config.ts`. A `generator` `output` path is also required.
- **Impact:** Simple tasks took much longer than following the video would have, and progress stalled.
- **Resolution:** Downgraded to Prisma 6 (see decision 4).

## Typo in `generateToken` broke the register API
- **Date:** Oct 27-28, 2026
- **Problem:** The register API kept failing. While creating `generateToken`, I wrote `{JWT_SECRET}` instead of `JWT_SECRET`, and it went unnoticed.
- **Cause:** The braces turn the secret into an object instead of the string, so `jwt.sign` doesn't get a valid secret.
- **Resolution:** Found it while debugging the register API and removed the braces.

```ts
export const generateToken = (userId: string): string => {
  return jwt.sign({ userId }, JWT_SECRET, { expiresIn: "7d" });
};
```
