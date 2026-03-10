---
name: deployment-ready
description: Prepare a web app for deployment — environment config, build optimization, health checks, error monitoring, and platform-specific setup for Vercel, Cloudflare, Railway, or Docker. Use before first deploy or when fixing production issues.
license: MIT
---

## What I Do

Take a working local app and make it deployable, with proper env handling, build config, health checks, and monitoring.

## When to Use Me

- Deploying an app for the first time
- Fixing "works locally, breaks in production" issues
- Adding health checks, error tracking, or analytics
- Switching deployment platforms
- Optimizing build size or cold start times

## Pre-Deploy Checklist

### 1. Environment Variables

```bash
# .env.example — every var documented
DATABASE_URL=           # SQLite: file:./local.db | Postgres: postgresql://...
AUTH_SECRET=            # Random string, min 32 chars: openssl rand -base64 32
NEXT_PUBLIC_APP_URL=    # https://myapp.vercel.app (no trailing slash)
```

Rules:
- `NEXT_PUBLIC_*` for client-accessible vars (Next.js)
- Never hardcode URLs — use env vars for all external services
- Validate ALL env vars at startup with Zod:

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  AUTH_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  NEXT_PUBLIC_APP_URL: z.string().url().optional(),
});

export const env = envSchema.parse(process.env);
```

### 2. Health Check Endpoint

Every deployable app needs:

```typescript
// app/api/health/route.ts (Next.js)
export async function GET() {
  try {
    // Check database connection
    await db.execute(sql`SELECT 1`);
    return Response.json({
      status: 'ok',
      timestamp: new Date().toISOString(),
      version: process.env.COMMIT_SHA || 'dev',
    });
  } catch (error) {
    return Response.json({ status: 'error', message: 'Database unreachable' }, { status: 503 });
  }
}
```

### 3. Build Verification

Before deploying, run locally:
```bash
npm run build          # Must complete without errors
npm run start          # Must serve the built app
curl localhost:3000/api/health  # Must return 200
```

### 4. Error Handling for Production

- Replace `console.error` with structured logging (pino, winston)
- Add global error boundary in React:
```typescript
// app/error.tsx
'use client';
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen gap-4">
      <h2 className="text-xl font-semibold">Something went wrong</h2>
      <button onClick={reset} className="px-4 py-2 rounded bg-primary text-primary-foreground">
        Try again
      </button>
    </div>
  );
}
```

- Add `not-found.tsx` for custom 404 pages

## Platform-Specific Setup

### Vercel (Next.js default)
```json
// vercel.json (usually not needed)
{
  "framework": "nextjs"
}
```
- Set env vars in Vercel dashboard or `vercel env pull`
- Database: Vercel Postgres, Neon, or Turso (for SQLite)
- Edge functions: middleware runs on edge by default
- Caching: ISR with `revalidate` export, or `revalidatePath()`

### Cloudflare Workers/Pages (Hono)
```toml
# wrangler.toml
name = "my-api"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[vars]
ENVIRONMENT = "production"

[[d1_databases]]
binding = "DB"
database_name = "my-db"
database_id = "xxx"
```

### Railway
```json
// railway.json
{
  "build": { "builder": "NIXPACKS" },
  "deploy": {
    "healthcheckPath": "/api/health",
    "healthcheckTimeout": 30
  }
}
```
- Add PostgreSQL service, link env var automatically
- Set `PORT` env var (Railway assigns it)

### Docker
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

### GitHub Actions CI

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm test
```

## Performance Quick Wins

- Next.js: Enable `output: 'standalone'` in `next.config.js` for Docker
- Images: Use `next/image` with proper `sizes` attribute
- Fonts: Use `next/font` for zero-layout-shift font loading
- Bundle: Run `npx @next/bundle-analyzer` and remove unused imports
- Database: Add indexes on queried columns, enable WAL mode for SQLite

## Rules

- NEVER deploy without checking that `npm run build` succeeds locally.
- NEVER hardcode URLs, secrets, or environment-specific values.
- ALWAYS have a health check endpoint.
- ALWAYS validate env vars at startup — fail fast on misconfiguration.
- Production error pages should be helpful ("Something went wrong" + retry), not technical.
- Verify the deploy works by hitting the health endpoint and testing the main flow.
