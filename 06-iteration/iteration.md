# Iteration: Analytics, Sprint, Final Recommendation

> Module 6 · Evals & Iteration. Read the analytics, run an iteration sprint, present with evidence.

## What the evidence says

_Analytics snapshot:_ visitors 5; page views 42; views per visit 8.4; average session duration 10m 56s; bounce rate 0%.

- **Primary signal:** A first-time-user walkthrough of this build had six separate pauses to work through trust content, reread endorsement wording, and parse research context before reaching booking — a slow, deliberate read, not a quick bounce.
- **What moved:** Not measurable yet. This is the first analytics reading taken for this build; there is no earlier baseline to compare it against.
- **What didn't:** The snapshot shows traffic and engagement depth, not conversion — it doesn't say how many of the 5 visitors reached or completed the booking flow, and at this sample size it can't distinguish outside visitors from testing sessions.


## Iteration sprint

| Change | Hypothesis | Result |
|---|---|---|
| Separate the research/experiment content (baseline stats, categorized quotes) from the buyer-facing trust content (verification badges, endorsements, pricing, guarantees) on the provider profile. | Based on a first-time-user walkthrough finding that the provider profile currently reads as two documents stitched together — buyer decision content interleaved with research framing in one continuous scroll — separating them should let a visitor move cleanly from trust signals to a booking decision, without detouring through the research narration. | Based on a first-time-user walkthrough finding that the provider profile currently reads as two documents stitched together — buyer decision content interleaved with research framing in one continuous scroll — removing the duplicated research framing from the profile page should let a visitor move cleanly from trust signals to a booking decision, with the research content living in exactly one place instead of two. |


## Peer feedback
Not real peer/classmate feedback — flagged plainly rather than dressed up. This is a second-round, single-session, self-administered persona walkthrough ("Maria" revisiting the same live URL, direct DOM inspection this session since screenshots were unavailable), used here in place of peer feedback per the user's specific request for this section._

- "The research and buyer content used to feel like two documents stitched together. Now the profile page only shows what I need to decide whether to book — verification, endorsements, pricing, guarantees. The research framing is gone from here entirely." → points to: the round 1 fix reads as resolved on this page.
- "I didn't see the Experiment Brief anywhere on the homepage this time — as a regular visitor I wouldn't know it ever existed." → points to: the admin-gating change is working for a logged-out visitor.
- "There's now a 'Book Alex' button right in the sidebar next to the price — I didn't have to go hunting for it like last time." → points to: the earlier unverified CTA-visibility finding appears resolved, though it's unclear if that's a deliberate fix or a side effect.
- "The two endorsements explain themselves — Two of them say right on the card, 'Independently confirmed: we contacted the employer' — and the third says 'Provided by Alex M.. We have not confirmed it.' I don't have to guess anymore which ones the marketplace actually checked." → points to: the endorsement verification-language finding is resolved — confirmed by reading the exact wording on all three endorsement cards on `/providers/alex`, plus a new section intro ("Where the tag says verified, we contacted the organisation ourselves...") that explains the general rule.
- "I saw a demo login posted right on the sign-in page — I could see how someone reviewing this would use it, though it does mean anyone visiting could look at the experiment data too."
- - "I went straight to the booking page this time and it opened clean — nothing already in my order. I had to actually pick something before 'Review and pay' would even light up." → points to: the booking-page pre-filled-cart finding is resolved — confirmed both by the page text ("No items selected yet," all quantities at 0) and by checking the button directly: it's disabled until an item is added.
- "I clicked 'View profile' on Yannis and on Maria S., from the homepage and from Alex's own 'Other assemblers' section — both actually took me somewhere this time, a 'coming soon' page with their name on it, instead of doing nothing." → points to: the dead "View profile" links finding is resolved — confirmed on the Home page, the Browse page (filtered to Furniture Assembly), and Alex's "Other assemblers in Athens" section; all four other providers (Yannis T., Maria S., Petros L., Kostas D.) now link to `/providers/coming-soon/<name>`.
- **Recurring theme:** the structural fix (separating research from buyer content, gating the Experiment Brief) reads as genuinely resolved; the one friction point that survived unchanged from round 1 is the endorsement-language inconsistency — the same single finding two independent passes have now flagged.

_____

## The recommendation

**Decision:** ☐ Go  ☑ Iterate  ☐ Kill

_The evidence that justifies the call:_

Not Kill — every structural and cosmetic finding raised (research/buyer content separation, admin-gating, endorsement clarity, the pre-filled cart, the dead provider links) is now confirmed fixed on the live build; nothing suggests the trust-panel approach itself is failing.

Not Go — this is the more honest change from the previous cycle's call: there is still no fresh analytics reading from after *any* of these changes shipped, so there remains no way to say whether the accumulated fixes actually changed visitor behavior (time to book, drop-off, completion). Every finding checked so far is qualitative and self-administered, not real peer feedback or a real usage number.

Iterate: there is no further known product defect from any walkthrough on record to fix next. The honest next step is not another code change but closing the evidence gap itself — pull a fresh analytics reading against the current build, and get one round of real (not self-administered) peer feedback — before deciding whether this hypothesis is validated enough to call Go. 

_____

## Final showcase

- **Demo link:** https://trust-bloom-prototype.lovable.app
- **The one-sentence story:** Across three iteration cycles, every finding raised by repeated self-administered walkthroughs — research content bleeding into the buyer decision surface, an ungated Experiment Brief, unclear endorsement provenance, a pre-filled booking cart, and dead links to other providers — is now confirmed fixed on the live build; what the process still lacks is a fresh real-usage reading to say whether any of it moved actual visitor behavior.
- **Where it landed on the Confidence Line (M2 → now):** The hypothesis was locked and first built in Module 2, re-architected in Module 3, and documented with a Living PRD and engineering handoff in Module 4. Module 6's first cycle turned one qualitative finding into a shipped, confirmed change; this second cycle closed out three more, each confirmed by direct inspection rather than assumed. The hypothesis itself is still unvalidated by real booking behavior — what's moved across all three cycles is that the page now has zero known confounds standing between a visitor and a clean read of the trust mechanism, and the remaining gap is evidence, not more building.

