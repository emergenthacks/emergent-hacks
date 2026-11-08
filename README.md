# Roomsure

Roomsure helps people compare rooms for short stays, understand the trade-offs, inspect a sample room tour, and check outside listings for warning signs. The hackathon demo focuses on Providence, Rhode Island, and synthetic listing data.

## The problem

People staying for an internship, clinical rotation, visiting term, or short contract often need a room for one to six months. Nightly rentals can be costly, year-long leases are too long, and informal listings are hard to compare and may contain scam signals. Roomsure makes the seeker's constraints explicit and explains each result.

## What works in the current MVP

- **Natural-language intake:** describe dates, budget, commute, destination, and room needs. The backend uses Claude when configured and returns the rule-based result if Claude is unavailable. The UI labels which path answered.
- **Explainable room ranking:** a deterministic TypeScript engine ranks synthetic rooms using the seeker's preferences. Changing priorities updates the deck locally.
- **Swipe, shortlist, and room details:** app state and synthetic room inventory are kept in the browser. On rooms that have one, **Walk the room in 3D** opens a sample scan (the team's kitchen; a bedroom for the Elmhurst dorm), labeled "Sample 3D scan · demo". It is not the listed room.
- **Outside-listing check:** Claude can suggest quoted warning signs; code validates quotes and computes the final verdict. Rules continue to work offline.
- **Demo match and reservation:** host replies and payment outcomes are simulated. No money moves.
- **List your room (demo):** a host flow at `/list-your-room` with simulated host checks, a listing form whose rules flag scam-like wording, and sample-video 3D scans. Published rooms stay in that browser; nothing is saved on the server, and live video-to-scan isn't deployed.

All people, listings, prices, verification badges, and outcomes shown in the demo are synthetic or simulated. Roomsure does not collect real ID or card details, verify ownership, fetch live inventory, guarantee safety, or hold deposits.

## Current deployment

- **Frontend:** [roomsure-hackathon.vercel.app](https://roomsure-hackathon.vercel.app/)
- **Backend API:** [roomsure-api.vercel.app](https://roomsure-api.vercel.app/)
- **Backend health/session endpoint:** [`/api/me`](https://roomsure-api.vercel.app/api/me)
- `GET /api/me` returned HTTP 200 during the latest connectivity check, and a one-time direct backend intake check returned `source: "claude"` and `fallback: null`. The production website is built from `main` of [DivyaPrakash04/emergent-hacks](https://github.com/DivyaPrakash04/emergent-hacks), which has the `/api/*` rewrite in `website/vercel.json`. This repository's `main` lacks that file and the newest website changes; see [Which code is live](ARCHITECTURE.md#which-code-is-live). **Before the demo,** confirm the production address serves the newest deployment: `https://roomsure-hackathon.vercel.app/scans/room3d/rooms/bedroom.json` should return 200, and the Elmhurst dorm room page should open a 3D tour.
- GitHub repository: [DivyaPrakash04/emergent-hacks](https://github.com/DivyaPrakash04/emergent-hacks)

Environment secrets are server-side. In Vercel, `ANTHROPIC_API_KEY` belongs on the `roomsure-api` project, never in `website/` or a `NEXT_PUBLIC_*` variable.

## Repository map

| Path | Role |
|---|---|
| `website/` | TanStack Start / React / TypeScript UI, local listing data, browser state, API client and UI tests. |
| `platform/` | Next.js API backend, session/demo account logic, Claude adapter, rule fallback, and Turso/SQLite data access. |
| `ranking/` | Standalone ranking-engine reference and test fixtures. The active website engine is in `website/src/lib/roomsure/`. |
| `roomwalk/` | RoomWalk 3D viewer and video-to-scan tooling. The website's sample scans live separately in `website/public/scans/room3d/`. |
| `lovable/` | UI design/export notes and prompts. |
| `Roomsure MVP Requirements (for LLM build).md` | Product behavior and acceptance requirements; consult with the code and solution design. |

## Run locally

Use Node 24+ for the backend (it uses `node:sqlite`). Open two terminals.

**Backend**

```sh
cd platform
npm ci
cp .env.example .env.local
npm run dev
```

Set `APP_URL=http://localhost:5173` in `platform/.env.local`; it must match the website origin for POST requests. Add `ANTHROPIC_API_KEY` there only if you want local Claude requests. The backend will use deterministic rule fallbacks when no key is available.

**Website**

```sh
cd website
npm ci
npm run dev -- --port 5173
```

Open `http://localhost:5173`. Vite proxies `/api` to `http://127.0.0.1:3000` by default. Set `VITE_ROOMSURE_BACKEND=off` only when intentionally demonstrating the UI's local fallback.

## Verification commands

```sh
# Backend unit tests and types
cd platform && npm test && npm run typecheck

# Backend end-to-end tests (requires Playwright Chromium installed)
cd platform && npx playwright install chromium && npm run test:e2e

# Frontend tests and production build
cd website && npm test && npm run build

# Ranking reference tests (run with the platform TS runner)
cd platform && npx tsx --test ../ranking/rankListings.test.ts
```

The browser tests set empty Claude credentials and exercise the rules path; they should never spend API credits. See [LLM_REVIEW_PROMPT.md](LLM_REVIEW_PROMPT.md) for a review and verification checklist. Do not run the paid Claude evaluation suite without the team's approval.

## Documents

- [Solution design](SOLUTION_DESIGN.md): user journeys, scope, architecture, API/data boundaries, safety and acceptance criteria.
- [Technical architecture](ARCHITECTURE.md): runtime diagram, deployment, data ownership, and implementation decisions.
- [Three-minute demo script](demo-script.md): timed stage instructions, exact text to use, and recovery plan.
- [Pitch deck prompt](PITCH_DECK_PROMPT.md): brief for another LLM to create the updated three-minute deck and paired video layout. The existing `Roomsure Pitch Deck.pptx` is an older draft and should be refreshed against this brief before presenting.
- [LLM review prompt](LLM_REVIEW_PROMPT.md): task brief for another LLM to verify the current code and refine the demo/docs/deck.
- [MVP requirements](Roomsure%20MVP%20Requirements%20%28for%20LLM%20build%29.md): detailed product requirements.
- [Master build prompt](MASTER_PROMPT.md): general coding-agent context.

## Next demo/integration milestone

Bring this repository's `main` up to date with the fork's `main` (the `/api/*` rewrite, the 3D tour fix, List your room and the sample scans), so both describe the same product. Keep seed rooms available locally so the demo can still browse and rank if the backend is unavailable. Decide whether host listings need backend persistence before promising more than a browser-only demo.
