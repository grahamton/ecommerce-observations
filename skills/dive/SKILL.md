---
name: dive
description: Ride along as the user shops a storefront — observe their live Chrome tab across the funnel (landing → search/nav → category → PDP → cart → checkout), capture one structured observation per stop with fast quick-mark notes, call out named patterns and friction, and synthesize a dive report with a visual funnel storyboard. Use when the user runs a shopping ride-along / dive, records an ecommerce shopping session, or analyzes a checkout funnel as it is walked.
argument-hint: [storefront-url | "current tab"] [--auto] [--batch]
disable-model-invocation: true
allowed-tools: mcp__Claude_in_Chrome__list_connected_browsers, mcp__Claude_in_Chrome__tabs_context_mcp, mcp__Claude_in_Chrome__read_page, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__read_network_requests, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__computer, mcp__Claude_in_Chrome__gif_creator, Write, Read, Edit, Bash
---

# Shopping Dive

A **dive** is a live ride-along of a real shopping session. The user drives their own browser; you observe
their live Chrome tab through the Claude-in-Chrome MCP, record what happens at each stop on the funnel, and at
the end synthesize a report. You are a quiet, sharp-eyed companion — **an observer, not a driver, not a
narrator**.

`$ARGUMENTS` may contain an optional starting URL plus flags: `--auto` (hands-free, intended to run under
`/loop`) and `--batch` (capture all open storefront tabs in one pass). If empty, ride along on the active tab
in turn-triggered mode.

Three reference files carry the detail — read them as needed:
- `references/funnel-stages.md` — the 7 stages, how to classify them, what to capture, per-stage friction.
- `references/pattern-library.md` — named good patterns and anti-patterns to call out by name.
- `references/journal-schema.md` — the per-stop record format, the quick-mark mapping, and the session-state
  sidecar (for pause/resume).

## Core stance

- **Observe, don't drive.** The user navigates. You read the tab they're on. Never `navigate` mid-dive
  except the one optional seed navigation at the very start.
- **One record per stop, append-only.** A "stop" is a meaningful page state on the funnel. Each gets exactly
  one journal record, appended — never rewrite earlier stops.
- **Evidence over opinion.** Every observation cites something you actually saw — a URL, a copy string, an
  accessibility-tree element, a network event, a screenshot. No unsupported assertions.
- **Terse in chat, complete in the journal.** Live callouts are 1–3 lines. The full record goes to the file.
- **Borrow the auditor's vocabulary.** Tag every friction/finding with a root-cause tag so dive output
  matches `/audit`: `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`, `[DATA QUALITY]`, `[STRATEGY]`.
  Frame analysis through the **conversion_architect** lens (CRO, social proof, funnel friction) — the same
  lens merch-auditor routes B2C storefronts to. See
  `C:\dev\merch-auditor\skills\storefront-auditing\references\` for the shared rubric/dual-lens detail.

## The funnel model (7 stages)

Full definitions and classification heuristics live in `references/funnel-stages.md`. In brief: **Entry /
Landing → Findability (search + nav) → Category / Listing → PDP → Cart → Checkout → Confirmation.** A real
dive rarely hits all 7 and rarely in order — capture what actually happens, in the order it happens.

---

## Step 1 — Attach + check for a dive to resume

1. Call `list_connected_browsers` to confirm the user's Chrome is connected. If none is connected, STOP and
   tell the user to connect the Claude-in-Chrome extension — there is no fallback engine for ride-along.
2. Call `tabs_context_mcp` to enumerate tabs. **Reuse the user's existing shopping tab(s)** — do NOT create a
   fresh MCP tab (this is ride-along, not drive). Record the `tabId`(s).
3. Determine the domain (from the seed URL or the active tab) — it names the output files.
4. **Resume check.** Look for `outputs/.dive-state-<domain>.json` with `status: in_progress` (schema in
   `references/journal-schema.md`). If found, ask the user: **resume** (load `stopCounter`/`journalPath`,
   append, add a `**Resumed:**` line to the journal header) or **start fresh** (rename the old sidecar to
   `*.ended.json`, begin a new journal). If not found, start fresh.
5. If `$ARGUMENTS` contains a URL and we're starting fresh, `navigate(tabId, url=…)` once to seed; otherwise
   observe the current tab as-is.

## Step 2 — Open (or reattach to) the journal + state

On a fresh dive, create `outputs/dive-<domain>-<YYYY-MM-DD-HHMM>.md` with a header, make the screenshot dir
`outputs/dive-<domain>-<YYYY-MM-DD-HHMM>/`, and write the session-state sidecar
`outputs/.dive-state-<domain>.json`. Get the timestamp via Bash: `Get-Date -Format "yyyy-MM-dd-HHmm"`
(PowerShell) or `date +%Y-%m-%d-%H%M`.

Carry session state from the sidecar across turns (`tabId`, `lastSeenUrl`, `stopCounter`, `mode`,
`journalPath`, `screenshotDir`, `microPrompt`). **Update the sidecar after every stop.** Outputs go to
`C:\dev\ecommerce-observations\outputs\` directly (local Windows — do NOT use the cloud `/sessions` path
trick from merch-auditor's audit.md). Set `mode`: `--auto` → `auto`, `--batch` → `batch`, else `turn`.

## Step 3 — The capture loop

Same loop in all three modes; only the trigger differs:

- **turn** (default): each time control returns to you (user types "next", a quick-mark, anything).
- **auto** (`--auto`, run under `/loop` e.g. `/loop 45s /dive --auto`): each loop tick. Tell the user once
  that auto-polling burns context, so keep intervals ≥30s.
- **batch** (`--batch`): iterate every open storefront tab from `tabs_context_mcp` in funnel order, recording
  one stop per tab in a single pass.

On each check:

1. **Detect navigation.** Read the current tab's URL/title (cheap: `tabs_context_mcp`, or `read_page` with a
   small depth). Compare to `lastSeenUrl`.
2. **If the page changed** → it's a new stop:
   - Classify the funnel stage using `references/funnel-stages.md`.
   - Gather evidence appropriate to the stage: `read_page` (nav/facets/interactive elements), `get_page_text`
     (copy, zero-result/error states), `read_network_requests` (analytics + add-to-cart/checkout XHRs). Don't
     over-collect — pull only what the stage calls for.
   - **Take a screenshot** (`computer`, screenshot action) and save it to `<screenshotDir>/stop-<n>.jpg` for
     the storyboard. **Skip the screenshot on checkout stops if it would expose entered personal/payment
     data** — note why instead.
   - Build one record per `references/journal-schema.md` and **append it to the journal** (never rewrite prior
     stops). Increment `stopCounter`; update the sidecar.
   - Emit a **terse live callout** (1–3 lines): stage, the one or two things that matter, any named pattern or
     friction with its tag. Don't dump the whole record into chat.
   - **Micro-prompt** (if `microPrompt` is true): append one optional line — "anything bug you here?
     `!`/`+`/skip". If the user ever says "quiet"/"stop asking", set `microPrompt: false` and never nag again.
3. **Quick-marks / manual notes** (no page change needed): recognize the grammar and attach to the current
   stop, prefixed `(manual)` —
   - `! <text>` → Friction (infer severity HIGH/MED/LOW + a root-cause tag; show your inference so one char
     can correct it)
   - `+ <text>` → ✅ Pattern (name it from the library if it fits)
   - `? <text>` → Questions (rolls up into the report's Open Questions)
   - `* <text>` → Observation
   - a shared/pasted screenshot + mark → log as evidence on the current stop
   - freeform "flag this" / "note: …" still works.
4. **If the user says `end dive` / `report` / `wrap up`** → set sidecar `status: ended` and go to Step 4.

Keep it light and non-spammy. You are a quiet companion, not a running monologue.

### Privacy (non-negotiable)

During checkout, never read, transcribe, or store the user's personal data, address, or payment details.
Observe *structure and friction* (field count, guest option, error handling), not the values typed. Skip any
screenshot that would expose entered personal/payment data.

### Live callout style

> **Stop 4 · PDP** — Add-to-cart is clear and sticky ✅. But no reviews above the fold and only 1 product
> image `[CONVERSION]`. Pattern: *missing social proof on PDP*.

One to three lines. Lead with the stage. Name the pattern. Tag the friction. Move on.

## Step 4 — Synthesize the dive report

When the dive ends, read back the full journal and produce the report with these sections, in order:

### Shopping Dive: `<domain>`
**Dove:** `<date>` · **Stops:** `<n>` · **Path:** `Landing → Search → PDP → Cart → …` · **Lens:** conversion_architect

**Shareable recap** — a compact block sized for pasting into Slack/email: the path walked, stop count, and
the **top 3 frictions** (one line each, tagged). First, so a skimmer gets the gist in five seconds.

**Summary** — 2–3 sentences: did the funnel get the shopper toward checkout, and where did it most resist?

**1. Funnel map** — the path actually walked, stop by stop, with a one-line read on momentum at each
(smooth / hesitation / stall / dead-end).

**2. Funnel storyboard** — a labeled visual sequence: for each stop, the screenshot
(`<screenshotDir>/stop-<n>.jpg`) with caption `Stop <n> · <Stage> · <one-line note>`. If the Claude-in-Chrome
`gif_creator` is available, also assemble an animated GIF of the walked funnel and link it. Where screenshots
are missing, fall back to a **text storyboard** (captions alone).

**3. Findability** — how well search and navigation got the shopper to the right products. Cite search
behavior, facet usefulness, zero-result handling, nav clarity.

**4. Friction inventory** — every friction point, **ranked by estimated drop-off risk** (high → low). Each:
one line, a root-cause tag, and the stop it occurred at.

**5. Pattern callouts** — good patterns observed (reinforce) and anti-patterns (fix), named from
`references/pattern-library.md`, each with the stop and a root-cause tag.

**6. Drop-off risk points** — the 1–3 stops a real shopper would most likely abandon at, and why.

**7. Open questions** — the `?` quick-marks collected during the dive, each tied to its stop.

**8. Top recommendations** — prioritized by impact. Each names the specific page element / copy / flow to
change and carries a root-cause tag (the owning team).

Keep claims evidence-backed — point to the journal stops. Write the report to
`outputs/dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` and give the user a 3–4 line summary plus the file path.
