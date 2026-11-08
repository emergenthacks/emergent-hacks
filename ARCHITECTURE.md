# Roomsure technical architecture

**Status:** current hackathon MVP architecture  
**Last reviewed:** 2026-11-08  
**Related:** [README](README.md) · [Solution design](SOLUTION_DESIGN.md) · [MVP requirements](Roomsure%20MVP%20Requirements%20%28for%20LLM%20build%29.md) · [Demo script](demo-script.md)

This document describes the code behind the live demo as inspected on 2026-11-08. The product has two deployable apps: `website/` is the browser experience and `platform/` is the API. The browser client calls same-origin `/api/*` paths.

### Which code is live

The production website (`https://roomsure-hackathon.vercel.app`) and API are built by Vercel from `main` of the fork [DivyaPrakash04/emergent-hacks](https://github.com/DivyaPrakash04/emergent-hacks). This repository's `main` is behind that branch. Where they differ:

| Feature | Fork `main` (what production builds) | This repository's `main` |
|---|---|---|
| `/api/*` rewrite in `website/vercel.json` | Present | Missing, so website requests fall back to local mocks |
| 3D tour on room pages | Opens sample scans: Fox Point, Downcity and College Hill show the team's kitchen; the Elmhurst dorm shows a bedroom | Shows a placeholder. The scan files are in `website/public/scans/room3d/`, but the demo rooms point at `/scans/<room>` paths that don't exist and the overlay only loads absolute URLs |
| List your room | In-app flow at `/list-your-room`: browser-only, simulated host checks, sample-video scans | A static demo page, `website/public/host/index.html`, linked from the welcome page |
| Backend 5xx falls back to local rules | Yes | Yes |

Before a demo, confirm the production address serves the newest deployment of the fork's `main`, not an older one: `https://roomsure-hackathon.vercel.app/scans/room3d/rooms/bedroom.json` should return 200, and the Elmhurst dorm room page should open a 3D tour.

## 1. System diagram

```mermaid
flowchart LR
  Person[Seeker / host / judge]
  subgraph Browser[Browser: Roomsure website]
    UI[TanStack Start + React UI]
    State[Browser state and localStorage]
    Inventory[Synthetic seed rooms]
    Rank[Deterministic ranking and fair-price rules]
    Tour[3D tour overlay: sample scans]
    APIClient[Typed API client with explicit mock fallback]
    UI <--> State
    UI --> Inventory
    UI --> Rank
    UI --> Tour
    UI --> APIClient
  end
  subgraph VercelFrontend[Vercel project: roomsure]
    Rewrite[/api/* rewrite in website/vercel.json]
  end
  subgraph VercelBackend[Vercel project: roomsure-api]
    Router[Next.js API catch-all]
    Sessions[Demo session and account routes]
    AI[Server-only Claude adapter]
    Rules[Deterministic rule fallbacks and validation]
    DB[(Turso SQLite-compatible database)]
    Router --> Sessions
    Router --> AI
    Router --> Rules
    Router --> DB
  end
  Claude[Anthropic Claude API]
  Person --> UI
  APIClient -.->|intended same-origin /api/*| Rewrite
  Rewrite --> Router
  AI -->|validated structured output| Claude
```

**Browser requests:** `website/src/lib/roomsure/api.ts` is the screen-facing API. `backend.ts` handles JSON requests and response mapping. `withFallback()` uses local mocks when the backend is unreachable, explicitly disabled, answers with something other than JSON, or returns a 5xx server error. A 4xx error from a reachable backend (validation, rate limit, duplicate reservation, origin check) is surfaced rather than hidden by a mock.

**Production routing:** The required rewrite is `/api/:path*` → `https://roomsure-api.vercel.app/api/:path*`. It lives in `website/vercel.json` on the fork's `main`, which production builds. This repository's `main` doesn't have the file; add it before deploying from here. Without a proxy, the browser's API client falls back locally on an HTML/non-JSON response. When restored, the rewrite keeps session cookies on the website origin from the browser's perspective. Backend `APP_URL` must exactly match the public frontend origin because POST routes enforce an origin check.

## 2. Deployable apps

| App | Source | Framework | Vercel settings | Responsibility |
|---|---|---|---|---|
| Website | `website/` | TanStack Start, React, Vite, TypeScript | Project `roomsure`, root directory `website/` | Screens, browser session state, local demo inventory, matching, swipe/match UI and RoomWalk viewer. |
| API | `platform/` | Next.js App Router, TypeScript, Node runtime | Project `roomsure-api`, root directory `platform/` | Claude requests, origin checks, demo session/account routes, simulated reservation state and database adapter. |

The projects are connected to GitHub and deploy when their tracked branch changes. Preview and production environment variables are managed separately in Vercel. `ANTHROPIC_API_KEY` is set only in the API project; never use a `NEXT_PUBLIC_` prefix for secrets.

## 3. Request flows

### Natural-language intake

1. `StartScreen` sends the user's sentence to `api.parseIntake()`.
2. The browser calls `POST /api/intake` with `{ "text": "..." }`.
3. The backend validates the request, applies a rate limit, and calls `readIntake()`.
4. The Claude adapter requests structured output, validates it with Zod, drops invalid values and protected-trait content, and converts it to typed preferences.
5. If the key is absent or the call times out, is refused, or returns invalid output, deterministic intake rules run. The response includes `source` and a fallback marker; the UI labels Claude vs rules.
6. The user can edit extracted preferences. `buildDeck()` reranks locally; changing priorities does not make a network request.

### Scam check

1. `CheckScreen` sends pasted text to `POST /api/check`.
2. Code always runs its deterministic signals and fair-price estimate.
3. Claude may suggest additional signals with exact evidence. The backend only accepts known signal IDs and quotes that appear verbatim in the submitted text.
4. Code computes the final verdict. Claude does not decide whether a listing is safe, and the product does not certify outside listings.

### Demo account and simulated reservation

1. The frontend calls `GET /api/me`; when there is no session it calls `POST /api/demo/login` and retries `GET /api/me`.
2. The backend issues an HTTP-only session cookie and stores demo account/session/payment state in the configured database.
3. Demo host replies and matches are UI simulations. A reservation flow uses server-side room IDs/prices and simulated completion; it does not move money.
4. `POST /api/demo/reset` clears the demo reservation/activity state for the current session.

### Host listing (current boundary)

**List your room** on the production build (`/list-your-room`, fork `main`) is a browser-only demo. A demo host signs in, completes simulated checks (email, ID and selfie, card) and a right-to-rent question, then fills in a listing form. Code checks the form's rules and flags scam-like wording; no AI reads the form. A published room is saved in that browser's localStorage and joins only that browser's deck.

To add a 3D tour, the host can upload one of two bundled sample videos (kitchen or bathroom). After a short staged progress bar, that room's pre-built scan is attached. Any other video needs the local scan service in `website/public/scans/room3d/scan-pipeline/`, which isn't deployed, so the page says live scanning isn't available.

There are no host-listing API routes or database tables. Do not describe host listings as shared, persisted across devices, or Claude-extracted. This repository's `main` has only the static demo page `website/public/host/index.html`.

## 4. Data ownership

| Data | Current source / storage | Persistent across devices? |
|---|---|---|
| Seeker demo inventory and listing details | Synthetic TypeScript seed data in `website/src/lib/roomsure/data/listings.ts` | Yes as code; not live inventory. |
| Ranking preferences, swipes, match state | Client state and browser localStorage | No; browser/device scoped. |
| Host profile, draft and published room | Browser localStorage | No; local demo only. |
| Demo users, sessions, payment/activity records | `platform/lib/db.ts`; Turso in hosted setup, SQLite for local development | Yes while Turso is configured; demo data only. |
| Claude API key | Vercel environment variable on `roomsure-api` | Secret; server execution only. |
| Sample 3D scans | Static files in `website/public/scans/room3d/` (rooms as JSON), shown in `TourOverlay` | Yes, as static files: the same sample for every visitor, not the listed room. |

Current Turso configuration: `TURSO_DATABASE_URL`, `TURSO_AUTH_TOKEN`, and `TURSO_REQUIRED=true`. Local SQLite is the development fallback. Do not use ephemeral Vercel filesystem storage as a production database.

## 5. API surface

The implementation uses `platform/app/api/[...path]/route.ts` as its API router. Every request runs the explicit provider-mode guard and initializes the demo account/schema. POST routes except the Stripe webhook require an `Origin` equal to `APP_URL`.

| Method and path | Auth | Purpose |
|---|---|---|
| `GET /api/me` | Optional | Returns current demo-safe user snapshot, mode, payment and recent activity. |
| `POST /api/demo/login` | None | Creates the demo seeker session. Demo mode only. |
| `POST /api/demo/reset` | Session | Clears simulated payment/activity state. |
| `POST /api/intake` | None | Text to typed preferences via Claude or rules. |
| `POST /api/check` | None | Listing text to facts, quoted signals and code-computed verdict. |
| `POST /api/payment/start` | Session / eligibility rules | Begins a provider or demo payment at a server-derived amount. |
| `POST /api/demo/complete` | Session | Demo-only simulated payment/identity completion. |
| `POST /api/auth/signup`, `/api/auth/login`, `/api/auth/logout` | Varies | Account/session flows. |
| `POST /api/email/send`, `/api/email/verify`, `/api/identity/start`, `/api/card/start` | Session | Demo/provider-backed onboarding routes; demo outcomes are simulated. |
| `POST /api/host/consent`, `/api/security/password` | Session | Host self-attestation and password change. |
| `POST /api/webhook` | Stripe signature | Provider webhook when a test provider is deliberately configured. |

Avoid adding routes for UI-only features. If host listings become a team milestone, define listing ownership, validation, visibility, duplicate publish behavior, edit/delete semantics, and image storage before implementation.

## 6. Ranking and trust rules

- Active website ranking code is in `website/src/lib/roomsure/rankListings.ts` and `deck.ts`; tests live next to it in `website/src/lib/roomsure/`.
- The ranking engine is deterministic TypeScript. AI never chooses a room score or the final scam verdict.
- Synthetic listing prices, fair-price baselines and commute estimates must be labeled as demo estimates. Do not present them as verified market data.
- Trust labels and identity/payment outcomes in the demo are simulations. A simulated badge is not real-world verification.
- Matching must not use protected traits. Treat listing text and AI output as untrusted; validate every server boundary.

## 7. AI service boundaries

- Adapter: `platform/lib/ai/claude.ts`; schemas: `platform/lib/ai/schemas.ts`.
- SDK client is created only on the server. `ANTHROPIC_API_KEY` and optional `ANTHROPIC_MODEL` are read from the API environment.
- The current adapter uses `claude-opus-5-5` by default, structured output, low effort, and an 8-second timeout.
- AI is optional. On missing key, timeout, refusal, connection/API error, or invalid schema, the routes return a deterministic rules result with a source label.
- Do not log API keys, expose raw prompts/responses to browsers, or add a client-side Anthropic SDK.
- Live calls spend Anthropic credits. Unit/E2E tests use fakes or empty keys; the paid evaluation harness must only be run with explicit team approval.

## 8. Environment and deployment configuration

### `roomsure-api`

| Variable | Required | Purpose |
|---|---:|---|
| `APP_URL` | Yes | Exact public frontend origin, e.g. `https://roomsure-hackathon.vercel.app`. |
| `PROVIDER_MODE=demo` | Yes | Keeps payment/verification outcomes simulated. |
| `TURSO_DATABASE_URL` | Hosted | Turso database URL. |
| `TURSO_AUTH_TOKEN` | Hosted | Secret Turso token. |
| `TURSO_REQUIRED=true` | Hosted | Prevents silent use of local SQLite if Turso is missing. |
| `ANTHROPIC_API_KEY` | Optional | Secret Claude API credential; server-side only. |
| `ANTHROPIC_MODEL` | Optional | Model override; otherwise adapter default. |

### `roomsure`

- `website/vercel.json` owns the production API rewrite. It is on the fork's `main` but missing from this repository's `main`. Verify `https://roomsure-hackathon.vercel.app/api/me` before the demo.
- Confirm the production address serves the newest deployment of the fork's `main`: `https://roomsure-hackathon.vercel.app/scans/room3d/rooms/bedroom.json` should return 200, and the Elmhurst dorm room page should open a 3D tour.
- Browser build must not contain Claude, Turso, Stripe, or Resend secrets.
- For preview environments, verify the preview frontend/backend URL and the API's `APP_URL`/origin policy before expecting POST requests to pass.

### Local development

- Backend: from `platform/`, set `APP_URL=http://localhost:5173` and run `npm run dev` on port 3000.
- Website: from `website/`, run `npm run dev -- --port 5173`; Vite proxies `/api` to port 3000.
- Backend requires Node 24+ for the built-in SQLite driver.

## 9. Validation and evaluation

- Backend unit tests: `cd platform && npm test`.
- Backend browser tests: `cd platform && npm run test:e2e` (install Chromium first).
- Website unit/component tests: `cd website && npm test`.
- Type-check/build: `cd platform && npm run typecheck && npm run build`; `cd website && npm run build`.
- Ranking reference: `cd platform && npx tsx --test ../ranking/rankListings.test.ts`.
- AI quality evaluation: fixed cases and plain-code graders in `platform/evals/`. It makes real Anthropic calls; never run it automatically as part of a demo or CI build.
- End-to-end smoke check: open production website; submit the demo intake and verify the source badge; change ranking weights; submit the scam sample and verify quoted evidence; open a sample 3D tour; exercise mock match/reservation; confirm the simulated label.

## 10. Known limitations and next work

1. Host listing CRUD/persistence and image upload are not implemented; listings created by hosts remain browser-local.
2. Inventory, room prices, fair-price baselines and commute estimates are synthetic, not live or audited market data.
3. ID/card/email checks, host replies, chat and payment in the presented demo are simulated. No escrow or deposit protection exists.
4. Swipes and preferences are browser-local and do not train or personalize a model across users.
5. 3D tours are samples: Fox Point, Downcity and College Hill open the team's kitchen scan, and the Elmhurst dorm opens a bedroom. They aren't the listed rooms and prove nothing about a listing. Live video-to-scan needs a local scan service that isn't deployed. On this repository's `main`, the room page still shows a placeholder.
6. Before real users: add real identity/payment provider configuration, legal/privacy/data-retention review, robust shared rate limiting, support and dispute handling, security monitoring, and trustworthy listing/host inventory.
