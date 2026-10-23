# ecommerce-observations

A live **ride-along recorder/reporter** for ecommerce shopping journeys, packaged as a Claude Code plugin.

You shop a real storefront in your own Chrome. `/dive` rides along, observing your live tab as you move through
the funnel — landing → search/nav → category → product detail → cart → checkout — and records structured
observations about **flow, findability, friction, and tracking**, calling out recognizable (anti-)patterns by
name as it goes. When you're done, it writes a **dive report** with a shareable recap, a funnel storyboard, a
ranked friction inventory, and open questions.

**Who it's for:** CRO consultants, ecommerce/merchandising teams, UX researchers, and analytics/GTM implementers
who already walk storefronts by hand — for audits, competitor teardowns, or QA after a release — and want the
walkthrough captured as structured evidence instead of scattered notes.

## How it differs from other tools

| | Page scorers / AI audit tools | Session replay (Hotjar, Clarity, …) | `/dive` |
|---|---|---|---|
| Unit | One URL, point-in-time | Many real customers, aggregate | One expert journey across the funnel |
| Who drives | The tool | Your customers | **You** (or the agent, in `--drive`) |
| Judgment | Generic rubric | None — you interpret | Yours, captured live via quick-marks |
| Output | Score | Recordings + heatmaps | Funnel map, friction inventory, named patterns, storyboard |

Findings use a fixed set of root-cause tags — `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
`[DATA QUALITY]`, `[STRATEGY]` — so reports from different dives read consistently.

**See it first:** [`examples/`](examples/) has full dive reports (journal + storyboard) from a B2C and a B2B
storefront.

## Prerequisites

- **Claude Code** with plugin support.
- **[Claude in Chrome](https://claude.com/chrome)** installed and connected to the Chrome you shop in (requires
  a paid Claude plan). `/dive` reads your live tab through its tools; there is no fallback engine.
- Not on Claude Code? See [`portable/`](portable/) for a version that runs in Gemini in Chrome, Copilot Cowork,
  or any browser agent that can see the page.

## Install

```
/plugin marketplace add grahamton/ecommerce-observations
/plugin install ecommerce-observations@grahamton
```

Restart Claude Code if `/dive` doesn't appear right away.

## Usage

```
/dive                       # ride along on whatever tab you're already shopping in
/dive https://example.com   # start by opening this URL, then ride along
/dive --auto                # hands-free; run under /loop, e.g. /loop 45s /dive --auto
/dive --batch               # capture all open storefront tabs in one pass
/dive https://example.com --drive --checkin   # the agent shops; pauses to check in with you
/dive --context "tracking is Adobe, not GTM"  # brief the dive with what you already know
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
- **Agent-driven (`--drive`):** the agent shops the site itself, stopping before any account creation or
  payment. Pair with `--checkin[=N]` to pause every N stops — and always before logging a HIGH finding the agent
  caused by its own interaction, so testing artifacts don't ship as site bugs.
- **Site context:** `--context "..."` briefs the dive upfront; durable facts are also saved per domain to
  `outputs/site-notes/<domain>.md` and loaded automatically next time.
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
- `site-notes/<domain>.md` — durable per-domain facts carried between dives.

**Privacy:** on checkout, the dive records structure and friction only — never your personal data, address,
payment details, or the contents of checkout network requests.

## Layout

```
.claude-plugin/plugin.json        # plugin manifest (skills auto-discovered)
.claude-plugin/marketplace.json   # lets the repo install via /plugin marketplace add
skills/dive/SKILL.md              # the /dive skill: workflow + methodology + report format
skills/dive/references/
  funnel-stages.md                # stage definitions, classification, per-stage friction
  pattern-library.md              # named good patterns + anti-patterns
  journal-schema.md               # per-stop record, sidecar, and site-notes formats
skills/sweep/SKILL.md             # experimental lean variant
portable/                         # platform-neutral version for Gemini in Chrome / Copilot Cowork
test-scenarios/dive-scenario.md   # manual acceptance walkthrough
examples/                         # real dive output from public storefronts
assets/icon.png                   # plugin icon
outputs/                          # generated journals, reports, site notes (gitignored)
```

See [CHANGELOG.md](CHANGELOG.md) for release history.

## Deferred (candidates for a later release)

- Analytical depth: compare/persona dives, funnel-health score + trend memory.
- Deep-data enrichment via a separate page-scraping tool (hybrid deep-scrape).

## License

MIT — see [LICENSE](LICENSE).
