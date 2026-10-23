# Test dive scenario

A manual acceptance walkthrough for `/dive`. Pick any public storefront you have no confidential relationship
with — ideally a **B2B** store (login-gated pricing, quote CTAs, spec-driven facets) for one run and a mainstream
**B2C** store for another, since the B2B variants are the easiest place to over-penalize normal structure.
Use `<store>` below for whichever you pick.

Run it in a session with the Claude-in-Chrome extension connected and Chrome open to the store.

## Runs

```
# A — ride-along (you shop, /dive observes)
/dive https://<store>/

# B — agent-driven with check-ins
/dive https://<store>/ --drive --checkin, shop for <a realistic product under a price cap>

# C — with upfront context
/dive https://<store>/ --drive --checkin --context "<a fact you already know, e.g. tracking is Adobe, not GTM>"
```

Suggested path: Landing → header search (Findability) → a category listing → a PDP → add to cart → cart →
first checkout step (stop before any account/payment entry).

## What to verify

1. **B2B structure isn't flagged as generic friction.** Login-gated price or "Request a Quote" should read as the
   documented B2B pattern; *price wall with no context* only if the gate has zero explanation.
2. **Tracking check isn't GTM-only.** At PDP/Cart, it checks for a GTM container and the stage's ecommerce
   event; if there's no GTM, it looks for an equivalent through the site's actual analytics vendor before
   calling a *silent tracking gap*.
3. **WebMCP readiness is reported neutrally.** Absence of `toolname`/`tooldescription` is "no *agent-ready
   form* found", not friction.
4. **Root-cause tags stay within the fixed set** — `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
   `[DATA QUALITY]`, `[STRATEGY]`.
5. **Viewport-stability guard.** Watch the Findability stop's Evidence line for a viewport note. If the viewport
   drifts, any layout finding there carries `, unconfirmed` rather than a bare HIGH.
6. **`--checkin` actually pauses** (Runs B/C) at the configured interval, and **always** before logging a
   self-sourced HIGH finding — and waits for you.
7. **Adaptive cadence.** After clean check-ins the interval widens beyond `N`; a self-sourced HIGH still forces
   an immediate pause.
8. **Post-hoc corrections are addenda.** A retest that doesn't reproduce gets a dated addendum; the original
   bullet stays intact.
9. **Output paths are environment-detected** — a local `outputs/` folder or the session outputs dir, never a
   hardcoded machine path.
10. **Screenshot persistence is checked once.** If saves no-op, it says so once and falls back to a text
    storyboard.
11. **`--context` is acknowledged upfront and actively checked** at the relevant stop, not just repeated in the
    report.
12. **Site notes round-trip.** After Run C, `outputs/site-notes/<store>.md` exists (or gained a dated section).
    A second dive on the same store with no `--context` loads it automatically and says so.
13. **Checkout network privacy.** Checkout-stop Evidence names events fired, never request URLs, query
    strings, or payloads.

## Quick-marks to try

- `! price hidden until login` — infers severity + a tag, ideally ties to the B2B pricing note.
- `+ spec table is thorough` — positive, possibly a named pattern.
- `? does this require a business account to check out` — rolls into Open Questions.
- `* clean product imagery` — neutral note.

## Report checklist (after `end dive`)

- [ ] Shareable recap up top; no unconfirmed finding in the top-3 frictions
- [ ] Header shows mode (`turn`/`drive`) and check-in status
- [ ] Funnel map shows momentum per stop (smooth / hesitation / stall / dead-end)
- [ ] Storyboard: screenshot + caption per stop (checkout may be skipped per privacy rule)
- [ ] Findability section covers search/nav specifically, incl. any viewport-drift note
- [ ] Friction inventory doesn't over-penalize normal B2B structure
- [ ] A line on tracking status and a line on WebMCP readiness
- [ ] Report saved as `dive-report-<store>-<timestamp>.md` in the environment's outputs location

## Guardrails

Never create a real account or enter real payment/personal details. Stopping at that boundary is logged as
"confirmation not reached" — expected behavior, not a test failure. In `--drive` mode the dive should stop
itself there without being told.
