# Prompt: Roomsure demo verification, refinement, and presentation package

Copy the prompt below into another coding LLM after opening this repository. It asks the agent to independently verify the live product and repository, make only justified low-risk UI fixes, update the project documentation, and produce a concise three-minute pitch/demo deck.

---

You are the implementation/review LLM for **Roomsure**, a hackathon MVP for short-term room rentals. Work in the existing repository and inspect the current branch before changing anything. The docs may be stale; source code, Vercel settings/health endpoints, and tests are the ground truth.

## Live links and repository

- GitHub: https://github.com/DivyaPrakash04/emergent-hacks
- Production frontend: https://roomsure-hackathon.vercel.app/
- Production backend: https://roomsure-api.vercel.app/
- Backend smoke endpoint: https://roomsure-api.vercel.app/api/me
- Main directories: `website/` (TanStack Start frontend), `platform/` (Next.js backend), `ranking/` (engine reference tests), `roomwalk/` (3D viewer/pipeline).

The backend key is configured on the `roomsure-api` Vercel project as `ANTHROPIC_API_KEY`. Never print, request, commit, or expose secret values. Never move the key into the frontend or any `NEXT_PUBLIC_*` variable. The production `/api/intake` was recently checked once and returned `source: "claude"`, `fallback: null`; treat this as a previous observation, not proof that it still works now.

## Product reality: retain these boundaries

- Seeker flow: demo welcome/login → natural-language intake → editable preferences → deterministic ranked swipe/list deck → details → matches/chat demo → simulated reserve. The latest `main` has no bundled RoomWalk scan assets; verify before describing/showing its tour.
- Scam check: pasted text → Claude suggestions (optional) → quotes validated against exact input → deterministic code verdict and fair-price estimate where possible.
- Rooms/listings, prices, commute estimates and people are synthetic demo data.
- Claude handles language extraction/suggestions only. Code computes rankings and final scam verdicts.
- Rules fallback must continue to work when AI is unavailable. UI must disclose whether Claude or rules answered.
- Identity checks, host replies, match/chat, and payment are simulations; no real ID/card data is collected and no money/messages are sent.
- The latest `main` does not include a host listing create/edit/publish workflow. Do not claim host listing extraction or publishing is available unless source code on the branch confirms it.
- The sample 3D tour is not proof of the listing's authenticity and might not correspond to its room photos.

## Task A — verify current state first

1. Check `git status`, branch, local changes, and recent commit history. Preserve unrelated work; do not reset, rebase, force-push, merge, or deploy.
2. Read `README.md`, `ARCHITECTURE.md`, `SOLUTION_DESIGN.md`, `demo-script.md`, product requirements, and the relevant code under `website/` and `platform/`.
3. Verify deployment health without exposing secrets. Check the frontend page and backend `/api/me`. If calling an AI endpoint, do at most one small controlled request and state that it can incur API usage; do not run the full paid evaluation suite without explicit approval.
4. Run existing no-cost checks: website tests/build; backend unit tests/typecheck/build; ranking tests via the existing TypeScript runner; backend browser tests if Playwright Chromium is available. Do not change test data, evaluation graders, or harness gates just to make scores look better.
5. Record command, pass/fail, and root cause for each check. Separate code failures from environment/tooling failures. Do not say “all tests pass” if any check was blocked or failed.
6. Check responsive behavior on desktop and mobile widths. If browser automation is available, inspect the welcome, intake, Discover, room detail, Check and matches flows; inspect 3D/host only if those features exist on the checked branch. Otherwise state the UI checks you could not perform.

## Task B — targeted frontend fixes

Make a frontend change only if you find a concrete, visible defect that is small, clearly safe, and relevant to the three-minute demo (examples: text overflow, inaccessible button label, broken link, misleading demo label, obvious mobile overflow). Keep the visual system and architecture. Avoid broad redesigns, new dependencies, auth/payment rewrites, backend changes, and opportunistic refactors. If no minor fix is justified, leave the code untouched and report that.

Add or adjust a focused regression test for any UI fix, then run the relevant existing test/build checks. Do not change the ranking or AI evaluation harness without the owner's explicit approval.

## Task C — evaluation and demo readiness

- Review existing unit/component/E2E tests and `platform/evals/` cases/grader.
- Add a concise manual acceptance/evaluation checklist if useful. Keep checks reproducible and based on observable outcomes: source badge, preference values, rank/reason update, quoted text, simulated-payment label, sample tour label, mobile layout.
- Do not invent model accuracy, user research, market size, reliability, trust or conversion numbers. Do not run paid model evaluations automatically. If model eval is approved later, report the measured output and API usage honestly.
- Refine `demo-script.md` into a timed **three-minute** path. Include exact UI actions, concise talk track, judging criteria (Functionality, Execution, Usability, Originality), fallback path, and a pre-demo checklist. Keep claims tied to implemented features.

## Task D — documents and architecture

Bring these files into alignment with verified source and deployment state:

1. `README.md`: what Roomsure does and why; current implemented features; setup/run/test instructions; live links; limitations; docs index.
2. `SOLUTION_DESIGN.md`: product goal/personas; scope and out-of-scope items; key user journeys; UX/functional behavior; component/service diagram; data/API contracts; security/privacy/fairness; test/acceptance plan; deploy/work roadmap.
3. `ARCHITECTURE.md`: accurate current split-app design, Mermaid architecture diagram, frontend rewrite, API route responsibilities, database/AI boundaries, environment config, fallback/validation behavior, known limitations. Remove or clearly label old plans that conflict with the code.
4. `demo-script.md`: exactly three minutes maximum, with rehearsal and failure recovery steps.
5. `Roomsure Pitch Deck.pptx`: a concise pitch/demo presentation that can be delivered within three minutes. Align every claim with the live code and mark all demo simulations. Use the product's actual design language where practical.

## Task E — paired desktop/mobile demo video in the deck

There are no desktop/mobile video files supplied unless you find them in the repository. Do not invent or embed fake footage. Create a dedicated split-screen slide with **two clearly labeled video placeholders** (Desktop walkthrough / Mobile walkthrough) and useful capture cues. Tell the owner exactly where/how to insert or replace the clips. Both recordings should show the same build, same intake sentence and corresponding actions, with captions/subtitles; avoid showing secrets, Vercel dashboards, personal data or API keys. Include a static screenshot fallback for presentation-room playback issues. If actual clip files are present, inspect and use them; otherwise leave unmistakable placeholders and report the missing assets.

## Constraints

- Work on a feature branch only if branch operations are available and authorized; never push or merge without explicit instruction.
- Do not add credentials, personal information, or secret values to docs, test output, screenshots, or slides.
- Keep provider/payment/verification claims honest. Clearly label synthetic listings and estimates.
- Keep the final demo short and coherent: intake → explainable reranking → one room/sample scan → scam warning evidence → short close. The host form is optional; don't imply AI extraction exists.
- No real-money or identity flow in a demo.

## Final response

Summarize changed files and meaningful decisions, exact tests/checks with results, any issues left, and the current branch/commit state. Include links to the production frontend/backend and to the deck. State plainly whether desktop/mobile demo videos are actually embedded or still need recording/insertion. Do not claim an external LLM reviewed your own work unless it actually did.

---

Use `demo-script.md` as the timing source after independently confirming its UI actions against the current application.
