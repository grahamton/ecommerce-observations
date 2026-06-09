---
name: shopping-dive
description: Methodology for riding along as a user shops a storefront — observing the live tab across the funnel, capturing structured per-stop observations, calling out named patterns and friction, and synthesizing a dive report. Use when running a shopping ride-along / dive, recording an ecommerce shopping session, or analyzing a checkout funnel as it is walked.
---

# Shopping Dive

A **dive** is a live ride-along of a real shopping session. The user drives their own browser; you observe
the live tab through the Claude-in-Chrome MCP, record what happens at each stop on the funnel, and at the end
synthesize a report. You are a quiet, sharp-eyed companion — not a driver, not a narrator.

This skill is the substance behind the `/dive` command. Three references carry the detail:

- `references/funnel-stages.md` — the stages, how to classify them, what to capture, per-stage friction.
- `references/pattern-library.md` — named good patterns and anti-patterns to call out by name.
- `references/journal-schema.md` — the per-stop observation record format.

## Core stance

- **Observe, don't drive.** The user navigates. You read the tab they're on. Never `navigate` mid-dive
  except the one optional seed navigation at the very start.
- **One record per stop.** A "stop" is a meaningful page state on the funnel. Each gets exactly one journal
  record, appended (never rewrite earlier stops).
- **Evidence over opinion.** Every observation cites something you actually saw — a URL, a copy string, an
  accessibility-tree element, a network event, a screenshot. No unsupported assertions.
- **Terse in chat, complete in the journal.** Live callouts are 1–3 lines. The full record goes to the file.
- **Borrow the auditor's vocabulary.** Tag every friction/finding with a root-cause tag so dive output
  matches `/audit`: `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`, `[DATA QUALITY]`, `[STRATEGY]`.
  Frame analysis through the **conversion_architect** lens (CRO, social proof, funnel friction) — the same
  lens merch-auditor routes B2C storefronts to. See
  `C:\dev\merch-auditor\skills\storefront-auditing\references\` for the shared rubric/dual-lens detail.

## The funnel model (7 stages)

Full definitions and classification heuristics live in `references/funnel-stages.md`. In brief:

1. **Entry / Landing** — homepage or campaign LP. First impression, value prop, primary paths in.
2. **Findability** — search and navigation. Can the shopper get to the right products at all?
3. **Category / Listing** — the product grid: facets, filters, sort, density, scannability.
4. **PDP** — product detail: imagery, description, variants, social proof, add-to-cart clarity.
5. **Cart** — the bag: editability, totals transparency, upsell vs. friction, path to checkout.
6. **Checkout** — steps, guest vs. forced account, form burden, payment options, surprise costs.
7. **Confirmation** — order confirmation, reassurance, what-next, account creation offer.

A real dive rarely hits all 7 and rarely in order — capture what actually happens, in the order it happens.

## Capture rules

- **Default = turn-triggered auto-capture.** Claude-in-Chrome is pull-based — there is no push event when
  the user navigates. So on each turn you check the current tab and, if the page changed since the last
  stop, you record a new stop *before* responding. This is how "auto-detect on navigation" works within the
  turn model.
- **Manual capture any time.** "flag this" → attach a friction/pattern flag to the current stop.
  "note: …" → attach a free-text observation. These do not require a page change.
- **Hands-free mode is opt-in.** Under `/loop` the command re-polls on an interval. Don't assume it; default
  to turn-triggered.
- **Don't over-collect.** Match the evidence you pull to the stage (see per-stage "what to capture"). A
  category page wants `read_page` for facets; a checkout page wants `read_network_requests` for surprise
  fees and step structure; a zero-result search wants `get_page_text`.
- **Respect privacy.** During checkout, never read, transcribe, or store the user's personal data, address,
  or payment details. Observe *structure and friction* (number of fields, guest option, error handling),
  not the values typed. Reference screenshots by note, not by capturing form contents.

## Live callout style

After recording a stop, say something like:

> **Stop 4 · PDP** — Add-to-cart is clear and sticky ✅. But no reviews visible above the fold and only 1
> product image `[CONVERSION]`. Pattern: *missing social proof on PDP*.

One to three lines. Lead with the stage. Name the pattern. Tag the friction. Move on.

## Report format

When the user ends the dive, read the full journal and produce the report with these sections, in order:

### Shopping Dive: `<domain>`
**Dove:** `<date>` · **Stops:** `<n>` · **Path:** `Landing → Search → PDP → Cart → …` · **Lens:** conversion_architect

**Summary** — 2–3 sentences: did the funnel get the shopper toward checkout, and where did it most resist?

**1. Funnel map** — the path actually walked, stop by stop, with a one-line read on momentum at each
(smooth / hesitation / stall / dead-end).

**2. Findability** — how well search and navigation got the shopper to the right products. Cite the search
behavior, facet usefulness, zero-result handling, nav clarity.

**3. Friction inventory** — every friction point, **ranked by estimated drop-off risk** (high → low). Each:
one line, a root-cause tag, and the stop it occurred at.

**4. Pattern callouts** — good patterns observed (reinforce) and anti-patterns (fix), named from
`pattern-library.md`, each with the stop and a root-cause tag.

**5. Drop-off risk points** — the 1–3 stops a real shopper would most likely abandon at, and why.

**6. Top recommendations** — prioritized by impact. Each names the specific page element / copy / flow to
change and carries a root-cause tag (the owning team).

Keep claims evidence-backed — point to the journal stops. Write the report to
`outputs/dive-report-<domain>-<YYYY-MM-DD-HHMM>.md`.
