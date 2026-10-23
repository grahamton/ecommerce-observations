# ecommerce-observations

A Claude Code plugin: a **live ride-along recorder/reporter** for ecommerce shopping. The user shops a real
storefront in their own Chrome; the `/dive` command observes the live tab and records the funnel journey,
then synthesizes a report.

This is the **journey** counterpart to the **snapshot** tooling in the merch ecosystem. Keep that division
clear when working here.

## How this fits with the merch ecosystem

| Project | Unit of analysis | Engine |
|---|---|---|
| **merch-connector** (MCP, separate private repo) | one page, static | Puppeteer / Firecrawl |
| **merch-auditor** (plugin, separate private repo) | one page, scored 4-dim | wraps merch-connector |
| **ecommerce-observations** (this, public) | a journey across the funnel | Claude-in-Chrome (live tab) |

When deciding where work belongs: anything *static / single-page / scored* is merch-auditor or
merch-connector. Anything *longitudinal / multi-page / flow-and-friction* is here.

## Architecture

- **Observe by default.** `/dive` reads the user's live tab via the **Claude-in-Chrome MCP**
  (`mcp__Claude_in_Chrome__*`). In the default modes it never drives the browser except one optional seed
  `navigate` at dive start; `--drive` (opt-in) lets the agent shop, gated by `--checkin`. There is no fallback engine — if the Chrome extension isn't connected, the dive can't run.
- **Pull-based, so turn-triggered capture.** Claude-in-Chrome has no navigation push event. The capture
  loop runs each turn: read current tab → if the page changed since the last stop, record a new stop. This
  is how "auto-detect on navigation" is achieved within the turn model. Three modes share this loop:
  `turn` (default), `auto` (`--auto` under `/loop`), `batch` (`--batch`, all open tabs in one pass).
- **One record per stop, append-only.** The live journal accumulates one record per funnel stop and is never
  rewritten mid-dive. The final report synthesizes from it.
- **Quick-marks for fast notes.** A 1–3 char grammar (`!` friction, `+` good, `?` question, `*` note) maps
  to journal fields; `!` marks get inferred severity + root-cause tag. See SKILL.md / journal-schema.md.
- **Pause/resume via a sidecar.** `outputs/.dive-state-<domain>.json` tracks `stopCounter`/`lastSeenUrl`/etc.
  so `/dive` can resume an in-progress dive across restarts.
- **Per-stop screenshots → storyboard.** Each stop is captured to `outputs/dive-<domain>-<ts>/stop-<n>.jpg`
  and assembled into a visual storyboard (+ optional `gif_creator` GIF) in the report.

## Funnel model (7 stages)

Entry/Landing → Findability (search+nav) → Category/Listing → PDP → Cart → Checkout → Confirmation. Real
dives skip stages and loop back; capture what actually happens, in order. Definitions, URL/structure
classification heuristics, and per-stage friction live in
`skills/dive/references/funnel-stages.md`.

## Shared vocabulary (do not diverge from merch-auditor)

Dive output reuses merch-auditor's tags and lens so findings read consistently across tools:

- **Root-cause tags:** `[DATA QUALITY]`, `[FRONTEND INTEGRATION]`, `[UX DESIGN]`, `[STRATEGY]`,
  plus `[CONVERSION]` for funnel-specific friction.
- **Lens:** conversion_architect (CRO, social proof, funnel friction). The plugin carries its own inline
  summary of the lens and severity criteria (SKILL.md, journal-schema.md) — never reference the private
  merch-auditor files from shipped skill content; public users don't have them.
- **Named patterns:** call (anti-)patterns out by their library name —
  `skills/dive/references/pattern-library.md`.

## Privacy rule (non-negotiable)

On checkout stops, observe **structure and friction only** — field count, guest-vs-forced-account, surprise
fees, error handling, step structure. **Never read, transcribe, or store** the user's personal data,
address, or payment details. Reference screenshots by note, not by capturing form contents.

## Layout

```
.claude-plugin/plugin.json        manifest (skills auto-discovered; no commands array)
skills/dive/SKILL.md              the /dive skill — workflow + methodology + report (the substance)
skills/dive/references/
  funnel-stages.md                stage definitions, classification, per-stage friction
  pattern-library.md              named good patterns + anti-patterns
  journal-schema.md               per-stop record format + sidecar + site-notes schema
skills/sweep/SKILL.md             experimental lean variant (not mirrored to portable/)
portable/                         platform-neutral version (Gemini in Chrome / Copilot Cowork)
test-scenarios/dive-scenario.md   manual acceptance walkthrough (use a public store, no client sites)
outputs/                          generated journals, reports, site-notes/ (gitignored)
```

> Format: `/dive` is a **skill** (`skills/dive/SKILL.md`) — the recommended Claude Code format. Migrated from
> the legacy `commands/dive.md` in v0.3 (commands merged into skills; same `/dive` invocation). Plugin skills
> auto-discover from `skills/`, so no `commands` array is needed in `plugin.json`.

## Output conventions

Written to the environment's outputs location (a local `outputs/` folder or the session outputs dir —
detected at dive start, never hardcoded):

- `dive-<domain>-<YYYY-MM-DD-HHMM>.md` — live journal, one record per stop.
- `dive-<domain>-<YYYY-MM-DD-HHMM>/stop-<n>.jpg` — per-stop screenshots for the storyboard.
- `dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` — synthesized report at dive end.
- `.dive-state-<domain>.json` — session-state sidecar for pause/resume (all `outputs/*` is gitignored).
- `site-notes/<domain>.md` — durable per-domain facts, appended at dive end, loaded at dive start.

## Parity: keep portable in sync

Every methodology change must land in **both** `skills/dive/` (Claude) and `portable/` (Gemini
single-prompt + Cowork foldered skill). Portable degrades gracefully where a capability needs Claude-only
tooling (auto-poll, file-based resume, GIF storyboard → manual/within-session/text).

## Working on the plugin

- Edit files here; this repo **is** the source of truth.
- This repo is public-facing: no client/employer site data, no machine-specific paths, no personal data in
  committed files. Per-site knowledge belongs in `outputs/site-notes/` (gitignored).
- After changes, reinstall the plugin (`/plugin marketplace update`, then reinstall) and restart so `/dive`
  re-registers. The command isn't invocable until installed.
- To validate before installing: dry-run the workflow by hand (manually walk the SKILL.md steps against a
  named site) to check the journal/report output.
- Remote: `github.com/grahamton/ecommerce-observations`. Users install via
  `/plugin marketplace add grahamton/ecommerce-observations`.
