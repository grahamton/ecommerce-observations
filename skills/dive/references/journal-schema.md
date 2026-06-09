# Journal schema

The live journal is one markdown file per dive. It opens with a header, then **one record appended per
stop** — never rewrite earlier stops. Records are the raw material the final report synthesizes. Alongside it,
a small **session-state sidecar** tracks progress so a dive can be paused and resumed.

## File header (written once at dive start)

```markdown
# Shopping Dive Journal — <domain>

**Started:** <YYYY-MM-DD HH:MM> · **Tab:** <tabId> · **Mode:** ride-along (live)
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
- **Evidence:** screenshot `outputs/dive-<domain>-<ts>/stop-<n>.jpg` (or none) · network: <key events, e.g. add_to_cart fired, shipping calc XHR>
```

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

```json
{
  "domain": "example.com",
  "status": "in_progress",            // "in_progress" | "ended"
  "tabId": 123,
  "lastSeenUrl": "https://example.com/cart",
  "stopCounter": 4,
  "mode": "turn",                     // "turn" | "auto" | "batch"
  "journalPath": "outputs/dive-example.com-2026-06-08-1530.md",
  "screenshotDir": "outputs/dive-example.com-2026-06-08-1530/",
  "microPrompt": true,                // false once the user says "quiet"
  "startedAt": "2026-06-08 15:30",
  "updatedAt": "2026-06-08 15:42"
}
```

On `/dive` start: if a sidecar with `status: in_progress` exists for the domain, offer **resume** (load
`stopCounter`/`journalPath`, append) or **start fresh** (rename the old sidecar to `*.ended.json`, begin a
new journal). Set `status: ended` when the user ends the dive.

## Field notes

- **`<n>`** — monotonic stop counter from the sidecar `stopCounter`.
- **`<Stage>`** — one of the seven from `funnel-stages.md` (Entry/Landing, Findability, Category, PDP, Cart,
  Checkout, Confirmation).
- **Severity** for friction: `HIGH` (likely abandons here), `MED` (slows/annoys), `LOW` (papercut).
- **Root-cause tag** — always one of `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
  `[DATA QUALITY]`, `[STRATEGY]` (matches merch-auditor).
- **Patterns** — name them from `pattern-library.md`. ✅ for good, ⚠️ for anti-patterns.
- **Questions** — only present when the user left a `?` mark; these roll up into the report.
- **Findability line** — include only on Findability stops; omit otherwise.
- **Evidence** — point to what you actually observed: the per-stop screenshot path, and meaningful network
  events. Never record the user's personal or payment data — on checkout stops, describe form *structure*,
  not entered values, and skip the screenshot if it would expose them.

## Empty / skipped fields

Omit a bullet entirely rather than writing "none" — except Friction and Patterns, where a single
`- (none observed)` line is fine when a stop is genuinely clean (it signals you looked). Omit Questions
unless a `?` mark was left.
