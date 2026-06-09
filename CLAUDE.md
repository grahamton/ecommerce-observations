# ecommerce-observations

A Claude Code plugin: a **live ride-along recorder/reporter** for ecommerce shopping. The user shops a real
storefront in their own Chrome; the `/dive` command observes the live tab and records the funnel journey,
then synthesizes a report.

This is the **journey** counterpart to the **snapshot** tooling in the merch ecosystem. Keep that division
clear when working here.

## How this fits with the merch ecosystem

| Project | Path | Unit of analysis | Engine |
|---|---|---|---|
| **merch-connector** (MCP) | `C:\dev\merchGent` | one page, static | Puppeteer / Firecrawl |
| **merch-auditor** (plugin) | `C:\dev\merch-auditor` | one page, scored 4-dim | wraps merch-connector |
| **ecommerce-observations** (this) | `C:\dev\ecommerce-observations` | a journey across the funnel | Claude-in-Chrome (live tab) |

When deciding where work belongs: anything *static / single-page / scored* is merch-auditor or
merch-connector. Anything *longitudinal / multi-page / flow-and-friction* is here.

## Architecture

- **Observe, don't drive.** `/dive` reads the user's live tab via the **Claude-in-Chrome MCP**
  (`mcp__Claude_in_Chrome__*`). It never drives the browser except one optional seed `navigate` at dive
  start. There is no fallback engine — if the Chrome extension isn't connected, the dive can't run.
- **Pull-based, so turn-triggered capture.** Claude-in-Chrome has no navigation push event. The capture
  loop runs each turn: read current tab → if the page changed since the last stop, record a new stop. This
  is how "auto-detect on navigation" is achieved within the turn model. Hands-free polling under `/loop` is
  opt-in, not the default.
- **One record per stop, append-only.** The live journal accumulates one record per funnel stop and is never
  rewritten mid-dive. The final report synthesizes from it.

## Funnel model (7 stages)

Entry/Landing → Findability (search+nav) → Category/Listing → PDP → Cart → Checkout → Confirmation. Real
dives skip stages and loop back; capture what actually happens, in order. Definitions, URL/structure
classification heuristics, and per-stage friction live in
`skills/shopping-dive/references/funnel-stages.md`.

## Shared vocabulary (do not diverge from merch-auditor)

Dive output reuses merch-auditor's tags and lens so findings read consistently across tools:

- **Root-cause tags:** `[DATA QUALITY]`, `[FRONTEND INTEGRATION]`, `[UX DESIGN]`, `[STRATEGY]`,
  plus `[CONVERSION]` for funnel-specific friction.
- **Lens:** conversion_architect (CRO, social proof, funnel friction) — see
  `C:\dev\merch-auditor\skills\storefront-auditing\references\` for the shared rubric.
- **Named patterns:** call (anti-)patterns out by their library name —
  `skills/shopping-dive/references/pattern-library.md`.

## Privacy rule (non-negotiable)

On checkout stops, observe **structure and friction only** — field count, guest-vs-forced-account, surprise
fees, error handling, step structure. **Never read, transcribe, or store** the user's personal data,
address, or payment details. Reference screenshots by note, not by capturing form contents.

## Layout

```
.claude-plugin/plugin.json        manifest + /dive registration
commands/dive.md                  the /dive command workflow (operational wrapper)
skills/shopping-dive/SKILL.md     methodology (the substance — loaded by the command)
skills/shopping-dive/references/
  funnel-stages.md                stage definitions, classification, per-stage friction
  pattern-library.md              named good patterns + anti-patterns
  journal-schema.md               per-stop observation record format
outputs/                          generated journals + reports (gitignored)
```

## Output conventions

Written to `outputs/` (local Windows path — do **not** use merch-auditor's cloud `/sessions/*/mnt` path
trick from its `audit.md`):

- `dive-<domain>-<YYYY-MM-DD-HHMM>.md` — live journal, one record per stop.
- `dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` — synthesized report at dive end.

## Working on the plugin

- Edit files here; this directory **is** the source of truth (unlike merch-auditor's mirror setup).
- After changes, reinstall the plugin via the local plugin/marketplace upload and restart so `/dive`
  re-registers. The command isn't invocable until installed.
- To validate before installing: dry-run the workflow by hand (manually walk the SKILL.md steps against a
  named site) to check the journal/report output.
- `git init` is done; no remote yet.
