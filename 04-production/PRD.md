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


Kill switch: "If trust signals do not increase first bookings, trust is not the barrier — pivot required. The kill switch is displayed on the Experiment Brief page."

## Users & jobs

- **Primary user:** the buyer hiring a home-services provider (Furniture Assembly category, Athens) who has never used this provider before.
- **Job to be done:** *"When I find a provider with no reviews, help me feel safe enough to book them, so I don't overpay for an established provider or abandon the search."*
- Evidence: *"If there are no reviews, I assume something's wrong with them. I'll pay more for someone with a track record."* — Buyer, churned at checkout. *"I wish I could see something, a verified ID, a portfolio, anything, before I commit money."* — Buyer, abandoned search. *"The established providers are booked out for weeks. I'd try someone new if I felt safe doing it."* — Repeat buyer

**Secondary user: the new provider** with skills but no marketplace history.

- **Job to be done:** *"Let my track record outside the marketplace count for something, so I can win my first booking before I churn."*
- Evidence: *"I'm great at my job but I'll never get a review if no one books me first. It's a chicken-and-egg trap."* — New provider, 0 bookings.


## Scope

| Area | What exists in the prototype |
|---|---|
| Home / discovery (`/`) | Value proposition, category entry, provider cards incl. a zero-review provider (Alex M.); Experiment Brief moved to its own page, reached from the top menu |
| Browse (`/browse`) | Provider grid, five working filter chips, filter empty state with Clear filters, loading skeleton, error state |
| Categories (`/categories`) | Category listing |
| Search results (`/search`) | Keyword matching (name, headline, badges, item words), matching price hints, no-results empty state, loading skeleton, error state **[Simulated availability]** |
| Provider profile (`/providers/alex`) | Trust panel (expandable checks), professional endorsements with verification tags, transparent pricing table + complexity tooltip, portfolio, guarantees, similar-providers carousel, sticky booking card |
| Booking flow (`/book/alex`) | Item quantities, slot picker, complex-item quote flag, compact summary card, "What's protected" expandable |
| Checkout (`/checkout`) | Items + slot recap, add-card form with full validation and live digit counter, payment-hold explanation, pending states |
| Booking detail (`/booking`) | "Awaiting provider acceptance" status, 2-hour window, items/total, cancel |
| My Bookings (`/bookings`) | Session booking card (labelled simulated), empty state, cancel |
| Research page (`/trust`) | Hypothesis, supplied baseline metrics, "What people told us" quotes, guarantees |
| Experiment Brief (`/experiment-brief`) | All brief sections (baseline/user-voice toggle, simulated session signal counters, selectable mock outcomes) plus the Kill switch / pivot indicator |
| Experiment instrumentation | Session-only counters for trust/booking interactions **[Simulated]** |


- **Out (explicitly):** 

- Real payments (the card form is a validation exercise; nothing is tokenized or sent)
- Real identity verification, background checks, or insurance verification
- Provider onboarding
- Fix-it / cancellation fulfilment operations
- Authentication and user accounts
- Live analytics or real experiment measurement
- Working profiles or booking flows for providers other than Alex M.


## Requirements

| # | Requirement | Priority | Acceptance criteria | Status |
|---|---|---|---|---|
| R1 | Provider profile shows an expandable trust panel with verification checks (ID, background, insurance, references) | P0 | Each check row expands to show summary, detail, and checked-on date; expanding increments the simulated "checks opened" counter | Database-backed content; counter simulated |
| R2 | Endorsements display only marketplace-checked professional references | P0 | Each endorsement shows name, relationship, organisation, years worked, quote, and a verification tag | Database-backed |
| R3 | Transparent per-item pricing table with complexity tooltip | P0 | Chairs €25, Small tables €35, Dressers €45, Wardrobes €70; complex items show "custom quote" with an explanatory tooltip; toggling the tooltip increments the simulated "proof expanded" counter | Database-backed content; counter simulated |
| R4 | Marketplace guarantees shown on the profile and collapsed into a "What's protected" row in the booking summary | P0 | Guarantees render from data; booking summary row expands/collapses with chevron | Database-backed |
| R5 | Booking flow: select item quantities, a slot, optional complex item | P0 | "Review and pay" writes an unconfirmed booking to session storage and navigates to checkout; entering the flow increments the simulated "booking starts" counter | Simulated (session-only) |
| R6 | Checkout recaps items + slot, collects card details with validation, explains the payment hold | P0 | Card number exactly 16 digits auto-formatted in groups of 4 with a live "n/16 digits" counter; expiry MM/YY valid month and non-past year; CVC exactly 3 digits; name ≥ 2 characters; inline errors on blur; "Request booking" disabled until all fields pass; hold message "You're not charged until the job is complete." with shield icon | Simulated — card data never leaves browser memory |
| R7 | Request submission shows pending states before landing on booking detail | P0 | On submit: "Sending request…" then "Awaiting provider response", then navigation to booking detail; skeleton shown while submitting | Simulated (fixed timers) |
| R8 | Booking detail shows awaiting-acceptance status with slot, items, total, and cancel | P0 | "Alex M. has 2 hours to accept." visible; cancel clears the session booking | Simulated (session-only) |
| R9 | My Bookings shows the confirmed session booking, clearly labelled | P0 | After confirmation the booking appears with a "Simulated session booking · not live data" label; empty state when none exists | Simulated (session-only) |
| R10 | Browse filters actually filter; empty result shows a designed empty state | P0 | Chips combine (AND); "New providers" + "Rated 4.9+" yields "No providers available yet, try another search" with a "Clear filters" action beside the chips and inside the empty card | Database-backed list; filtering client-side |
| R11 | Search results screen with keyword matching and no-results state | P0 | `/search?q=…` matches providers by name/headline/area/badges and item words (e.g. "wardrobe" → Wardrobes €70 price hint); day words match simulated availability labelled as such; zero matches shows "No providers match — try a nearby category." with nearby categories, suggested searches, and "Clear search" | Database-backed list; matching and availability simulated |
| R12 | Loading skeletons on Browse, Search, Checkout, and My Bookings | P1 | Skeleton cards/rows appear while data loads; no spinners | Implemented |
| R13 | Error state on fetch failure | P1 | "Something went wrong. Try again." with retry; triggerable via the "Simulate fetch failure" toggle | Simulated failure flag |
| R14 | Dedicated Experiment Brief page | P1 | `/experiment-brief` shows hypothesis, simulated session signal counters, selectable mock outcomes (lift / no lift / insufficient data), baseline vs user-voice toggle, and the kill-switch panel; linked from the top navigation on every page | Content database-backed; counters and outcomes simulated |
| R15 | "How we verify" research page | P2 | Baseline metrics ("14% for reviewed" in red), "What people told us" quotes, guarantees; links to the experiment brief via top nav | Database-backed |



## Data & events

_What gets stored, what gets tracked._

**Content data (static modules, supplied copy — not fetched):** 

| Data | Table(s) | Used by |
|---|---|---|
| Providers (5 rows) | `providers` | Home, Browse, Search, Categories |
| Verification checks (4 for Alex M.) | `provider_verification_checks` | Provider profile trust panel |
| Endorsements (3) | `provider_endorsements` | Provider profile |
| Per-item prices | `provider_price_items` | Provider profile, booking flow |
| Categories (6) | `categories` | Categories, Search no-results |
| Guarantees (3) | `guarantees` | Profile, How we verify |
| Research metrics (4) | `research_metrics` | How we verify, Experiment Brief |
| Research quotes (4) | `research_voices` | How we verify, Experiment Brief |

All public content tables are read-only for anonymous visitors (SELECT policies only). Reads go through server functions (`getProviders`, `getProviderProfile`, `getCategories`, `getTrustContent`).


### Simulated (session-only, clearly labelled in the UI)

| State | Storage | Notes |
|---|---|---|
| Booking state | `sessionStorage: oikos-booking-state` | One booking at a time; no IDs, ownership, or history |
| Experiment signals | `sessionStorage: oikos-experiment-signals` | Counters: checks opened, proof expanded, booking starts, completed |
| Fetch-failure flag | `sessionStorage: oikos-simulate-failure` | Drives the designed error states |
| Search availability | in-code list | "tomorrow"/weekday words match a hard-coded availability list, labelled simulated |
| Provider acceptance | fixed timers | Prototype stops at "awaiting acceptance" |
| Card details | React memory only | Validated, never transmitted or stored |



Events (all local, emitted via a browser custom event; nothing leaves the device):

| Event | Fired when |
|---|---|
| Checks opened | A trust-panel check is expanded on the provider profile |
| Proof expanded | The pricing complexity tooltip is opened |
| Booking starts | The booking flow is entered |
| Completed | The booking request completes (confirmation reached) |

There is **no backend, no analytics pipeline, and no persistence** beyond the browser session. For a real launch, these events map to: `trust_check_expanded`, `pricing_tooltip_opened`, `booking_started`, `booking_completed` — with first-booking rate as the primary metric.

- The `bookings` and `booking_items` tables exist in the database with owner-scoped security policies, but confirmed bookings are **not** saved to them — there is no sign-in yet, so a booking has no owner.
- Experiment counters are client-controlled and cannot support real measurement.


## Open questions

1. **Authentication model** — guest-only booking via link, or sign-in (email / Google) so bookings persist and have an owner? This unblocks saving bookings to the database.
2. **Real verification supply chain** — which providers supply ID, background, and insurance verification, and how is checked-on freshness maintained?
3. **Payment provider** — hosted fields / tokenization, authorization hold on request, capture on completion, refunds and disputes.
4. **Cancellation and fix-it operations** — who can cancel, within what window, and how are fix-it requests routed?
5. **Provider onboarding** — how do new providers submit verification evidence and endorsements?
6. **Analytics instrumentation** — event definitions, experiment assignment, sample size, and duration rules to replace the simulated counters.
7. **Experiment success threshold** — what lift in first bookings constitutes success before the kill switch is evaluated?



