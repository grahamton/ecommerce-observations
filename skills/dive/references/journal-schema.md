# Journal schema

The live journal is one markdown file per dive. It opens with a header, then **one record appended per
stop** — never rewrite earlier stops. Records are the raw material the final report synthesizes. Alongside it,
a small **session-state sidecar** tracks progress so a dive can be paused and resumed.

## File header (written once at dive start)

```markdown
# Shopping Dive Journal — <domain>

**Started:** <YYYY-MM-DD HH:MM> · **Tab:** <tabId> · **Mode:** ride-along (live) | agent-driven (--drive)
**Resumed:** <YYYY-MM-DD HH:MM>   ← add this line each time the dive is resumed

---
```

## Per-stop record (appended each stop)

Append exactly one block like this for each stop:

```markdown
## Stop <n> · <Stage> · <HH:MM>

- **URL:** <url>
- **Page:** <page title / h1>
- **What I did:** <the action that led here — "searched 'trail runners'", "clicked into PDP", "added to cart">
- **Observations:**
  - <evidence-backed note>
  - (manual) <a `*` quick-mark or freeform note from the user>
- **Friction:**
  - [<SEVERITY>] <note> `[ROOT-CAUSE TAG]`
  - (manual) [<SEVERITY>] <a `!` quick-mark> `[ROOT-CAUSE TAG]`
- **Patterns:**
  - ✅ *<good pattern name>* — <one line>
  - (manual) ✅ <a `+` quick-mark>
  - ⚠️ *<anti-pattern name>* — <one line> `[ROOT-CAUSE TAG]`
- **Questions:**
  - (manual) <a `?` quick-mark — an open question to resolve later>
- **Findability:** <only on Findability stops: query, autocomplete?, relevance, zero-result handling>
- **Evidence:** screenshot `outputs/dive-<domain>-<ts>/stop-<n>.jpg` (or none) · network: <key events, e.g. add_to_cart fired, shipping calc XHR> · viewport: `<WxH>` (only note if it changed from the previous stop without an intentional resize — see the viewport-stability guard in `funnel-stages.md`)
```

## Post-hoc addendums (corrections after the stop was already appended)

If evidence surfaces *after* a stop's record was written — most commonly, a finding turns out unreproducible
on manual retest, or new context changes how a finding should be read — **do not edit the original bullet
text**. Append a dated addendum immediately after the affected stop's record instead:

```markdown
- **Reproducibility caveat (added post-dive, <YYYY-MM-DD>):** <what changed, what evidence prompted the
  correction, and the revised severity/confidence>. Original finding left intact above for the record.
```

This keeps the journal an honest record of what was observed and when, rather than quietly rewriting history —
and it's the mechanism the report's "unconfirmed" framing (see `SKILL.md` Step 4) reads from.

## Quick-mark mapping

User notes use a fast grammar; map each to a field and prefix the entry `(manual)`:

| Mark | → field | Notes |
|---|---|---|
| `! <text>` | Friction | infer `SEVERITY` + root-cause tag if not stated |
| `+ <text>` | Patterns (✅) | name from `pattern-library.md` if it fits, else plain |
| `? <text>` | Questions | collected into the report's **Open Questions** section |
| `* <text>` | Observations | neutral note |
| freeform / "flag this" / "note:" | Observations or the best-fit field | the marks are a fast path, not a requirement |

## Session-state sidecar (pause/resume)

Write `outputs/.dive-state-<domain>.json` at dive start and **after every stop**. It lets `/dive` detect and
resume an in-progress dive (even across restarts).

```jsonc
{
  "domain": "example.com",
  "status": "in_progress",            // "in_progress" | "ended"
  "tabId": 123,
  "lastSeenUrl": "https://example.com/cart",
  "stopCounter": 4,
  "mode": "turn",                     // "turn" | "auto" | "batch" | "drive"
  "checkinEvery": null,               // null = off, or the base N (see SKILL.md --checkin)
  "cleanCheckinStreak": 0,            // consecutive clean (nothing-above-LOW) check-ins; drives adaptive cadence
  "siteContext": null,                // combined text from --context + outputs/site-notes/<domain>.md, or null
  "lastViewport": "1428x1111",        // from the most recent read_page call — for the viewport-stability guard
  "journalPath": "outputs/dive-example.com-2026-06-08-1530.md",
  "screenshotDir": "outputs/dive-example.com-2026-06-08-1530/",
  "screenshotsPersist": true,         // set false at dive start if save_to_disk no-ops in this environment
  "microPrompt": true,                // false once the user says "quiet"
  "startedAt": "2026-06-08 15:30",
  "updatedAt": "2026-06-08 15:42"
}
```

On `/dive` start: if a sidecar with `status: in_progress` exists for the domain, offer **resume** (load
`stopCounter`/`journalPath`, append) or **start fresh** (rename the old sidecar to `*.ended.json`, begin a
new journal). Set `status: ended` when the user ends the dive.

## Persistent site notes (`outputs/site-notes/<domain>.md`)

Separate from the per-dive journal, this file accumulates durable facts about a domain across dives — see
"Site context" in `SKILL.md` Step 3 for when it's loaded and updated. Append-only, one dated section per dive
that added something worth keeping:

```markdown
# Site notes — example.com

## 2026-07-02 dive
- Runs two typeahead backends (a new search vendor + a legacy one) — check both suggest-endpoint patterns.
- Tracking vendor is Adobe AppMeasurement, not GTM — don't flag a missing GTM container as a tracking gap.
- Guest checkout confirmed working; no ratings/reviews yet.
```

Keep entries to facts likely to still hold next dive (vendor/backend identity, structural capabilities) —
not per-dive friction that's likely to be fixed or to change (a specific broken CTA, a one-off error state),
which belongs in that dive's report instead.

## Field notes

- **`<n>`** — monotonic stop counter from the sidecar `stopCounter`.
- **`<Stage>`** — one of the seven from `funnel-stages.md` (Entry/Landing, Findability, Category, PDP, Cart,
  Checkout, Confirmation).
- **Severity** for friction: `HIGH` (likely abandons here — blocks or breaks the path, hides price/fees
  until late, forces an account with no warning), `MED` (slows or annoys but the path still works — extra
  steps, unclear labels, weak trust signals), `LOW` (papercut — cosmetic, inconsistent copy). Append
  `, unconfirmed` when the finding depends on an interaction you performed yourself and hasn't been
  independently verified (see the viewport-stability guard in `funnel-stages.md` and the `--checkin` gate in
  `SKILL.md`) — an unconfirmed HIGH should still be *labeled* HIGH-and-unconfirmed, not silently downgraded,
  so the distinction is visible in the record.
- **Root-cause tag** — always one of `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
  `[DATA QUALITY]`, `[STRATEGY]`.
- **Patterns** — name them from `pattern-library.md`. ✅ for good, ⚠️ for anti-patterns.
- **Questions** — only present when the user left a `?` mark; these roll up into the report.
- **Findability line** — include only on Findability stops; omit otherwise.
- **Evidence** — point to what you actually observed: the per-stop screenshot path (or the text-storyboard
  note if screenshots aren't persisting in this environment), and meaningful network events — including
  whether the stage's expected ecommerce event fired (GTM's `view_item`/`add_to_cart`/`begin_checkout`/
  `purchase`, or the site's actual analytics vendor's equivalent — see the tracking check in
  `funnel-stages.md`), not just that the UI updated. Note the viewport reading when it changed unexpectedly.
  Never record the user's personal or payment data — on checkout stops, describe form *structure*, not
  entered values, and skip the screenshot if it would expose them.

## Empty / skipped fields

Omit a bullet entirely rather than writing "none" — except Friction and Patterns, where a single
`- (none observed)` line is fine when a stop is genuinely clean (it signals you looked). Omit Questions
unless a `?` mark was left.
