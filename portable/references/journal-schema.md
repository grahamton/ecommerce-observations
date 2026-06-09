# Journal schema

The journal accumulates **one record per stop**, appended in order — never rewrite earlier stops. Records are
the raw material the final report synthesizes.

**Where the journal lives:** if your environment can write files, keep it as one markdown file per dive
(`dive-<domain>-<date>.md`). If it can't (e.g. a single-prompt browser skill), keep the journal as a running
section **in the conversation** and reprint the full thing when the user ends the dive.

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
  - <evidence-backed note>
- **Friction:**
  - [<SEVERITY>] <note> `[ROOT-CAUSE TAG]`
- **Patterns:**
  - ✅ *<good pattern name>* — <one line>
  - ⚠️ *<anti-pattern name>* — <one line> `[ROOT-CAUSE TAG]`
- **Findability:** <only on Findability stops: query, autocomplete?, relevance, zero-result handling>
- **Evidence:** <what you actually saw — a screenshot you were shown, key copy strings, analytics/network
  events if visible (e.g. add_to_cart fired)>
```

## Field notes

- **`<n>`** — monotonic stop counter.
- **`<Stage>`** — one of the seven from `funnel-stages.md` (Entry/Landing, Findability, Category, PDP, Cart,
  Checkout, Confirmation).
- **Severity** for friction: `HIGH` (likely abandons here), `MED` (slows/annoys), `LOW` (papercut).
- **Root-cause tag** — always one of `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
  `[DATA QUALITY]`, `[STRATEGY]`.
- **Patterns** — name them from `pattern-library.md`. ✅ for good, ⚠️ for anti-patterns. Omit the line if
  none seen.
- **Findability line** — include only on Findability stops; omit otherwise.
- **Evidence** — point to what you actually observed. Never record the user's personal or payment data — on
  checkout stops, describe form *structure*, not entered values.
- **Manual flags** — a user "flag this" / "note: …" between navigations is appended to the **current** stop
  under Observations or Patterns, prefixed `(manual)`.

## Empty / skipped fields

Omit a bullet entirely rather than writing "none" — except Friction and Patterns, where a single
`- (none observed)` line is fine when a stop is genuinely clean (it signals you looked).
