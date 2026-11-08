# Roomsure: three-minute demo

**Goal:** show the value in one short journey: Roomsure understands a renter's needs, explains a ranked choice, and points to evidence in a suspicious listing. Keep every simulation visible and honest.

## Before the demo

- Open the public production site: <https://roomsure-hackathon.vercel.app/>.
- Confirm the page is public, then click **Continue as Maya (demo)** and use **Reset demo** from the Passport if a reservation was made in an earlier rehearsal.
- Have the demo sentence and suspicious listing below copied to the clipboard.
- The current `main` build shows a placeholder for the 3D tour because scan assets are absent. Skip this screen in the timed demo unless assets are restored and verified.
- Confirm the intake source badge says **Read by Claude**. If it says **Read by rules (offline)**, proceed and explain the fallback; do not wait on stage for AI.
- Set browser zoom so text fits in the viewport. Silence notifications and keep the mobile device ready if recording a split-screen clip.
- Record both views against the same build and same demo sentence. The desktop recording shows the complete UI; the mobile recording shows the responsive layout. Use the deck's paired video slots in [Roomsure Pitch Deck.pptx](Roomsure%20Pitch%20Deck.pptx). No video assets are currently included in this repository.

## Timed run of show

| Time | Action | Presenter line | Judging signal |
|---|---|---|---|
| **0:00–0:20** | Show welcome. Click **Continue as Maya (demo)**. | “Short stays fall between nightly rentals and year-long leases. Informal sublets are hard to compare and can be risky. Roomsure makes the trade-offs visible.” | Problem clarity, usability |
| **0:20–0:55** | On **What do you need?**, paste the sample sentence and click **Find my rooms**. Show the source badge and editable preferences. Click **Show my rooms**. | “Maya describes her internship stay in one sentence. Claude extracts dates, budget, commute and room needs. She can review everything before the app uses it. If Claude is unavailable, the rules path still works.” | Functionality, originality, execution |
| **0:55–1:35** | In **Discover**, show rank and reason on the first card. Open **Adjust** and increase commute importance or change the priority, then show the deck reorder. Optionally show the room detail; skip the unavailable 3D scan placeholder. | “The order is calculated by deterministic code from Maya's priorities. Every card explains its trade-off.” | Functionality, execution, usability |
| **1:35–2:20** | Go to **Check**. Select **Too good to be true**, click **Check this listing**. Point to the high-risk result and exact quoted phrases. | “For a listing from anywhere, Roomsure shows the evidence: payment pressure, inability to view, and urgency. The quotes are checked against the text, and code decides the verdict. It flags risk; it does not certify safety.” | Originality, execution, trust |
| **2:20–2:45** | Return to Discover. Swipe right / press **Interested** on a room. If a match overlay appears, show it briefly; otherwise go to Matches. Optionally show the simulated reservation label only if time. | “A positive swipe starts a simulated host response. Chat and reservation are demo interactions; no messages are sent and no money moves.” | Functionality, honest execution |
| **2:45–3:00** | Close on the product headline or a clean slide. | “Roomsure turns a stressful search into a choice you can understand: your criteria, clear trade-offs, and specific warning evidence.” | Clear close |

### Intake sentence

> I'm interning downtown June 1 to August 15. Budget around $900, $1,050 max. Commute ideally 20 minutes, 35 at most. I need a kitchen, furnished matters a lot, and I can live with up to 5 days' gap.

### Scam-check sample

Use the **Too good to be true** preset in the Check screen. It includes a low price, an owner overseas who cannot show the room, a request to wire a deposit, and urgency. This avoids typing errors on stage.

## Rehearsal checklist

- [ ] Reset demo and rehearse the entire sequence with a stopwatch; finish in 2:45 to leave room for pauses.
- [ ] Confirm ranking changes visibly after a priority change. If the new top card does not change, use the app's supplied demo preferences and rehearse a different priority control.
- [ ] Confirm both API-assisted screens show the source badge. Do not claim Claude answered unless the badge says so.
- [ ] Confirm the scam result displays exact quotes and High risk.
- [ ] Skip the 3D tour unless scan assets have been restored and verified on the deployed build.
- [ ] Confirm reserve UI says **Simulated payment** / **No money moves in this demo**.
- [ ] Record a full desktop walkthrough and matching mobile walkthrough as backup media.
- [ ] Test the video clips in the final slide deck on the presentation laptop; retain a static screenshot fallback in case video playback fails.

## Recovery lines

| Issue | Recovery |
|---|---|
| Claude request is slow or falls back | Continue with the rules result. Say, “The rules fallback keeps the demo usable; this badge shows which path answered.” |
| Backend is unavailable | Use the frontend's local fallback. Be clear that the live backend is temporarily unavailable; don't call the local mock a backend response. |
| 3D scan fails | Close the overlay and continue. Say, “The scan is a sample and the viewer is an inspection prototype.” |
| Match does not appear in the timed window | Skip the match and move to the close; the core demo is intake, explainable ranking, and evidence-based scam checking. |
| Live demo fails | Play the recorded desktop/mobile clip. Keep a static screenshot and this script as a backup. |

## Demo truth labels

- Synthetic inventory and estimated room prices/commutes.
- Simulated profile checks, host replies, mutual match, chat and payment.
- The latest `main` does not include a host listing workflow or scan assets; do not present these as working demo features.
- The 3D viewer is a prototype and is not proof of a listing's authenticity.
- Scam check identifies textual warning signals; it cannot guarantee whether a person or listing is legitimate.
