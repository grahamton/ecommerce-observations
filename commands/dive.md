---
description: Ride along as you shop a storefront — record the funnel journey and produce a dive report
argument-hint: [storefront-url | "current tab"]
allowed-tools: mcp__Claude_in_Chrome__list_connected_browsers, mcp__Claude_in_Chrome__tabs_context_mcp, mcp__Claude_in_Chrome__read_page, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__read_network_requests, mcp__Claude_in_Chrome__navigate, Write, Read, Edit, Bash
---

Ride along as the user shops a storefront and record the funnel journey. You are an **observer, not a
driver** — the user navigates their own tab; you watch the live page, capture structured observations at
each stop, emit terse live callouts, and at the end synthesize a report.

**Load the methodology first.** Read `skills/shopping-dive/SKILL.md` (and its `references/`) for the funnel
model, capture rules, stage-classification heuristics, the journal record schema, the named pattern
library, and the report format. Follow it. This command file is the operational wrapper; the skill is the
substance.

`$ARGUMENTS` is an optional starting URL. If empty, ride along on whatever tab is already active.

---

## Step 1 — Attach to the live tab (once, at dive start)

1. Call `list_connected_browsers` to confirm the user's Chrome is connected. If none is connected, STOP and
   tell the user to connect the Claude-in-Chrome extension — there is no fallback engine for ride-along.
2. Call `tabs_context_mcp` to enumerate tabs. **Reuse the user's existing shopping tab** — do NOT create a
   fresh MCP tab (this is ride-along, not drive). Record its `tabId`.
3. If `$ARGUMENTS` contains a URL, `navigate(tabId, url=$ARGUMENTS)` once to seed the dive; otherwise observe
   the current tab as-is.
4. Note the domain — it names the output files.

---

## Step 2 — Open the journal

Create `outputs/dive-<domain>-<YYYY-MM-DD-HHMM>.md` with a header (domain, start time, "live journal").
Get the timestamp via Bash: `Get-Date -Format "yyyy-MM-dd-HHmm"` (PowerShell) or `date +%Y-%m-%d-%H%M`.

Hold session state across turns: `tabId`, `lastSeenUrl`, `stopCounter`, and a running list of stops. Outputs
go to `C:\dev\ecommerce-observations\outputs\` directly (local Windows — do NOT use the cloud `/sessions`
path trick from merch-auditor's audit.md).

---

## Step 3 — The capture loop (every turn the user hands back)

Each time control returns to you (the user types "next", "ok", a note, or anything):

1. **Detect navigation.** Read the current tab's URL/title (cheap: `tabs_context_mcp`, or `read_page` with a
   small depth). Compare to `lastSeenUrl`.
2. **If the page changed** → it's a new stop:
   - Classify the funnel stage using the heuristics in `references/funnel-stages.md`.
   - Gather evidence appropriate to the stage: `read_page` (nav/facets/interactive elements →
     findability + flow), `get_page_text` (copy, zero-result/error/empty states), and
     `read_network_requests` (analytics/dataLayer + add-to-cart/checkout XHRs → funnel instrumentation).
     Don't over-collect — pull only what the stage calls for.
   - Build one observation record per `references/journal-schema.md` and **append it to the journal file**
     (Edit/append, never rewrite prior stops).
   - Emit a **terse live callout** (1–3 lines): stage, the one or two things that matter, any named pattern
     or friction with its root-cause tag. Do not dump the whole record into chat.
3. **If the page did NOT change** → handle the user's intent: a manual `flag this` / `note: …` becomes a
   manual observation or pattern flag against the current stop; a question gets answered from what you see.
4. **If the user says `end dive` / `report` / `wrap up`** → go to Step 4.

Keep it light and non-spammy. You are a quiet companion, not a running monologue.

---

## Step 4 — Synthesize the dive report

When the dive ends, read back the full journal and synthesize the report per the **Report format** section
of `SKILL.md`. It must contain:

- **Funnel map** — the path actually walked, stage by stage, with where momentum stalled.
- **Findability assessment** — could the user find what they wanted (search + nav)?
- **Friction inventory** — ranked by estimated drop-off risk, each tagged with a root-cause tag.
- **Pattern callouts** — good patterns observed and anti-patterns, named from `pattern-library.md`.
- **Drop-off risk points** — the stops most likely to lose a real shopper.
- **Top recommendations** — specific, each naming the page element and owning team via root-cause tag.

Write it to `outputs/dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` and give the user a 3–4 line summary plus the
file path.
