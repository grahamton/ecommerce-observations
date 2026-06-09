---
description: Ride along as you shop a storefront — record the funnel journey (fast quick-mark notes, pause/resume, hands-free/batch capture) and produce a dive report with a visual storyboard
argument-hint: [storefront-url | "current tab"] [--auto] [--batch]
allowed-tools: mcp__Claude_in_Chrome__list_connected_browsers, mcp__Claude_in_Chrome__tabs_context_mcp, mcp__Claude_in_Chrome__read_page, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__read_network_requests, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__computer, mcp__Claude_in_Chrome__gif_creator, Write, Read, Edit, Bash
---

Ride along as the user shops a storefront and record the funnel journey. You are an **observer, not a
driver** — the user navigates their own tab; you watch the live page, capture structured observations at
each stop, emit terse live callouts, and at the end synthesize a report with a visual storyboard.

**Load the methodology first.** Read `skills/shopping-dive/SKILL.md` (and its `references/`) for the funnel
model, capture modes, quick-mark grammar, stage-classification heuristics, the journal + session-state
schema, the named pattern library, and the report format. Follow it. This command file is the operational
wrapper; the skill is the substance.

`$ARGUMENTS` may contain an optional starting URL plus flags: `--auto` (hands-free, intended to run under
`/loop`) and `--batch` (capture all open storefront tabs in one pass). If empty, ride along on the active tab
in turn-triggered mode.

---

## Step 1 — Attach + check for a dive to resume

1. Call `list_connected_browsers` to confirm the user's Chrome is connected. If none is connected, STOP and
   tell the user to connect the Claude-in-Chrome extension — there is no fallback engine for ride-along.
2. Call `tabs_context_mcp` to enumerate tabs. **Reuse the user's existing shopping tab(s)** — do NOT create a
   fresh MCP tab (this is ride-along, not drive). Record the `tabId`(s).
3. Determine the domain (from the seed URL or the active tab) — it names the output files.
4. **Resume check.** Look for `outputs/.dive-state-<domain>.json` with `status: in_progress` (schema in
   `journal-schema.md`). If found, ask the user: **resume** (load `stopCounter`/`journalPath`, append, add a
   `**Resumed:**` line to the journal header) or **start fresh** (rename the old sidecar to `*.ended.json`,
   begin a new journal). If not found, start fresh.
5. If `$ARGUMENTS` contains a URL and we're starting fresh, `navigate(tabId, url=…)` once to seed; otherwise
   observe the current tab as-is.

---

## Step 2 — Open (or reattach to) the journal + state

On a fresh dive, create `outputs/dive-<domain>-<YYYY-MM-DD-HHMM>.md` with a header, make the screenshot dir
`outputs/dive-<domain>-<YYYY-MM-DD-HHMM>/`, and write the session-state sidecar
`outputs/.dive-state-<domain>.json`. Get the timestamp via Bash: `Get-Date -Format "yyyy-MM-dd-HHmm"`
(PowerShell) or `date +%Y-%m-%d-%H%M`.

Carry session state from the sidecar across turns (`tabId`, `lastSeenUrl`, `stopCounter`, `mode`,
`journalPath`, `screenshotDir`, `microPrompt`). **Update the sidecar after every stop.** Outputs go to
`C:\dev\ecommerce-observations\outputs\` directly (local Windows — do NOT use the cloud `/sessions` path
trick from merch-auditor's audit.md).

Set `mode`: `--auto` → `auto`, `--batch` → `batch`, else `turn`.

---

## Step 3 — The capture loop

Capture works the same in all modes — the only difference is what triggers a check:

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
   - `! <text>` → Friction (infer severity + tag, show your inference so one char can correct it)
   - `+ <text>` → ✅ Pattern (name it from the library if it fits)
   - `? <text>` → Questions (rolls up into the report's Open Questions)
   - `* <text>` → Observation
   - a shared/pasted screenshot + mark → log as evidence on the current stop
   - freeform "flag this" / "note: …" still works.
4. **If the user says `end dive` / `report` / `wrap up`** → set sidecar `status: ended` and go to Step 4.

Keep it light and non-spammy. You are a quiet companion, not a running monologue.

---

## Step 4 — Synthesize the dive report

When the dive ends, read back the full journal and synthesize the report per the **Report format** section of
`SKILL.md`. It must contain, in order:

- **Shareable recap** — compact block for Slack/email: path walked, stop count, top-3 frictions (tagged).
- **Summary** — 2–3 sentences.
- **Funnel map** — the path walked, stop by stop, with where momentum stalled.
- **Funnel storyboard** — labeled screenshot sequence (`<screenshotDir>/stop-<n>.jpg` + caption per stop). If
  `gif_creator` is available, also assemble an animated GIF of the walked funnel and link it. Where shots are
  missing, fall back to a text storyboard (captions only).
- **Findability assessment** — could the user find what they wanted (search + nav)?
- **Friction inventory** — ranked by drop-off risk, each tagged.
- **Pattern callouts** — good + anti, named from `pattern-library.md`.
- **Drop-off risk points** — the stops most likely to lose a real shopper.
- **Open questions** — the `?` quick-marks, each tied to its stop.
- **Top recommendations** — specific, each naming the element + owning team via tag.

Write it to `outputs/dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` and give the user a 3–4 line summary plus the
file path.
