# ecommerce-observations

A live **ride-along recorder/reporter** for ecommerce shopping experiences.

You shop a real storefront in your own browser. This tool rides along, observing your live tab as you move
through the funnel — landing → search/nav → category → product detail → cart → checkout — and records
structured observations about **structure, flow, findability, and friction**, calling out recognizable
(anti-)patterns by name as it goes. When you're done, it synthesizes a **dive report**.

It complements (does not replace) the `merch-auditor` plugin: that tool scores a *single page* statically;
this one captures the *journey* across many pages.

## How it differs from `/audit`

| | `merch-auditor /audit` | `ecommerce-observations /dive` |
|---|---|---|
| Unit | One page, point-in-time | A journey across the funnel |
| Who drives | The tool scrapes | **You drive**, the tool observes |
| Output | 4-dimension scorecard | Funnel map + friction inventory + pattern callouts |
| Engine | `merch-connector` (Puppeteer/Firecrawl) | Claude-in-Chrome (your live tab) |

To keep findings consistent across the toolset, the dive report reuses merch-auditor's root-cause tags
(`[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`, `[DATA QUALITY]`, `[STRATEGY]`) and its
conversion_architect lens.

## Prerequisites

- **Claude-in-Chrome extension connected** to the Chrome you shop in. The dive reads your live tab through
  the Claude-in-Chrome MCP (`mcp__Claude_in_Chrome__*`). If it isn't connected, install/enable it first —
  there is no fallback engine for ride-along.

## Install (local testing)

This is a local Claude Code plugin. Install it the same way as `merch-auditor` — upload/register the plugin
directory (`C:\dev\ecommerce-observations`) via your local plugin/marketplace mechanism, then restart so
the `/dive` command registers.

## Usage

```
/dive                       # ride along on whatever tab you're already shopping in
/dive https://example.com   # start by opening this URL, then ride along
/dive --auto                # hands-free; run under /loop, e.g. /loop 45s /dive --auto
/dive --batch               # capture all open storefront tabs in one pass
```

Then just shop. After each navigation, hand control back (type "next", "ok", or any note) and the dive
records the new stop with a short live callout. You can also:

- **Quick-marks (fast notes):** drop a tagged note in 1–3 chars — `! buried add-to-cart` (friction),
  `+ guest checkout` (good), `? is shipping shown yet` (open question), `* nice imagery` (note). The dive
  infers severity + root-cause tag for `!` marks. Freeform `flag this` / `note: …` still works too.
- **Pause/resume:** stop any time; re-run `/dive` on the same domain and it offers to resume where you left
  off (state persists across restarts).
- **Hands-free mode (opt-in):** `/loop 45s /dive --auto` re-polls your tab on an interval instead of waiting
  for you to type. Heavier on context; turn-triggered is the default.
- **End and report:** `end dive` (or `report`) → writes the synthesized report.

## Experimental: `/sweep` (lean variant)

`skills/sweep/SKILL.md` is a stripped counterpart to `/dive` for passive shopping: it captures minimal
evidence per stop (screenshot, URL, commerce network events) in a single append-only `journey.jsonl`,
says nothing while you shop (one-line acknowledgments only), and spends all analysis in one synthesis
pass when you say `report`. No quick-marks, no live callouts, no modes — anything you type mid-shop is
pinned verbatim to the current stop and quoted back in the report. It reuses `/dive`'s funnel-stage and
pattern references at report time. Deliberately not mirrored to `portable/` until it earns it; run both
against the same site to compare.

## Outputs

Written to `outputs/` (gitignored):

- `dive-<domain>-<YYYY-MM-DD-HHMM>.md` — the live journey log, one record per funnel stop.
- `dive-<domain>-<YYYY-MM-DD-HHMM>/stop-<n>.jpg` — per-stop screenshots that feed the report storyboard.
- `dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` — the synthesized report (shareable recap + funnel storyboard
  + ranked friction + open questions) at the end.
- `.dive-state-<domain>.json` — session-state sidecar enabling pause/resume.

## Layout

```
.claude-plugin/plugin.json        # plugin manifest (skills auto-discovered — no commands array)
skills/dive/SKILL.md              # the /dive skill: workflow + methodology + report format
skills/dive/references/
  funnel-stages.md                # stage definitions, classification, per-stage friction
  pattern-library.md              # named good patterns + anti-patterns
  journal-schema.md               # the per-stop observation record schema
portable/                         # platform-neutral version for Gemini in Chrome / Copilot Cowork
outputs/                          # generated journals + reports
```

> Format note: `/dive` is a **skill** (`skills/dive/SKILL.md`), the recommended Claude Code format. It was
> migrated from the legacy `commands/dive.md` in v0.3 — same `/dive` invocation, now with the supporting-files
> directory and skill frontmatter (`disable-model-invocation`, `allowed-tools`).

## New in v0.3

- **Skill-format migration** — `/dive` moved from the legacy `commands/dive.md` to `skills/dive/SKILL.md`
  (commands merged into skills in Claude Code v2.1.3+). Same `/dive` invocation; cleaner structure, richer
  frontmatter, auto-discovered (no `commands` array).

## New in v0.2

- **Quick-marks** — 1–3 char tagged notes (`!` / `+` / `?` / `*`) with inferred severity + root-cause tag.
- **Easier driving** — opt-in hands-free `--auto` (under `/loop`), pause/resume via a state sidecar, and
  `--batch` multi-tab capture.
- **Visual funnel storyboard + shareable recap** — per-stop screenshots assembled into the report (optional
  GIF), plus a Slack/email-ready recap block and an Open Questions roll-up.
- All of the above are mirrored in `portable/` for Gemini in Chrome and Copilot Cowork.

## Deferred (candidates for a later release)

- Analytical depth: compare/persona dives, funnel-health score + trend memory.
- Deep-data enrichment via `merch-connector acquire()` (hybrid deep-scrape).
