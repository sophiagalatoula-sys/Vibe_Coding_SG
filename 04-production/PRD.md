# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

The Marketplace Trust Problem: nearly half of active provider profiles on the marketplace have zero reviews, and buyers treat "no reviews" as "not trustworthy."

Supplied baseline evidence (zero-review providers, last 90 days — supplied research data, not live analytics):

| Metric | Value |
|---|---|
| Active provider profiles with 0 reviews | 41% |
| Booking rate, zero-review providers | 2.3% (vs **14%** for reviewed providers — a 6x gap on the same searches) |
| Search → profile → exit without booking | 68% |
| Median time to a new provider's first booking | 19 days |

Buyers reach a profile, find nothing to trust, and leave; new providers churn before they ever earn a review.

Hypothesis under test: a trust panel, verification badges, and early social proof on new-provider profiles will increase first bookings for zero-review providers — measured by first-booking rate rising above the 2.3% baseline. Treatment is the trust-forward profile; control is the standard zero-review profile.


Kill switch: "If trust signals do not increase first bookings, trust is not the barrier — pivot required."

## Users & jobs

- **Primary user:** the buyer hiring a home-services provider (Furniture Assembly category, Athens) who has never used this provider before.
- **Job to be done:** *"When I find a provider with no reviews, help me feel safe enough to book them, so I don't overpay for an established provider or abandon the search."*
- Evidence: *"If there are no reviews, I assume something's wrong with them. I'll pay more for someone with a track record."* — Buyer, churned at checkout. *"I wish I could see something, a verified ID, a portfolio, anything, before I commit money."* — Buyer, abandoned search. *"The established providers are booked out for weeks. I'd try someone new if I felt safe doing it."* — Repeat buyer

**Secondary user: the new provider** with skills but no marketplace history.

- **Job to be done:** *"Let my track record outside the marketplace count for something, so I can win my first booking before I churn."*
- Evidence: *"I'm great at my job but I'll never get a review if no one books me first. It's a chicken-and-egg trap."* — New provider, 0 bookings.


## Scope

- **In:** 

| Area | What exists in the prototype |
|---|---|
| Home / discovery (`/`) | Value proposition, category entry, provider cards incl. a zero-review provider (Alex M.), sticky Experiment Brief panel |
| Browse (`/browse`) | Provider grid, working filter chips, filter empty state with Clear filters, loading skeleton, error state |
| Categories (`/categories`) | Category listing |
| Provider profile (`/providers/alex`) | Trust panel (expandable checks), professional endorsements with verification tags, transparent pricing table + complexity tooltip, portfolio, guarantees, similar-providers carousel, sticky booking card |
| Booking flow (`/book/alex`) | Item quantities, slot picker, complex-item quote flag, compact summary card, "What's protected" expandable |
| Checkout (`/checkout`) | Items + slot recap, add-card form with full validation, payment-hold explanation, pending states |
| Booking detail (`/booking`) | "Awaiting provider acceptance" status, 2-hour window, items/total, cancel |
| My Bookings (`/bookings`) | Session booking card (labelled simulated), empty state, cancel |
| Research page (`/trust`) | Hypothesis, supplied baseline metrics, "What people told us" quotes, mock outcome selector, guarantees, kill switch |
| Experiment instrumentation | Session-only counters for trust/booking interactions **[Simulated]** |


- **Out (explicitly):** 

- Real payments or card processing — the card form is validated client-side but never submitted anywhere **[Simulated]**
- Real identity verification, background checks, insurance validation — check details are supplied copy
- Search screen and search-results/empty-search view
- Provider onboarding / provider-side app
- Exercisable cancellation or fix-it flows (guarantees are stated, not operable)
- Any backend, accounts, or persistence beyond the browser session
- Live analytics — all counters and outcomes are session-only or mock selections

## Requirements

| # | Requirement | Priority | Acceptance criteria | Status |
|---|---|---|---|---|
| R1 | Trust panel on new-provider profile | P0 | Four checks (Verified ID, Background check, Insurance, Marketplace-verified) each expand to show what was checked and when; microcopy "New provider on the marketplace — verified by identity, background, insurance, and professional endorsements." | Built; check content **[Simulated]** |
| R2 | Marketplace-verified badge | P0 | Badge appears on profile header and provider card; means all required category checks passed; never self-declared | Built; issuance logic **[Simulated]** |
| R3 | Professional endorsements distinct from reviews | P0 | Separate card style, relationship + organisation shown; verification tag only where marketplace confirmed employment ("Employment verified by marketplace" vs "Identity verified, employment self-reported") | Built; verification **[Simulated]** |
| R4 | Transparent per-item pricing | P0 | Chairs €25, Small tables €35, Dressers €45, Wardrobes €70, Complex items → custom quote; complexity tooltip explains variation | Built |
| R5 | Booking flow | P0 | Buyer picks items (quantities), a slot, flags complex items; running estimated total; CTA disabled with no items; "not charged until job is complete" assurance under the CTA; guarantees in "What's protected" expandable | Built; state session-only **[Simulated]** |
| R6 | Checkout / payment-method | P0 | Recap of items + slot; card form (number, expiry MM/YY, CVC, name) with inline blur errors; card number auto-formatted in 4-digit groups with live "n/16 digits" counter; CTA disabled until all fields valid | Built; card never submitted **[Simulated]** |
| R7 | Request booking pending states | P0 | On submit: "Sending request…" → "Awaiting provider response" → lands on booking detail | Built; timing **[Simulated]** |
| R8 | Booking detail | P0 | Status "Awaiting provider acceptance — Alex has 2 hours to accept."; slot, items, total, provider name; Cancel booking returns to empty state | Built; acceptance window **[Simulated]** |
| R9 | My Bookings reflects the session booking | P0 | After a confirmed request, the booking appears on My Bookings labelled "Simulated session booking · not live data"; empty state otherwise | Built **[Simulated]** |
| R10 | Browse filters | P1 | Chips filter the grid; zero-match combination shows "No providers available yet, try another search" with "Clear filters" | Built; availability data **[Simulated]** |
| R11 | Error state on failed fetch | P1 | "Something went wrong. Try again." with retry, on checkout submit, My Bookings, and Browse | Built; failures triggered by a prototype-only "Simulate fetch failure" toggle **[Simulated]** |
| R12 | Marketplace guarantees restated at decision points | P1 | Payment protection, 48-hour fix-it window, free cancellation shown on profile, booking, checkout | Built; guarantees not yet operable **[Simulated]** |
| R13 | Experiment instrumentation | P1 | Counters: Checks opened, Proof expanded, Booking starts, Completed; session-only, labelled "Local prototype interactions only. These are not live analytics." | Built **[Simulated]** |
| R14 | Mock test outcome selector | P2 | Lift observed / No meaningful lift / Insufficient data, each with its result copy; kill-switch statement always visible | Built **[Simulated]** |
| R15 | Research page (`/trust`) | P2 | Hypothesis + metadata, supplied baseline evidence, four user quotes, guarantees, kill-switch decision rule; supplied/simulated/mock content clearly distinguished from live analytics | Built |


## Data & events

_What gets stored, what gets tracked._

**Content data (static modules, supplied copy — not fetched):** provider profiles (Alex M. + 4 comparable providers), trust signals with check details and dates, 3 endorsements, pricing table + complexity tooltip, 4 baseline metrics, 4 user-voice quotes with attributions, 3 guarantees, 6 categories.

**Session-only state (browser `sessionStorage`, lost when the tab closes):**

| Store | Contents | Real? |
|---|---|---|
| `oikos-booking-state` | Provider, item counts, slot, complex flag, confirmed flag | **[Simulated]** — no server ever sees it |
| `oikos-experiment-signals` | `checksOpened`, `proofExpanded`, `bookingStarts`, `completed` counters | **[Simulated]** — local interactions only, explicitly not live analytics |
| `oikos-simulate-failure` | On/off flag for the simulated fetch failure toggle | **[Simulated]** prototype testing aid |


Events (all local, emitted via a browser custom event; nothing leaves the device):

| Event | Fired when |
|---|---|
| Checks opened | A trust-panel check is expanded on the provider profile |
| Proof expanded | The pricing complexity tooltip is opened |
| Booking starts | The booking flow is entered |
| Completed | The booking request completes (confirmation reached) |

There is **no backend, no analytics pipeline, and no persistence** beyond the browser session. For a real launch, these events map to: `trust_check_expanded`, `pricing_tooltip_opened`, `booking_started`, `booking_completed` — with first-booking rate as the primary metric.


## Open questions

1. **Verification supply chain** — who actually performs ID, background, and insurance checks, at what cost per provider, and how are re-checks scheduled (the prototype claims 24-month ID re-checks)?
2. **Payments** — which provider supports the hold-and-capture model ("charged only after the job is complete")? What are the dispute and refund mechanics?
3. **Guarantee operations** — how are the 48-hour fix-it window and free cancellation fulfilled and staffed? What do they cost per booking?
4. **Provider onboarding** — how do endorsements get collected and verified at scale? What happens when an endorser can't be reached?
5. **Experiment design** — what lift over the 2.3% baseline counts as success, over what sample size and duration? Who sees control vs treatment?
6. **Real instrumentation** — which analytics stack replaces the simulated counters, and how is first booking attributed (search → profile → book)?
7. **Cancellation & no-show edge cases** — provider declines or misses the 2-hour acceptance window: auto-rematch, refund, or notify?
8. **Category expansion** — does the trust panel generalize beyond Furniture Assembly (different required checks per category)?


