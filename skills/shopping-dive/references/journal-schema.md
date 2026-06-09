# Journal schema

The live journal is one markdown file per dive. It opens with a header, then **one record appended per
stop** — never rewrite earlier stops. Records are the raw material the final report synthesizes.

## File header (written once at dive start)

```markdown
# Shopping Dive Journal — <domain>

**Started:** <YYYY-MM-DD HH:MM> · **Tab:** <tabId> · **Mode:** ride-along (live)

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
  - <evidence-backed note>
- **Friction:**
  - [<SEVERITY>] <note> `[ROOT-CAUSE TAG]`
- **Patterns:**
  - ✅ *<good pattern name>* — <one line>
  - ⚠️ *<anti-pattern name>* — <one line> `[ROOT-CAUSE TAG]`
- **Findability:** <only on Findability stops: query, autocomplete?, relevance, zero-result handling>
- **Evidence:** screenshot <ref/none> · network: <key events seen, e.g. add_to_cart fired, shipping calc XHR>
```

## Field notes

- **`<n>`** — monotonic stop counter from session state.
- **`<Stage>`** — one of the seven from `funnel-stages.md` (Entry/Landing, Findability, Category, PDP, Cart,
  Checkout, Confirmation).
- **Severity** for friction: `HIGH` (likely abandons here), `MED` (slows/annoys), `LOW` (papercut).
- **Root-cause tag** — always one of `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`,
  `[DATA QUALITY]`, `[STRATEGY]` (matches merch-auditor).
- **Patterns** — name them from `pattern-library.md`. ✅ for good, ⚠️ for anti-patterns. Omit the line if
  none seen.
- **Findability line** — include only on Findability stops; omit otherwise.
- **Evidence** — point to what you actually observed. If you took a screenshot, reference it; for network,
  list the meaningful events (analytics fired? add-to-cart event? fee/shipping calc call?). Never record
  the user's personal or payment data — on checkout stops, describe form *structure*, not entered values.
- **Manual flags** — a user "flag this" / "note: …" between navigations is appended to the **current** stop
  under Observations or Patterns, prefixed `(manual)`.

## Empty / skipped fields

Omit a bullet entirely rather than writing "none" — except Friction and Patterns, where a single
`- (none observed)` line is fine when a stop is genuinely clean (it signals you looked).
