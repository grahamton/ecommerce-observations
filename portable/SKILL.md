---
name: shopping-dive
description: Ride along as a user shops a storefront — observe each page across the funnel (landing → search/nav → category → PDP → cart → checkout), capture one structured observation per stop (with fast quick-mark notes), call out named patterns and friction, and synthesize a dive report with a funnel storyboard. Use when running a shopping ride-along / dive, recording an ecommerce shopping session, or analyzing a checkout funnel as it is walked.
---

# Shopping Dive (portable)

A **dive** is a live ride-along of a real shopping session. The user drives — they navigate their own
browser; you **observe the page(s) they're on**, record what happens at each stop on the funnel, and at the
end synthesize a report. You are a quiet, sharp-eyed companion — not a driver, not a narrator.

This is the platform-neutral version. It assumes only one capability: **you can see the web page the user is
currently looking at** (a live tab you can read, or page content/screenshots they share with you). It works
in any agent that can browse or be shown the active page — e.g. Gemini in Chrome, Copilot Cowork, or any
browser agent. The Claude Code plugin version of this same methodology adds file-writing, automatic
per-stop screenshots, and turn-triggered/hands-free capture; everything analytical here is identical.

Three references carry the detail:
- `references/funnel-stages.md` — the 7 stages, how to classify them, what to look at, per-stage friction.
- `references/pattern-library.md` — named good patterns and anti-patterns to call out by name.
- `references/journal-schema.md` — the per-stop observation record format + the quick-mark mapping.

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
3. **Quick-marks / manual notes** — see below; attach to the current stop, no page change required.
4. **End** — when the user says "end dive" / "report", synthesize the report (format below) and output it
   (write the file if you can, otherwise print the full journal + report as markdown to copy).

> Cadence note: if your platform has no automatic navigation event (most browser skills don't), the
> "one record per stop" loop is driven by the user re-invoking you / sharing the new page as they move
> through the funnel. Same outcome, manual cadence.

### Multi-tab batch (great for Gemini in Chrome)

If the user shares several open tabs at once (Gemini supports up to 10), capture them **in one pass, in
funnel order** — one stop record per tab. This is the portable equivalent of the plugin's batch mode and is
the fastest way to record a journey the user shopped ahead on.

### Pause / resume

If you can write files, persist a session-state sidecar (see `journal-schema.md`) so a dive can be paused and
resumed — on start, offer to resume an in-progress dive or start fresh. If you can't write files (e.g. a
single-prompt skill), the journal-in-conversation already gives implicit resume *within* the session — just
keep appending; there's no cross-session resume.

## Quick-marks (fast notes)

The user should be able to drop a note in 1–3 characters. Recognize this grammar and attach it to the
**current stop** (prefix `(manual)`):

| Mark | Means | Lands as |
|---|---|---|
| `! <text>` | friction | a Friction line — **you infer severity** (HIGH/MED/LOW) + a root-cause tag |
| `+ <text>` | something good | a ✅ Pattern line (name from the library if it fits) or an Observation |
| `? <text>` | open question | a Questions entry → surfaces in the report's **Open Questions** |
| `* <text>` | neutral note | an Observation line |

- For a `!` mark with no severity, infer it and show your inference so one char corrects it.
- Bare text / "flag this" / "note: …" still works — the marks are a fast path, not a requirement.
- If the user shares a screenshot with a mark, log it as evidence on the current stop.
- **Micro-prompt (default on, one line, suppressible):** after a stop you may append "anything bug you here?
  `!`/`+`/skip". If the user says "quiet", stop asking for the rest of the dive.

## Privacy rule (non-negotiable)

On checkout pages, observe **structure and friction only** — field count, guest-vs-forced-account, surprise
fees, error handling, step structure. **Never read, transcribe, or store** the user's personal data,
address, or payment details. Describe the form, not the values typed. Don't capture screenshots that expose
entered personal/payment data.

## Live callout style

After recording a stop, say something like:

> **Stop 4 · PDP** — Add-to-cart is clear and sticky ✅. But no reviews above the fold and only 1 product
> image `[CONVERSION]`. Pattern: *missing social proof on PDP*.

One to three lines. Lead with the stage. Name the pattern. Tag the friction. Move on.

## Report format

When the user ends the dive, read the full journal and produce the report with these sections, in order:

### Shopping Dive: `<domain>`
**Dove:** `<date>` · **Stops:** `<n>` · **Path:** `Landing → Search → PDP → Cart → …` · **Lens:** conversion

**Shareable recap** — a compact block for pasting into Slack/email: path walked, stop count, and the **top 3
frictions** (one line each, tagged). First, so a skimmer gets the gist in five seconds.

**Summary** — 2–3 sentences: did the funnel get the shopper toward checkout, and where did it most resist?

**1. Funnel map** — the path actually walked, stop by stop, with a one-line read on momentum at each
(smooth / hesitation / stall / dead-end).

**2. Funnel storyboard** — a labeled sequence of the journey: for each stop, the screenshot the user shared
(if any) with a caption `Stop <n> · <Stage> · <one-line note>`. If you have no images, produce a **text
storyboard** (the captions alone). (Animated-GIF assembly exists only where the agent has image tooling —
the Claude plugin; skip it elsewhere.)

**3. Findability** — how well search and navigation got the shopper to the right products. Cite the search
behavior, facet usefulness, zero-result handling, nav clarity.

**4. Friction inventory** — every friction point, **ranked by estimated drop-off risk** (high → low). Each:
one line, a root-cause tag, and the stop it occurred at.

**5. Pattern callouts** — good patterns observed (reinforce) and anti-patterns (fix), named from
`pattern-library.md`, each with the stop and a root-cause tag.

**6. Drop-off risk points** — the 1–3 stops a real shopper would most likely abandon at, and why.

**7. Open questions** — the `?` quick-marks collected during the dive, each tied to its stop.

**8. Top recommendations** — prioritized by impact. Each names the specific page element / copy / flow to
change and carries a root-cause tag (the owning team).

Keep claims evidence-backed — point to the journal stops.
