---
name: shopping-dive
description: Methodology for riding along as a user shops a storefront — observing the live tab across the funnel, capturing structured per-stop observations (with fast quick-mark notes), calling out named patterns and friction, and synthesizing a dive report with a visual funnel storyboard. Use when running a shopping ride-along / dive, recording an ecommerce shopping session, or analyzing a checkout funnel as it is walked.
---

# Shopping Dive

A **dive** is a live ride-along of a real shopping session. The user drives their own browser; you observe
the live tab through the Claude-in-Chrome MCP, record what happens at each stop on the funnel, and at the end
synthesize a report. You are a quiet, sharp-eyed companion — not a driver, not a narrator.

This skill is the substance behind the `/dive` command. Three references carry the detail:

- `references/funnel-stages.md` — the stages, how to classify them, what to capture, per-stage friction.
- `references/pattern-library.md` — named good patterns and anti-patterns to call out by name.
- `references/journal-schema.md` — the per-stop observation record format, the quick-mark mapping, and the
  session-state sidecar (for pause/resume).

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

## Capture modes

Three ways to capture, all writing the same one-record-per-stop journal. **Turn-triggered is the default.**

- **Turn-triggered (default).** Claude-in-Chrome is pull-based — there is no push event when the user
  navigates. So on each turn you check the current tab and, if the page changed since the last stop, record a
  new stop *before* responding. This is "auto-detect on navigation" within the turn model.
- **Hands-free auto (opt-in).** When the dive runs under `/loop` (e.g. `/loop 45s /dive`), each tick re-reads
  the tab and records a stop if the URL/title changed since `lastSeenUrl` — no typing needed. Document the
  cost/cache caveat to the user once: interval polling burns context, so keep intervals ≥30s. Never default
  to this; the user opts in.
- **Batch.** When several storefront tabs are already open, capture them in one pass, in funnel order:
  enumerate the tab group via `tabs_context_mcp` and record one stop per tab. Useful for catching up after
  shopping ahead.

### Pause / resume

A dive persists so it can be stopped and picked up later, even across restarts. On `/dive` start, check for an
in-progress **session-state sidecar** for the domain (`outputs/.dive-state-<domain>.json`, schema in
`journal-schema.md`). If found, offer **resume** (continue the stop counter, append to the same journal) or
**start fresh** (archive the old state, new journal). Update the sidecar after every stop.

## Quick-marks (fast notes)

The user should be able to drop a note in 1–3 characters without breaking shopping flow. Recognize this
grammar in any message and attach it to the **current stop** (prefix the entry `(manual)`):

| Mark | Means | Lands as |
|---|---|---|
| `! <text>` | friction | a Friction line — **you infer severity** (HIGH/MED/LOW) + a root-cause tag |
| `+ <text>` | something good | a ✅ Pattern line (name it from the library if it fits) or an Observation |
| `? <text>` | open question | a `questions[]` entry → surfaces in the report's **Open Questions** |
| `* <text>` | neutral note | an Observation line |

- **Auto severity + tag.** For a `!` mark with no severity stated, infer it and show your inference inline in
  the callout (e.g. "logged as `[CONVERSION]` HIGH — reply `med`/`low` to change"). One char corrects it.
- **Bare text still works.** Freeform "flag this" / "note: …" / plain sentences are still accepted — the
  marks are a fast path, not a requirement.
- **Screenshot annotation.** If the user shares/pastes a screenshot with a mark, log it as evidence on the
  current stop (reference the image; never transcribe form contents — see privacy).
- **Proactive micro-prompt (default on, one line, suppressible).** After a stop's callout you *may* append a
  single nudge: "anything bug you here? `!`/`+`/skip". If the user says "quiet" / "stop asking", disable it
  for the rest of the dive and never nag again.

## Don't over-collect & privacy

- **Match evidence to the stage.** A category page wants `read_page` for facets; a checkout page wants
  `read_network_requests` for surprise fees and step structure; a zero-result search wants `get_page_text`.
- **Privacy (non-negotiable).** During checkout, never read, transcribe, or store the user's personal data,
  address, or payment details. Observe *structure and friction* (field count, guest option, error handling),
  not the values typed. Screenshots on checkout stops are referenced by note only.

## Live callout style

After recording a stop, say something like:

> **Stop 4 · PDP** — Add-to-cart is clear and sticky ✅. But no reviews visible above the fold and only 1
> product image `[CONVERSION]`. Pattern: *missing social proof on PDP*.

One to three lines. Lead with the stage. Name the pattern. Tag the friction. Move on.

## Per-stop screenshot (for the storyboard)

When capturing a stop, also take a screenshot (Claude-in-Chrome) and save it to
`outputs/dive-<domain>-<ts>/stop-<n>.jpg`; reference the path in the journal `Evidence` field. These feed the
report's visual storyboard. On checkout stops, capture only if it doesn't expose entered personal/payment
data — otherwise skip the image and note why.

## Report format

When the user ends the dive, read the full journal and produce the report with these sections, in order:

### Shopping Dive: `<domain>`
**Dove:** `<date>` · **Stops:** `<n>` · **Path:** `Landing → Search → PDP → Cart → …` · **Lens:** conversion_architect

**Shareable recap** — a compact block sized for pasting into Slack/email: the path walked, stop count, and
the **top 3 frictions** (one line each, tagged). This goes first so a skimmer gets the gist in five seconds.

**Summary** — 2–3 sentences: did the funnel get the shopper toward checkout, and where did it most resist?

**1. Funnel map** — the path actually walked, stop by stop, with a one-line read on momentum at each
(smooth / hesitation / stall / dead-end).

**2. Funnel storyboard** — a labeled visual sequence of the journey: for each stop, the screenshot
(`outputs/dive-<domain>-<ts>/stop-<n>.jpg`) with a caption `Stop <n> · <Stage> · <one-line note>`. If the
Claude-in-Chrome `gif_creator` is available, also assemble an animated GIF of the walked funnel and link it.
Where screenshots are missing, fall back to a **text storyboard** (the captions alone).

**3. Findability** — how well search and navigation got the shopper to the right products. Cite the search
behavior, facet usefulness, zero-result handling, nav clarity.

**4. Friction inventory** — every friction point, **ranked by estimated drop-off risk** (high → low). Each:
one line, a root-cause tag, and the stop it occurred at.

**5. Pattern callouts** — good patterns observed (reinforce) and anti-patterns (fix), named from
`pattern-library.md`, each with the stop and a root-cause tag.

**6. Drop-off risk points** — the 1–3 stops a real shopper would most likely abandon at, and why.

**7. Open questions** — the `?` quick-marks collected during the dive, each tied to its stop, so nothing the
shopper wondered about gets lost.

**8. Top recommendations** — prioritized by impact. Each names the specific page element / copy / flow to
change and carries a root-cause tag (the owning team).

Keep claims evidence-backed — point to the journal stops. Write the report to
`outputs/dive-report-<domain>-<YYYY-MM-DD-HHMM>.md`.
