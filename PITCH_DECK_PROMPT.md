# Prompt for the Roomsure three-minute pitch deck

Copy the prompt below into your slide-making LLM and provide it access to this repository. It can read `Roomsure Pitch Deck.pptx` as visual/source material, but must verify every statement against the current code and docs before reusing it.

---

Create a concise, polished **Roomsure hackathon pitch deck for a total three-minute presentation and product demo**. Work from the current repository and use the existing Roomsure visual identity. Output an editable PowerPoint (`Roomsure Pitch Deck.pptx`) plus any source assets needed to reproduce it.

## Product context

Roomsure helps people compare rooms for one-to-six-month stays. A renter describes their needs, reviews extracted preferences, sees a ranked set of rooms with reasons, and can check an outside listing for quoted warning signals.

## Ground truth to verify before writing slides

Read these sources first:

- `README.md`
- `SOLUTION_DESIGN.md`
- `ARCHITECTURE.md`
- `demo-script.md`
- `Roomsure MVP Requirements (for LLM build).md`
- Current app code under `website/` and `platform/`

Live deployment links:

- Frontend: https://roomsure-hackathon.vercel.app/
- Backend: https://roomsure-api.vercel.app/
- GitHub: https://github.com/DivyaPrakash04/emergent-hacks

Do not assume that older slide claims remain true. In particular, don't claim the following as real production capabilities:

- Real ID, email, or card verification
- Verified ownership or a guarantee that a room/host is safe
- Real chat, a confirmed host match, money movement, escrow, deposit protection, refunds, or payouts
- Live listing inventory, audited market-price data, or live commute estimates
- Cross-device persistence for host-created rooms
- AI listing extraction on the host form

The product currently uses synthetic rooms/prices/commute data. The backend uses Claude for intake and scam-signal suggestions when configured; deterministic rules are the fallback. TypeScript code computes ranking and the final scam-check verdict. Chat, host response, identity state and reservation outcomes shown in the demo are simulated. The sample 3D viewer is an inspection prototype and may not depict the card's actual room.

## Main story

Use one renter scenario: a Providence summer intern needs a furnished room from June 1 to August 15, wants to spend about $900 and can go to $1,050, prefers a 20-minute commute and can accept up to 35 minutes.

Explain the product in this order:

1. The short-stay renter has to combine nightly rentals, long leases, and informal posts.
2. Roomsure reads one plain-language request and creates preferences the renter can edit.
3. A deterministic, explainable engine ranks demo rooms according to the renter's priorities.
4. A scam check flags warning signals and quotes the evidence from pasted text.
5. The room tour helps inspect a sample; it is not proof of authenticity.

## Three-minute pacing

Build a deck that supports this exact total runtime (including the product demo):

- **0:00–0:20 — Hook/problem:** short stays fall between nightly stays and year-long leases; informal sublets can be hard to compare.
- **0:20–0:55 — Intake:** enter the supplied sentence; show the `Read by Claude` or `Read by rules (offline)` source badge and editable preferences.
- **0:55–1:35 — Ranking:** show an explainable room card, change a preference/priority and show the deck reorder; optionally open one sample 3D tour briefly.
- **1:35–2:20 — Scam check:** use the `Too good to be true` sample; point out the exact warning quotes and code-computed verdict.
- **2:20–2:45 — Outcome:** show a simulated interested/match response if time permits. Do not imply real chat or payment.
- **2:45–3:00 — Close:** restate that Roomsure gives renters criteria, trade-offs and warning evidence in one flow.

Use about five slides, with very little text. Give the presenter a concise speaker note/talk track and visible timing cues. Prefer product UI visuals over generic stock photos. Do not add unsupported market-size numbers, survey claims, success rates, budgets, investment asks, testimonials, or competitive superlatives.

## Side-by-side desktop/mobile demo video slide

Include a dedicated slide with two synchronized, side-by-side panels:

- **Desktop walkthrough** (wide landscape video frame)
- **Mobile walkthrough** (portrait phone-shaped video frame)

Both clips should show the same production build and corresponding actions: intake, live reranking, then scam-check evidence. Add short captions/subtitles, keep them visually readable, and align timestamps so the audience can compare responsive layouts. Don't put a small mobile screen in a corner where it cannot be read.

No video asset has been supplied in the current task context. Search the repository for actual recordings. If none exist, create clear video placeholders with capture cues and tell the owner exactly where/how to replace them; do not fabricate footage or imply the placeholder is a real recording. Include a static screenshot fallback for rooms where PowerPoint video playback fails. If clips are added, do not show API keys, Vercel settings, personal data, or browser secrets.

## Judging criteria

Make the four criteria legible in a slide or small footer and tie each to a visible demo action:

- **Functionality:** the intake/check flow returns usable results and seed rooms appear.
- **Execution:** explainable deterministic ranking; validated quote evidence; explicit fallback; working frontend/backend connection.
- **Usability:** one-sentence intake, editable preferences, readable reasons, responsive desktop/mobile flow.
- **Originality:** renter-controlled trade-offs, near misses, and evidence-first scam review in one short-stay journey.

Don't imply the product is complete or production-ready. A concise next step may be: pilot one focused supply community, persist host listings, then validate trust flows with users.

## Deliverables and final review

1. Editable PowerPoint, 16:9, around five slides, three-minute total runtime.
2. Side-by-side demo-video layout, with real clips only if available; otherwise obvious placeholders.
3. Static fallback image or guidance for fallback playback.
4. Speaker notes or a separate short script that matches the three-minute timing.
5. A one-paragraph list of sources/files and any missing assets or unverified assumptions.

Check text size, contrast, spelling, slide overflow, and media playback. Rehearse the slide timing against `demo-script.md`. Do not alter app code, commit, merge, or deploy as part of creating the deck unless separately asked.

---
