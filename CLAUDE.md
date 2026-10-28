# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
EventHub is a full-stack event ticket booking platform built for QA training. Users can browse events, book tickets, manage bookings, and create events. Each user operates in an isolated sandbox: user-created events and bookings are scoped by `userId`, while seeded events (`isStatic = true`) are shared and immutable.

## Tech Stack
- **Frontend**: Next.js 14 (App Router), React 18, TypeScript, Tailwind CSS, React Query v5
- **Backend**: Express.js, Prisma 5 ORM, MySQL 8+, express-validator, Swagger UI (`/api/docs`)
- **Auth**: JWT (7-day expiry), bcryptjs
- **Testing**: Playwright E2E (Chromium only)

## Commands
```bash
npm run setup        # Install deps in backend/ and frontend/
npm run dev          # Start backend (3001, nodemon) + frontend (3000) concurrently
npm run db:push      # Push Prisma schema to DB (non-interactive)
npm run migrate      # prisma migrate dev (interactive)
npm run seed         # Seed 10 static events
npm run build        # Build the Next.js frontend
npm run lint         # Lint the frontend (next lint)
npm run test         # Run all Playwright tests
npm run test:ui      # Playwright UI mode
npm run test:report  # Open last HTML report
npx playwright test tests/<file>.spec.js --reporter=line   # Single file
npx playwright test -g "<test title>" --reporter=line      # Single test by title
docker compose run --rm tests                              # Run tests in the Playwright Docker image
```

Env files: `backend/.env` (`DATABASE_URL`, `PORT`, `NODE_ENV`, `CORS_ORIGIN`, JWT secret) and `frontend/.env.local` (`NEXT_PUBLIC_API_URL=http://localhost:3001/api`).

**Important:** `playwright.config.ts` sets `baseURL` to the deployed site `https://eventhub.rahulshettyacademy.com`, not localhost. Tests run against production by default (also in CI and Docker); change `baseURL` to `http://localhost:3000` to test local changes. Config: 30s test timeout, 5s expect timeout, no retries, not fully parallel.

## Architecture
Backend (`backend/`) is layered: Routes → Controllers → Services → Repositories → Prisma.
- `app.js` wires CORS, routes, Swagger; `server.js` connects the DB and starts HTTP.
- Routes carry the Swagger JSDoc and attach `validators/` (express-validator) and `middleware/authMiddleware.js` (Bearer JWT → `req.user`).
- Services hold business logic and transactions (atomic seat decrement/restore, FIFO pruning, ownership checks). Domain errors live in `src/utils/errors.js` and are mapped to HTTP responses by `middleware/errorHandler.js` — throw these from services rather than building responses.
- Prisma models: `User`, `Event` (`isStatic`, nullable `userId`), `Booking` (`userId`), with cascading deletes from User.

Frontend (`frontend/`):
- API access via an axios client in `lib/api/client.js` that reads the JWT from `localStorage['eventhub_token']` and clears it on 401. Note `lib/api/` contains both `.js` (`authApi`, `eventsApi`, `bookingsApi`) and `.ts` (`events`, `bookings`, `index.ts`) clients — check which one a page imports before editing.
- Data fetching through React Query hooks in `lib/hooks/` (`useAuth`, `useEvents`, `useBookings`); providers in `lib/providers.jsx`.

## Key Business Rules
- Max 6 user-created events (FIFO pruning on overflow)
- Max 9 bookings per user (FIFO pruning on overflow)
- Booking ref first character = event title first character (uppercase)
- Seat count reduces on booking, restores on cancellation
- Refund eligibility: 1 ticket = eligible, >1 ticket = not eligible (client-side)
- Cross-user booking access returns "Access Denied"
- Static events (seeded) are immutable

## Testing Conventions
- Test files go in `tests/` as `<feature-name>.spec.js` (JavaScript)
- Follow the `playwright-best-practices` skill
- Locator priority: data-testid > role > label/placeholder > ID > CSS class
- No `page.waitForTimeout()` — use `expect().toBeVisible()`
- Tests must be self-contained (login → action → assert); add step comments
- Test account: `rahulshetty1@gmail.com` / `Magiclife1!`

## Claude Skills (`.claude/skills/`)
- `eventhub-domain` — business rules, API reference, user flows, UI selectors (see its sub-files)
- `playwright-best-practices` — Playwright testing standards
- `generate-tests <feature>` — generates Playwright tests
- `review-tests <file>` — reviews test code quality
- `create-scenarios <area>` — creates test scenario documents (output in `docs/test-scenarios.md`)
- `test-strategy <scenarios>` — assigns tests to pyramid layers (output in `docs/test-strategy.md`)

## CI (`.github/workflows/`)
- `ci.yml` (PRs) — backend checks (JS syntax, Prisma validate/format/generate), schema drift check against the production DB, frontend typecheck + production build
- `playwright.yml` — runs the Playwright suite against the live site on push to `main`, uploads the HTML report
- `deploy.yml` — production deploy after pre-deploy checks

## Code Style
- Backend: JavaScript with JSDoc, Express patterns
- Frontend: TypeScript (some legacy `.jsx`/`.js` files), React hooks, Tailwind utility classes
