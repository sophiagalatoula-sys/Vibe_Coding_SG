# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain:  checkout/payment-method step and the booking detail screen

### Step 1: Expand, add the checkout/payment-method step and the booking detail screen so the payment-hold promise and the 2-hour acceptance window are both visible, anchored to TaskRabbit.
```
Build the next phase of this app in a strict sequence:
1. Add a screen "Checkout / payment-method". Show the items + slot recap, an "add card" step, and the payment-hold explanation ("you're not charged until the job is complete"). Match the layout and spacing of the TaskRabbit.
2. Add a screen "Booking detail". Show status: awaiting provider acceptance, the booked slot, items, price, and a cancel option — this is where "Alex has 2 hours to accept" becomes visible. Match the data density of the TaskRabbit.
3. Navigation: write the logic so Checkout / payment-method links to Booking detail — tapping "Request booking" on checkout lands the buyer on Booking detail, and "Go to my bookings" from the confirmation also reaches it instead of the current empty-state page.

Build these in order so Checkout / payment-method is the anchor for Booking detail.
```

### Step 2: Behavior, hard-code the three friction points the audit flagged — the silent booking submit, the My Bookings dead-end, and the decorative filters — with exact copy.
```
Apply the following logic constraints to the Request booking, My Bookings, and Browse filter flows:
- Use a skeleton/pending state for the Request booking loading state: on submit, show "Sending request…" followed by "Awaiting provider response" before landing on the booking detail screen — this is what makes the "Alex has 2 hours to accept" promise believable and testable.
- If no data is present, show the empty state: on My Bookings, right after a confirmed request the session booking must appear there (session-only, labeled like the other experiment counters) — not "No upcoming bookings yet." On Browse, if a selected filter chip returns zero providers, show the empty state: "No providers available yet, try another search" with a "Clear filters" action.
- On fetch failure, trigger the error state: "Something went wrong. Try again."

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, polish the booking summary card so the price and the not-charged-until-complete assurance sit in the same eyeline.
```
The booking summary card (right column of /book/alex) needs a professional trust-forward polish, benchmarked against TaskRabbit.
1. Start by listing the 3 biggest gaps in typography and spacing compared to TaskRabbit.
2. Once you've identified those, resize the headers and update the booking summary card to: a compact order recap, the "not charged until job is complete" assurance placed directly under the CTA with a shield icon, and the guarantees collapsed into a single "What's protected" expandable row.

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- Testing the unhappy path right after a Behavior prompt (typing garbage into a form, submitting empty fields) surfaces gaps a passing build hides — the checkout form looked complete but had zero input validation until we tried it.
- When a prompt claims a feature works but it's not visible in the UI, ask the tool to explain where it lives and how to trigger it before writing another build prompt — cheaper than guessing and re-prompting blind.
- The Refine template's "don't change anything else" line held in practice — the surgical polish prompt touched only the booking summary card, nothing else broke.

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

Checkout payment-method fields (card number, expiry, CVC, name) had no input validation — "Request booking" enabled regardless of what was typed or how many characters. Fix: a follow-up prompt added exact validation rules per field (16-digit card, valid MM/YY, 3-digit CVC, non-empty name) with inline error copy, and disabled Request booking until all four fields pass.

The Browse filters' empty state (built by the Behavior prompt) wasn't discoverable in the UI after the build — asked the tool to explain exactly where it lives and how to trigger it, then tested it directly and confirmed it fires as specified.

