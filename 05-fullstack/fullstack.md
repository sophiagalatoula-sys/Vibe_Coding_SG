# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

_____

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| providers | id, slug, name, headline, area, reviews, rating, jobs, price, is_new, badges | Public trust/marketplace content. Everyone reads (anon + authenticated); no client writes. |
| provider_verification_checks | provider_id, check_key, label, status, summary, detail, checked_on | The trust panel's four checks (Verified ID, Background check, Insurance, Marketplace-verified). Public read. |
| provider_endorsements | provider_id, name, relationship, organisation, years_worked, quote, verification, marketplace_checked | Early social proof with verification tags. Public read. |
| provider_price_items | provider_id, item, price_label, amount_eur, note | Per-item pricing. Public read; also the server-side source of truth for booking totals. |
| provider_availability | provider_id, starts_at, label, is_booked | Time slots. Public read; `is_booked` is flipped only inside `create_booking` (one-step slot lock). |
| provider_portfolio | provider_id, image_path, alt, sort_order | Profile photos. Public read. |
| categories | slug, name, provider_count, sort_order | Browse/Category entry. Public read. |
| guarantees | title, body, sort_order | Marketplace guarantees. Public read. |
| research_metrics | value, label, detail | Baseline metrics (41% zero-review, 2.3% vs 14%, 68% exit, 19 days). Public read. |
| research_voices | quote, who, mechanism | The four research quotes. Public read. |
| profiles | id (= auth user id), display_name, created_at | Auto-created at sign-up by the `handle_new_user` trigger. Owner-only read/update. |
| user_roles | user_id, role (app_role: admin, moderator, user) | Read via the `has_role()` security-definer function. Own roles readable; **no** client writes — roles are granted directly in the database. |
| bookings | user_id, provider_id, slot, slot_id, status, total_eur, accept_by, notes | Statuses: `awaiting_acceptance`, `accepted`, `declined`, `cancelled`, `expired`. Owner-only RLS. Direct INSERT/UPDATE/DELETE revoked from clients — writes go through `create_booking` / `cancel_booking` only. |
| booking_items | booking_id, item_id, label, unit_price_eur, quantity | Line items. Triggers validate quantity ≥ 1, that each price matches the provider's price list, and that the total equals the item sum. |
| invites | sender_email, recipient_email, status (pending/accepted/declined/expired), created_at | Clients **cannot** SELECT (email is personal data). The dashboard reads via `invite_dashboard()`, which masks addresses. Anyone may INSERT a `pending` row (prototype-only — no real emails are sent; tighten to signed-in users before real use). |
| experiment_events | session_id, user_id (nullable), event_type, provider_id | Prototype experiment signals (checks_opened, proof_expanded, booking_started, booking_completed). Anyone inserts; only admins read. Session-only counters on screen are simulated. |


## Access rules

_Who can see / do what? Where are the auth boundaries?_

- **Public (anon + signed-in read):** all marketplace content — providers, verification checks, endorsements, prices, slots, portfolio, categories, guarantees, research content.
- **Owner-only:** bookings and booking items (RLS scoped to `auth.uid()`), profiles (own row), own user_roles rows.
- **Auth boundaries:** Checkout, Booking detail, and My Bookings live behind the `_authenticated` layout gate — signed-out visitors are redirected to `/auth?redirect=…` and land back where they were after sign-in. All booking server functions use `requireSupabaseAuth`, so every database call runs as that user with RLS enforced.
- **Write path:** the client sends only item ids + quantities + a slot id. `create_booking` (security-definer) looks up prices from the price list, computes the total, locks the slot in a single step, and stamps `accept_by = created_at + 2h`. `cancel_booking` frees the slot and refuses cancelled/accepted/expired bookings. The browser is never trusted for prices, totals, or slot state.
- **Invites:** insert-only for anyone (status forced to `pending` by policy); reads only through `invite_dashboard()` with masked emails — full addresses are unreachable from the app.
- **Privileged work:** `service_role` bypasses RLS and is used server-side only (never shipped to the browser).

_____

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| **Empty / first-run state** | No bookings → raw error or "not found" noise; no open slots → dead UI | Friendly "No upcoming bookings yet" empty state; "no open slots" message when every slot is booked; a provider with no price list pauses the booking flow with a notice (code in place; every provider currently has prices, so not yet seen on screen) |
| **Bad / malicious input** | Client-supplied prices and totals were trusted | Server recalculates prices/totals from the price list — tampered values rejected by triggers; quantity clamped 1–20, max 4 line items, duplicates rejected; card form validates inline on blur (exactly 16 digits, valid MM/YY not in the past, 3-digit CVC, name ≥ 2 chars) with the CTA disabled until all pass; malformed or foreign booking ids return "not found", never another user's data; notes capped at 500 chars, search input at 100 |
| **Failure / offline** | Errors surfaced as blank screens or raw messages | Any fetch failure shows "Something went wrong. Try again." with retry (a dev-only "Simulate fetch failure" switch tests it; hidden in production); two buyers racing for one slot → exactly one wins, the rest get "That time slot was just booked by someone else"; no provider acceptance within 2 hours → booking shows **Expired**, "not charged", slot freed; double cancel or cancel-after-acceptance → clear message and the Cancel button disappears; session expires mid-checkout → sign in, then back to Checkout with items and slot kept; signing out from Booking detail clears cached data with no 401 storm |


## Stress test results

_What you threw at it, and what held / broke._

**Held (verified):**
- 50 concurrent booking attempts on a single slot → exactly **1** succeeded; every other request got the friendly SLOT_TAKEN error and the slot never double-booked.
- Tampered/direct writes: attempts to insert or update bookings and booking items directly, or inject wrong prices/quantities, were all rejected (revoked grants + trigger validation). Custom-quote-only bookings saved correctly at €0.
- Cancellation rules: double cancel, cancel-after-acceptance, and cancel-after-expiry all refused with clear error codes; expired bookings release their slots.
- Cross-user access: another account opening someone else's booking link (valid or malformed) sees "We couldn't find this booking" — no data leak.
- Long/unsafe input: a 300-character search was truncated to 100 and rendered as plain text (no injection).

**Broke, then fixed:**
- The invites table initially allowed anonymous reads, exposing full email addresses — fixed in migration `0007_protect_invite_emails`: SELECT revoked from anon/authenticated and the dashboard switched to `invite_dashboard()` with masked addresses. Counts unchanged.

**Not yet exercised (honest gaps):**
- 200-parallel-read soak test and a 10,000-row `experiment_events` insert test were planned but not run.
- The no-price-list browser path (every provider currently has prices).
- Automated regression tests don't exist yet — the stress tests above were run once, by hand.

_____
