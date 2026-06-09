---
name: shopping-dive
description: Ride along as a user shops a storefront — observe each page across the funnel (landing → search/nav → category → PDP → cart → checkout), capture one structured observation per stop, call out named patterns and friction, and synthesize a dive report. Use when running a shopping ride-along / dive, recording an ecommerce shopping session, or analyzing a checkout funnel as it is walked.
---

# Shopping Dive (portable)

A **dive** is a live ride-along of a real shopping session. The user drives — they navigate their own
browser; you **observe the page(s) they're on**, record what happens at each stop on the funnel, and at the
end synthesize a report. You are a quiet, sharp-eyed companion — not a driver, not a narrator.

This is the platform-neutral version. It assumes only one capability: **you can see the web page the user is
currently looking at** (a live tab you can read, or page content/screenshots they share with you). It works
in any agent that can browse or be shown the active page — e.g. Gemini in Chrome, Copilot Cowork, or any
browser agent. The Claude Code plugin version of this same methodology adds file-writing and
turn-triggered capture; everything analytical here is identical.

Three references carry the detail:
- `references/funnel-stages.md` — the 7 stages, how to classify them, what to look at, per-stage friction.
- `references/pattern-library.md` — named good patterns and anti-patterns to call out by name.
- `references/journal-schema.md` — the per-stop observation record format.

If your environment can't load reference files (e.g. a single-prompt skill), use the flattened
all-in-one prompt instead — it inlines these.

## Core stance

- **Observe, don't drive.** The user navigates. You read the page they're on. Don't click or navigate for
  them unless they explicitly ask.
- **One record per stop.** A "stop" is a meaningful page state on the funnel. Each gets exactly one journal
  record, appended — never rewrite earlier stops.
- **Evidence over opinion.** Every observation cites something you actually saw — the URL, a copy string, a
  button/label, a layout fact. No unsupported assertions.
- **Terse in chat, complete in the journal.** Live callouts are 1–3 lines. The full record goes in the
  journal (a file if your environment allows; otherwise kept in the conversation).
- **Consistent vocabulary.** Tag every friction/finding with a root-cause tag: `[UX DESIGN]`,
  `[CONVERSION]`, `[FRONTEND INTEGRATION]`, `[DATA QUALITY]`, `[STRATEGY]`. Frame analysis through a
  conversion lens (CRO, social proof, funnel friction).

## How a dive runs (any platform)

1. **Start** — the user names a storefront or is already on one. Note the domain; open a journal (a file if
   you can write one, otherwise a running section in this conversation).
2. **Each stop** — when the user moves to a new page (they re-invoke you, share a new tab, or tell you
   they've moved): read the current page, classify the funnel stage (see `funnel-stages.md`), build one
   observation record (see `journal-schema.md`), append it, and give a 1–3 line live callout naming any
   pattern or friction with its tag.
3. **Manual flags** — "flag this" / "note: …" attaches an observation or pattern flag to the current stop,
   no page change required.
4. **End** — when the user says "end dive" / "report", synthesize the report (format below) and output it
   (write the file if you can, otherwise print the full journal + report as markdown to copy).

> Cadence note: if your platform has no automatic navigation event (most browser skills don't), the
> "one record per stop" loop is driven by the user re-invoking you / sharing the new page as they move
> through the funnel. Same outcome, manual cadence.

## The funnel model (7 stages)

Full definitions and classification heuristics live in `references/funnel-stages.md`. In brief:

1. **Entry / Landing** — homepage or campaign LP. First impression, value prop, primary paths in.
2. **Findability** — search and navigation. Can the shopper get to the right products at all?
3. **Category / Listing** — the product grid: facets, filters, sort, density, scannability.
4. **PDP** — product detail: imagery, description, variants, social proof, add-to-cart clarity.
5. **Cart** — the bag: editability, totals transparency, upsell vs. friction, path to checkout.
6. **Checkout** — steps, guest vs. forced account, form burden, payment options, surprise costs.
7. **Confirmation** — order confirmation, reassurance, what-next, account-creation offer.

Real dives skip stages and loop back — capture what actually happens, in the order it happens.

## Privacy rule (non-negotiable)

On checkout pages, observe **structure and friction only** — field count, guest-vs-forced-account, surprise
fees, error handling, step structure. **Never read, transcribe, or store** the user's personal data,
address, or payment details. Describe the form, not the values typed.

## Live callout style

After recording a stop, say something like:

> **Stop 4 · PDP** — Add-to-cart is clear and sticky ✅. But no reviews above the fold and only 1 product
> image `[CONVERSION]`. Pattern: *missing social proof on PDP*.

One to three lines. Lead with the stage. Name the pattern. Tag the friction. Move on.

## Report format

When the user ends the dive, read the full journal and produce the report with these sections, in order:

### Shopping Dive: `<domain>`
**Dove:** `<date>` · **Stops:** `<n>` · **Path:** `Landing → Search → PDP → Cart → …` · **Lens:** conversion

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

Keep claims evidence-backed — point to the journal stops.
