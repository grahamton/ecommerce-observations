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
```

Then just shop. After each navigation, hand control back (type "next", "ok", or any note) and the dive
records the new stop with a short live callout. You can also:

- **Flag a moment:** `flag this` or `note: the size filter is buried`
- **Hands-free mode (opt-in):** run it under `/loop`, e.g. `/loop 45s /dive` — re-polls your tab on an
  interval instead of waiting for you to type. Heavier on context; turn-triggered is the default.
- **End and report:** `end dive` (or `report`) → writes the synthesized report.

## Outputs

Written to `outputs/` (gitignored):

- `dive-<domain>-<YYYY-MM-DD-HHMM>.md` — the live journey log, one record per funnel stop.
- `dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` — the synthesized report at the end.

## Layout

```
.claude-plugin/plugin.json        # plugin manifest + /dive registration
commands/dive.md                  # the /dive command workflow
skills/shopping-dive/SKILL.md     # dive methodology (loaded by the command)
skills/shopping-dive/references/
  funnel-stages.md                # stage definitions, classification, per-stage friction
  pattern-library.md              # named good patterns + anti-patterns
  journal-schema.md               # the per-stop observation record schema
outputs/                          # generated journals + reports
```

## Not in v0.1 (deferred)

- Optional `merch-connector acquire()` enrichment on a paused page (hybrid deep-scrape).
- Polished hands-free polling.
- Multi-dive history / comparison.
