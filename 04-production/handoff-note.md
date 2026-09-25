# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

Oikos is a high-fidelity research prototype for testing whether stronger trust evidence improves first-booking intent for new, zero-review home-service providers. The working journey is discovery → Browse or Search → Alex M.'s provider profile → booking setup → checkout → booking detail → My Bookings, with supporting trust, loading, empty, error, cancellation, and experiment views. The frontend is structured like a maintainable React application: URLs and metadata live in thin TanStack Start route files, while screens, components, data helpers, and state helpers are grouped by feature. Marketplace content — providers, verification checks, endorsements, per-item pricing, categories, guarantees, and research metrics/quotes — is served from a managed Postgres database (Lovable Cloud) through typed server functions and TanStack Query. Everything transactional is still simulated: bookings live only in browser session storage, provider acceptance, network delays, and fetch failures are timers and toggles, the card form never touches a payment provider, and experiment counters are client-side session values. There is no account system, payment processor, or production analytics pipeline. Treat the current code as a validated interaction and visual prototype — not as a production commerce foundation without replacing the simulated flows and browser-only state.
_____

## Architecture (plain language)

### Frontend

- **Framework:** React 19 on TanStack Start, built by Vite. TanStack Router supplies file-based routing and the server-rendered application shell.
- **Routes:** `src/routes/` owns URLs, route-specific metadata, loaders, and the small adapters that mount screens. Do not edit `src/routeTree.gen.ts`; TanStack generates it from route files.
- **Features:** `src/features/` contains the named screens and the data, components, and helpers they own:
  - `home-discovery/`
  - `browse/`
  - `categories/`
  - `providers/`
  - `booking/`
  - `search/` (keyword matching, search results screen, no-results component)
  - `marketplace-trust/`
  - `prototype/` for explicit simulation controls
- **Shared presentation:** `src/components/layout/` contains the header, footer, and breadcrumbs. `src/components/feedback/StateBlocks.tsx` contains reusable loading, empty, and error states. `src/components/ui/` contains general UI primitives.
- **Styling:** Tailwind CSS 4, semantic tokens in `src/styles.css`, and the Bricolage Grotesque/DM Sans font pair loaded by the root route.
- **Application shell:** `src/routes/__root.tsx` provides document metadata, global error and not-found screens, TanStack Query context, and the nested-route outlet.
- **Start middleware:** `src/start.ts` defines a server error middleware, a CSRF middleware for server functions, and registers `attachSupabaseAuth`, which adds the Supabase bearer token to authenticated server-function calls.
- **Build/runtime configuration:** `vite.config.ts` uses Lovable's TanStack preset and points server rendering at `src/server.ts`. The preset supplies the TanStack, React, Tailwind, aliases, and deployment build integration.

### Route-to-screen map

| URL | Route adapter | Screen |
|---|---|---|
| `/` | `src/routes/index.tsx` | `HomeDiscoveryScreen` |
| `/categories` | `src/routes/categories.tsx` | `CategoriesScreen` |
| `/browse` | `src/routes/browse.tsx` | `BrowseScreen` |
| `/search?q=` | `src/routes/search.tsx` | `SearchResultsScreen` (zero matches render `NoResults`) |
| `/providers/$slug` | `src/routes/providers.$slug.tsx` | `ProviderProfileScreen` |
| `/book/$slug` | `src/routes/book.$slug.tsx` | `BookingFlowScreen` |
| `/checkout` | `src/routes/checkout.tsx` | `CheckoutPaymentMethodScreen` |
| `/booking` | `src/routes/booking.tsx` | `BookingDetailScreen` |
| `/bookings` | `src/routes/bookings.tsx` | `MyBookingsScreen` |
| `/trust` | `src/routes/trust.tsx` | `HowWeVerifyScreen` |
| `/experiment-brief` | `src/routes/experiment-brief.tsx` | `ExperimentBriefScreen` |

The two dynamic routes currently accept only the `alex` slug. Other provider slugs return not found, even though additional providers appear in discovery fixtures.

The shared header (`src/components/layout/Header.tsx`) carries "Browse Providers", "Categories", "My Bookings", the search pill (links to `/search`), and the links "How we verify" and "Experiment Brief" (in that order).

### Backend and data

- **Database (real):** a managed Postgres instance (Lovable Cloud) holds the marketplace content in seeded tables: `providers`, `provider_verification_checks`, `provider_endorsements`, `provider_price_items`, `categories`, `guarantees`, `research_metrics`, and `research_voices`. All of these are publicly readable (anon SELECT policies) and are seeded with exactly the prototype's content. The `bookings` and `booking_items` tables exist with owner-scoped RLS, but nothing writes to them yet because there is no sign-in — this is the project's main open question.
- **Server functions (real):** `src/lib/marketplace.functions.ts` exposes `getProviders`, `getProviderProfile({ slug })`, `getCategories`, and `getTrustContent`. They use a server-side publishable-key Supabase client with an apikey-only header (opaque `sb_` keys are not JWTs, so the default bearer header is stripped). All reads are public-table reads under anon SELECT policies; no privileged (service-role) client is used anywhere.
- **Query wiring (real):** data modules under `src/features/*/data/` export TanStack Query `queryOptions`; screens fetch with `useSuspenseQuery`, and route loaders warm the cache via `context.queryClient.ensureQueryData(...)`.

Session-only (simulated) state:

- **Booking persistence:** `src/features/booking/lib/booking-session.ts` stores one booking in `window.sessionStorage` under `oikos-booking-state`.
- **Experiment counters:** `src/features/marketplace-trust/lib/experiment-session.ts` stores session counters under `oikos-experiment-signals` and uses a browser `CustomEvent` to refresh the Experiment Brief.
- **Failure simulation:** `src/features/prototype/lib/simulated-request.ts` uses timers and a session-storage flag. It does not make a network request.
- **Search availability:** `src/features/search/lib/match-providers.ts` matches day words against invented fixture availability, labeled as simulated on the results page.
- **Payment details:** entered card values remain in React memory only. They are validated for prototype interaction but are never tokenized or sent anywhere.

### Key flows

1. **Discovery:** Home, Categories, and Browse present database-backed provider/category content. Browse filters are combined and can produce a designed empty state ("No providers available yet, try another search" with Clear filters).
2. **Search:** `/search` matches a free-text query (e.g. "wardrobe assembly Friday") against provider name, headline, badges, and price items, with a loading skeleton. Zero matches render the designed `NoResults` screen — "No providers match — try a nearby category." — with nearby categories, suggested searches, and a Clear search action.
3. **Provider trust:** `/providers/alex` presents verification checks, professional endorsements, work examples, transparent prices, guarantees, and similar-provider cards. Trust-panel interactions increment simulated experiment counters.
4. **Booking setup:** `/book/alex` selects quantities, a fixed fixture slot, and an optional complex item. Continuing writes an unconfirmed `BookingState` to session storage.
5. **Checkout:** `/checkout` reads that state, recaps the order, and validates card number (16 digits, formatted 4-4-4-4 with a live digit counter), expiry, CVC, and cardholder name. Submitting cycles "Sending request…" → "Awaiting provider response" before navigating to `/booking`. The form explicitly does not process payment.
6. **Booking management:** `/booking` shows the awaiting-acceptance detail ("Alex M. has 2 hours to accept"). `/bookings` shows the same session booking with a simulated-data label. Cancellation clears the session booking and reveals the empty state.
7. **Experiment brief:** `/experiment-brief` combines the hypothesis, supplied baseline/research content, session interaction counters, and manually selectable mock outcomes, and carries the kill switch / pivot indicator. It is a research narrative, not live reporting.
8. **Failure states:** Browse, Search, Checkout, and My Bookings can be forced into their designed error states ("Something went wrong. Try again.") using the visible simulation control.

## What is solid vs. what is duct tape

### Solid

- Feature-based boundaries make screen ownership and dependencies easy to locate.
- Route files are thin and keep URL behavior and route-specific metadata separate from display code.
- Marketplace content is database-backed and read through typed server functions with narrow public SELECT policies — no service-role access on read paths.
- Data access goes through TanStack Query `queryOptions` with route loaders warming the cache, and every route declares `errorComponent` / `notFoundComponent` fallbacks.
- Booking calculation, session access, and payment validation are typed and separated from screen markup.
- Checkout has field-level validation, clear errors, formatted card-number input with a live digit counter, and a disabled submit action until all fields pass.
- Browse and search flows include explicit loading, empty, error, and pending presentation states.
- The root route includes user-facing not-found and error handling.
- The main journey has been exercised on desktop and mobile after the feature refactor: Home → Search/Browse → Provider Profile → Booking Flow → Checkout → Booking Detail → My Bookings.
- TypeScript is configured strictly, including unchecked indexed-access and exact optional-property checks.

### Duct tape / prototype-only

- The database is seeded with the prototype's fixture content; only Alex M. is a functional provider. The other browse cards do not have working profile or booking routes.
- `bookings` and `booking_items` tables exist but nothing writes to them — booking state is session storage only. There are no booking IDs visible to the user, ownership checks, history, concurrency controls, or durable status transitions.
- Provider acceptance is not real; the prototype stops at an awaiting-response state.
- The card form is a UI validation exercise, not a payment integration. It does not perform a Luhn check, tokenize data, authorize a card, place a hold, or meet payment-compliance requirements.
- "Fetching" is a timer; failure is a manually controlled browser flag. Search availability ("Friday", "tomorrow") is matched against invented fixture data.
- Experiment counters are client-controlled session values. Mock outcome buttons are manually selectable and are not calculated from experiment data.
- There is no automated test suite or declared test command. Current confidence comes from strict type checking and manual browser-flow verification.
- Some small data/state modules are intentionally terse; reformat them before extending complex behavior so future diffs remain readable.

## Risks and assumptions

### Current risks

- **State durability:** session storage is isolated to a browser session and is not a source of truth. A new tab/session or cleared browser data can lose or isolate a booking.
- **Direct navigation:** opening Checkout, Booking Detail, or My Bookings without completing the preceding session flow can legitimately show a missing/empty state.
- **Bookings unwired:** the booking tables and their RLS policies are in place, but without authentication nothing can be saved. Deciding the sign-in model (email/Google vs. guest) is the prerequisite for real bookings.
- **Client trust:** all stored booking and experiment values are editable by the user. None can support authorization, pricing, billing, or analytics decisions.
- **Payment safety:** never connect the existing card form directly to an internal endpoint or store its raw values. Replace it with hosted fields or client tokenization from a compliant payment provider.
- **Fixture freshness:** slots, dates, coverage, verification dates, availability, prices, and research metrics are seeded copy and can become misleading.
- **Marketplace completeness:** the UI implies multiple providers, but only one provider has an end-to-end route and booking flow.
- **Measurement validity:** the prototype proves that the proposed experience can be demonstrated and tested. It does not prove the trust hypothesis. Real validation needs agreed event definitions, assignment/cohorts, sample-size and duration rules, and server-side or independently verifiable measurement.
- **Regression coverage:** there are no automated unit, integration, accessibility, or browser tests committed to the project.
- **Toolchain coupling:** `vite.config.ts` delegates setup to `@lovable.dev/vite-tanstack-config`; do not duplicate its plugins. Confirm the target hosting environment before changing deployment configuration.

### Productionization assumptions

A production implementation will need:

1. Authenticated customer and provider identities, with server-side ownership and role checks.
2. Provider content evolving from seeded fixtures into a real provider source with provenance for verification and endorsements.
3. Server-side availability and price validation at request time; displayed seeded values cannot be authoritative.
4. Durable booking writes with IDs, idempotent creation, status transitions, timestamps, cancellation rules, and an audit trail (the tables exist; the app does not write to them yet).
5. A compliant payment provider for tokenization, authorization/holds, capture, refunds, disputes, and webhook-driven reconciliation.
6. Real provider notification and acceptance/decline behavior, including expiry of the two-hour response window.
7. Production analytics with stable event names, anonymous/authenticated identity strategy, experiment assignment, and trustworthy conversion reporting.
8. Observable APIs with structured errors, logs, tracing, retry/idempotency policies, and monitoring.
9. Automated tests covering calculations, validation, state transitions, routes, accessibility, and the critical booking journey.


_____

## How to run it

```
[setup + run commands]
```

### Prerequisites

- A current Node.js installation. The existing README suggests installing through `nvm`.
- npm is documented. The repository also contains Bun configuration, so the team should select one package manager and avoid generating competing lockfiles.

### Install and start

Using the documented npm workflow:

```sh
npm install
npm run dev
```

Then open the local URL printed by Vite. In the Lovable workspace, the preview is served at `http://localhost:8080`.

Equivalent Bun commands can be used if Bun is the team's chosen package manager:

```sh
bun install
bun run dev
```

### Available scripts

These commands match `package.json`:

```sh
npm run dev        # Start the Vite development server
npm run build      # Create a production build
npm run build:dev  # Create a development-mode build
npm run preview    # Serve the built application locally
npm run lint       # Run ESLint across the repository
npm run format     # Rewrite files with Prettier
```

There is no `test` or dedicated `typecheck` script in `package.json`.

### Principal URLs

```text
/                    Home and discovery
/categories          Service categories
/browse              Provider browse and filters
/search?q=           Search results (No results variant at zero matches)
/providers/alex      Functional provider profile
/book/alex           Item and slot selection
/checkout            Payment-method prototype
/booking             Current booking detail
/bookings            Current session booking or empty state
/trust               Trust evidence and research content
/experiment-brief    Experiment brief and kill switch
```

### Happy-path smoke test

1. Open `/search?q=wardrobe%20assembly%20Friday` and confirm one matching provider (Alex M.) with the simulated-availability note.
2. Search `plumbing` and confirm the "No providers match — try a nearby category." screen with categories, suggestions, and Clear search.
3. Open `/browse` and select Alex M.
4. Open trust-panel rows and verify the details expand.
5. Choose **Book Alex** and confirm item quantities and a slot on `/book/alex`.
6. Select **Review and pay**.
7. On `/checkout`, use any 16 numeric digits, a future `MM / YY`, any three numeric CVC digits, and a fictional cardholder name of at least two characters.
8. Confirm **Request booking** stays disabled until all fields are valid.
9. Submit and observe **Sending request…** followed by **Awaiting provider response**.
10. Confirm `/booking` shows the slot, items, total, and "Alex M. has 2 hours to accept."
11. Open My Bookings and confirm the session booking is visible and labeled as simulated.
12. Cancel it and confirm the empty state appears.

### Designed state checks

- **Browse empty state:** open `/browse`, wait for provider cards, select **New providers**, then **Rated 4.9+**. The result becomes "No providers available yet, try another search." Use either visible **Clear filters** action to restore the list.
- **Search no-results:** `/search?q=plumbing` renders the dedicated no-results screen; **Clear search** empties the query.
- **Simulated fetch error:** on Browse, Search, Checkout, or My Bookings, turn on **Simulate fetch failure**. Retry or reload the relevant data view to show "Something went wrong. Try again." Turn the toggle off before continuing the happy path.

These controls exercise presentation states only; they do not reproduce real API or infrastructure failures.


