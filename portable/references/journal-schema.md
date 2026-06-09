# Journal schema

The journal accumulates **one record per stop**, appended in order — never rewrite earlier stops. Records are
the raw material the final report synthesizes.

**Where the journal lives:** if your environment can write files, keep it as one markdown file per dive
(`dive-<domain>-<date>.md`) and persist a session-state sidecar (below) for cross-session resume. If it can't
(e.g. a single-prompt browser skill), keep the journal as a running section **in the conversation** and
reprint the full thing when the user ends the dive (the conversation itself is your implicit resume).

## Journal header (once at dive start)

```markdown
# Shopping Dive Journal — <domain>

**Started:** <date/time> · **Mode:** ride-along (live observation)

---
```

## Per-stop record (appended each stop)

Append exactly one block like this for each stop:

```markdown
## Stop <n> · <Stage> · <time>

- **URL:** <url>
- **Page:** <page title / h1>
- **What I did:** <the action that led here — "searched 'trail runners'", "clicked into PDP", "added to cart">
- **Observations:**
  - <evidence-backed note>
  - (manual) <a `*` quick-mark or freeform note>
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
- **Evidence:** <what you actually saw — a screenshot you were shown, key copy strings, analytics/network
  events if visible (e.g. add_to_cart fired)>
```

## Quick-mark mapping

User notes use a fast grammar; map each to a field and prefix `(manual)`:

| Mark | → field | Notes |
|---|---|---|
| `! <text>` | Friction | infer `SEVERITY` + root-cause tag if not stated |
| `+ <text>` | Patterns (✅) | name from `pattern-library.md` if it fits, else plain |
| `? <text>` | Questions | rolls up into the report's **Open Questions** |
| `* <text>` | Observations | neutral note |
| freeform / "flag this" / "note:" | best-fit field | the marks are a fast path, not a requirement |

## Session-state sidecar (file-capable environments only)

If you can write files, write `.dive-state-<domain>.json` at start and **after every stop** so a dive can be
paused/resumed across sessions:

```json
{
  "domain": "example.com",
  "status": "in_progress",        // "in_progress" | "ended"
  "lastSeenUrl": "https://example.com/cart",
  "stopCounter": 4,
  "journalPath": "dive-example.com-2026-06-08-1530.md",
  "microPrompt": true,
  "updatedAt": "2026-06-08 15:42"
}
```

On start, if a sidecar with `status: in_progress` exists, offer **resume** or **start fresh**. In a
single-prompt skill (no file writes), skip this — the conversation is your within-session resume.

## Field notes

- **`<n>`** — monotonic stop counter.
- **`<Stage>`** — one of the seven from `funnel-stages.md` (Entry/Landing, Findability, Category, PDP, Cart,
  Checkout, Confirmation).
- **Severity** for friction: `HIGH` (likely abandons here), `MED` (slows/annoys), `LOW` (papercut).
- **Root-cause tag** — always one of `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
  `[DATA QUALITY]`, `[STRATEGY]`.
- **Patterns** — name them from `pattern-library.md`. ✅ for good, ⚠️ for anti-patterns.
- **Questions** — only present when a `?` mark was left; these roll up into the report.
- **Findability line** — include only on Findability stops; omit otherwise.
- **Evidence** — point to what you actually observed. Never record the user's personal or payment data — on
  checkout stops, describe form *structure*, not entered values.

## Empty / skipped fields

Omit a bullet entirely rather than writing "none" — except Friction and Patterns, where a single
`- (none observed)` line is fine when a stop is genuinely clean. Omit Questions unless a `?` mark was left.
