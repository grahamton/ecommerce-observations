---
name: dive
description: Ride along as the user shops a storefront — observe their live Chrome tab across the funnel (landing → search/nav → category → PDP → cart → checkout), capture one structured observation per stop with fast quick-mark notes, call out named patterns and friction, and synthesize a dive report with a visual funnel storyboard. Use when the user runs a shopping ride-along / dive, records an ecommerce shopping session, or analyzes a checkout funnel as it is walked.
argument-hint: [storefront-url | "current tab"] [--auto] [--batch] [--drive] [--checkin[=N]] [--context "..."]
disable-model-invocation: true
allowed-tools: mcp__Claude_in_Chrome__list_connected_browsers, mcp__Claude_in_Chrome__tabs_context_mcp, mcp__Claude_in_Chrome__read_page, mcp__Claude_in_Chrome__get_page_text, mcp__Claude_in_Chrome__read_network_requests, mcp__Claude_in_Chrome__navigate, mcp__Claude_in_Chrome__computer, mcp__Claude_in_Chrome__gif_creator, Write, Read, Edit, Bash
---

# Shopping Dive

A **dive** is a live walkthrough of a real shopping session — either the user driving their own browser while
you observe, or you driving it yourself in `--drive` mode. Either way, you record what happens at each stop on
the funnel and synthesize a report at the end. In ride-along mode you are a quiet, sharp-eyed companion — **an
observer, not a narrator**. In `--drive` mode you are the shopper — **but a supervised one**; see `--checkin`.

`$ARGUMENTS` may contain an optional starting URL plus flags:
- `--auto` — hands-free polling of a human-driven tab, intended to run under `/loop`.
- `--batch` — capture all open storefront tabs in one pass.
- `--drive` — **agent-driven**: no human is shopping, so you navigate/click/type through the funnel yourself,
  picking a reasonable path (or following a path the user described in `$ARGUMENTS`). This is a deliberate
  exception to the ride-along stance below — see "Agent-driven navigation" under Step 3.
- `--checkin[=N]` — pairs with `--drive` (or `--auto`). Pause and post a status check-in every `N` stops
  (default 1 = every stop) instead of running silently to the end. See "Check-in gate" under Step 3. This is
  the recommended default when driving autonomously — running `--drive` without `--checkin` means nothing
  catches a bad finding before it's in the final report.
- `--context "<text>"` — upfront site knowledge to load *before* the capture loop starts (e.g. "this site runs
  two typeahead backends: Fusion + legacy", "tracking vendor is Adobe AppMeasurement, not GTM"). Combined with
  any persistent site notes for the domain (see "Site context" under Step 3) into one `siteContext` string
  carried in the sidecar. The dive should actively check claims in it during evidence-gathering, not just log
  it as a passive note.

If `$ARGUMENTS` is empty, ride along on the active tab in turn-triggered mode (a human is shopping, driven by
you observing only).

Three reference files carry the detail — read them as needed:
- `references/funnel-stages.md` — the 7 stages, how to classify them, what to capture, per-stage friction, and
  the viewport-stability guard.
- `references/pattern-library.md` — named good patterns and anti-patterns to call out by name.
- `references/journal-schema.md` — the per-stop record format, the quick-mark mapping, and the session-state
  sidecar (for pause/resume).

## Core stance

- **Observe, don't drive — unless `--drive` says otherwise.** In ride-along/turn/auto/batch modes, the user
  navigates and you only read the tab. Never `navigate` mid-dive except the one optional seed navigation at
  the very start. **In `--drive` mode this is intentionally overridden**: there is no human to observe, so you
  perform the shopping actions yourself. Don't silently blend the two — check the sidecar's `mode` field
  before deciding whether an action you're about to take is in-bounds.
- **One record per stop, append-only.** A "stop" is a meaningful page state on the funnel. Each gets exactly
  one journal record, appended — never rewrite earlier stops. Corrections to an already-appended stop (e.g. a
  reproducibility caveat learned after the fact) are appended as a dated addendum to that stop, not an edit of
  the original text — keep the original observation intact so the record of what you saw, and when, stays
  honest.
- **Evidence over opinion.** Every observation cites something you actually saw — a URL, a copy string, an
  accessibility-tree element, a network event, a screenshot. No unsupported assertions.
- **Distrust your own automation before you distrust the site.** If a finding depends on an *interaction* you
  performed (click/type) rather than a passive read, and nothing else on the page corroborates it, treat it as
  provisional. See the viewport-stability guard in `references/funnel-stages.md` — a resized window, a stolen
  focus event, or a race condition in your own tool calls can look exactly like a site bug. Retrying the same
  action twice in the same unstable session is not independent confirmation.
- **Terse in chat, complete in the journal.** Live callouts are 1–3 lines. The full record goes to the file.
- **Borrow the auditor's vocabulary.** Tag every friction/finding with a root-cause tag so dive output
  matches `/audit`: `[UX DESIGN]`, `[CONVERSION]`, `[FRONTEND INTEGRATION]`, `[DATA QUALITY]`, `[STRATEGY]`.
  Frame analysis through the **conversion_architect** lens: at every stop ask (1) does the shopper know what
  to do next, (2) is there a reason to trust it (social proof, price/fee/shipping clarity, returns), and
  (3) what here adds effort or doubt between intent and purchase. Severity follows likely impact on that
  path — see the severity definitions in `references/journal-schema.md`.

## The funnel model (7 stages)

Full definitions and classification heuristics live in `references/funnel-stages.md`. In brief: **Entry /
Landing → Findability (search + nav) → Category / Listing → PDP → Cart → Checkout → Confirmation.** A real
dive rarely hits all 7 and rarely in order — capture what actually happens, in the order it happens.

---

## Step 1 — Attach + check for a dive to resume

1. Call `list_connected_browsers` to confirm Chrome is connected. If none is connected, STOP and tell the user
   to connect the Claude-in-Chrome extension — there is no fallback engine.
2. Call `tabs_context_mcp` to enumerate tabs.
   - **Ride-along modes (default/`--auto`/`--batch`):** reuse the user's existing shopping tab(s) — do NOT
     create a fresh MCP tab.
   - **`--drive` mode:** it's fine to create a fresh tab if none is open, since you're the one shopping.
   Record the `tabId`(s).
3. Determine the domain (from the seed URL or the active tab) — it names the output files.
4. **Load site context.** Parse `--context "<text>"` from `$ARGUMENTS` if present. Then check for
   `outputs/site-notes/<domain>.md` (persistent notes auto-accumulated from prior dives on this domain —
   see "Site context" under Step 3). If it exists, read it. Combine whichever of the two exist (site notes
   first, then the `--context` value) into one `siteContext` string, stored in the sidecar for the rest of the
   dive. If either source is present, say so in one terse line ("Loaded site notes for example.com (3 prior
   facts)" / "Context noted: two typeahead backends") — don't dump the full text into chat.
5. **Resume check.** Look for `outputs/.dive-state-<domain>.json` with `status: in_progress` (schema in
   `references/journal-schema.md`). If found, ask the user: **resume** (load `stopCounter`/`journalPath`,
   append, add a `**Resumed:**` line to the journal header) or **start fresh** (rename the old sidecar to
   `*.ended.json`, begin a new journal). If not found, start fresh.
6. If `$ARGUMENTS` contains a URL and we're starting fresh, `navigate(tabId, url=…)` once to seed; otherwise
   observe the current tab as-is (or, in `--drive` mode with no URL given, ask the user what to shop for
   before doing anything else).

## Step 2 — Open (or reattach to) the journal + state

On a fresh dive, create `outputs/dive-<domain>-<YYYY-MM-DD-HHMM>.md` with a header, make the screenshot dir
`outputs/dive-<domain>-<YYYY-MM-DD-HHMM>/`, and write the session-state sidecar
`outputs/.dive-state-<domain>.json`. Get the timestamp via Bash: `date +%Y-%m-%d-%H%M` (or
`Get-Date -Format "yyyy-MM-dd-HHmm"` in PowerShell), whichever matches the current shell.

Carry session state from the sidecar across turns (`tabId`, `lastSeenUrl`, `stopCounter`, `mode`,
`journalPath`, `screenshotDir`, `microPrompt`, `checkinEvery`, `siteContext`). **Update the sidecar after
every stop.**

**Environment note — don't hardcode an output path.** Detect where you're actually running rather than
assuming one environment: if there's a Cowork/session outputs directory available, use it; if you're running
locally on the user's machine with a project-relative `outputs/` folder (e.g. a local checkout of this
plugin), use that instead. Check what's writable/available at dive start and use it consistently for the rest
of the session — don't assume a specific hardcoded path exists.

Set `mode` in the sidecar from the flags: `--auto` → `auto`, `--batch` → `batch`, `--drive` → `drive`, else
`turn`. If `--checkin` is present, also set `checkinEvery` (default `1`) regardless of the base mode.

## Step 3 — The capture loop

Same loop in all modes; only the trigger (and, in `--drive`, who's clicking) differs:

- **turn** (default): each time control returns to you (user types "next", a quick-mark, anything).
- **auto** (`--auto`, run under `/loop` e.g. `/loop 45s /dive --auto`): each loop tick, observing a
  human-driven tab. Tell the user once that auto-polling burns context, so keep intervals ≥30s.
- **batch** (`--batch`): iterate every open storefront tab from `tabs_context_mcp` in funnel order, recording
  one stop per tab in a single pass.
- **drive** (`--drive`): you choose and perform the next action yourself (click a nav link, type a search
  query, click into a PDP, add to cart) based on the funnel model and any path the user described. Immediately
  after acting, run the same detect-and-capture steps below as if you were observing a human's navigation.

On each check:

1. **Detect navigation.** Read the current tab's URL/title (cheap: `tabs_context_mcp`, or `read_page` with a
   small depth). Compare to `lastSeenUrl`. **Also record the `Viewport` dimensions from `read_page`'s output**
   and compare to the previous stop's recorded viewport.
2. **If the page changed** → it's a new stop:
   - Classify the funnel stage using `references/funnel-stages.md`.
   - Gather evidence appropriate to the stage: `read_page` (nav/facets/interactive elements), `get_page_text`
     (copy, zero-result/error states), `read_network_requests` (analytics + add-to-cart/checkout XHRs — check
     for a GTM container and whether the stage's ecommerce dataLayer event, `view_item` / `add_to_cart` /
     `begin_checkout` / `purchase`, actually fired; **if no GTM container is found at all, don't conclude
     tracking is broken — check whether an equivalent event fired through the site's actual analytics stack
     (Adobe/Segment/custom) before calling it a gap**). Don't over-collect — pull only what the stage calls
     for. Works the same on B2B/B2B2C storefronts — read `references/funnel-stages.md`'s B2B notes on
     PDP/Cart/Checkout before flagging a quote-cart or price-on-request as friction. On any stop with a form,
     also check for real WebMCP declarative attributes (`toolname`/`tooldescription` — see the WebMCP
     readiness check in `references/funnel-stages.md`); call out an *agent-ready form* when found, but don't
     score its absence as friction. **If `siteContext` is loaded and names something concrete and checkable at
     this stage** (a specific backend, vendor, or feature — e.g. "two typeahead systems"), actively look for
     it in this stage's evidence gathering (e.g. check network requests for both systems' endpoint patterns)
     instead of passively noting after the fact that only one (or neither) showed up.
   - **Viewport-stability guard.** If the viewport dimensions from step 1 differ from the previous stop's
     without an intentional resize, note it in this stop's Evidence and **downgrade confidence on any
     layout-related finding at this stop** (missing/vanished elements, broken CTAs, elements not where
     expected) — these are classic symptoms of a responsive breakpoint reflow triggered by the tooling itself,
     not necessarily the site. Full detail and the motivating case in `references/funnel-stages.md`.
   - **Take a screenshot** (`computer`, screenshot action) and try to save it to `<screenshotDir>/stop-<n>.jpg`
     for the storyboard. **If screenshot persistence to disk isn't available in the current environment (the
     save silently no-ops), don't retry per-stop — note the limitation once at dive start and fall back to a
     text storyboard entry for every stop**, same as the existing checkout-privacy fallback. **Skip the
     screenshot on checkout stops if it would expose entered personal/payment data** — note why instead.
   - Build one record per `references/journal-schema.md` and **append it to the journal** (never rewrite prior
     stops). Increment `stopCounter`; update the sidecar.
   - Emit a **terse live callout** (1–3 lines): stage, the one or two things that matter, any named pattern or
     friction with its tag. Don't dump the whole record into chat.
   - **Micro-prompt** (if `microPrompt` is true, and not already covered by a `--checkin` this stop): append
     one optional line — "anything bug you here? `!`/`+`/skip". If the user ever says "quiet"/"stop asking",
     set `microPrompt: false` and never nag again.
3. **Quick-marks / manual notes** (no page change needed): recognize the grammar and attach to the current
   stop, prefixed `(manual)` —
   - `! <text>` → Friction (infer severity HIGH/MED/LOW + a root-cause tag; show your inference so one char
     can correct it)
   - `+ <text>` → ✅ Pattern (name it from the library if it fits)
   - `? <text>` → Questions (rolls up into the report's Open Questions)
   - `* <text>` → Observation
   - a shared/pasted screenshot + mark → log as evidence on the current stop
   - freeform "flag this" / "note: …" still works.
4. **If the user says `end dive` / `report` / `wrap up`** → set sidecar `status: ended` and go to Step 4.

Keep it light and non-spammy. You are a quiet companion, not a running monologue — except at check-ins, which
are meant to be a real pause, not a rhetorical one.

### Agent-driven navigation (`--drive`)

When there's no human shopping, you pick the path. Default to a sensible funnel walk (Landing → Findability →
Category → PDP → Cart → Checkout, stopping short of real purchase per the privacy rule below) unless the user
described a specific path or product to shop for. Prefer the most direct real UI action available (click the
actual nav link / search box / Add to Cart button) over typing a URL directly — you're trying to observe how
the funnel *actually behaves* under normal interaction, and a raw URL navigation skips exactly the kind of
interaction bugs a dive exists to catch. It's fine to fall back to direct navigation when a UI path is broken
or blocked (log why), but don't default to it for convenience.

**Because you're both the shopper and the observer in this mode, you have no independent witness to your own
actions.** Hold interaction-derived findings — especially anything above LOW severity, and especially anything
tagged `[FRONTEND INTEGRATION]` — to a higher bar before finalizing: recheck the viewport-stability guard, and
prefer flagging "observed once, unconfirmed" over asserting a defect outright when you're the only source of
evidence. A `--checkin` (below) is the cheapest way to get a second opinion before it goes in the report.

### Site context (`--context` + persistent site notes)

Two sources feed the `siteContext` string loaded in Step 1:

- **`--context "<text>"`** — one-off knowledge for this dive, e.g. domain facts the user already knows that
  would otherwise only surface mid-dive via a check-in or quick-mark.
- **`outputs/site-notes/<domain>.md`** — a persistent, auto-accumulated file per domain. If this dive's
  domain has one, its content is loaded the same way, so a repeat dive on a domain already-visited starts
  pre-briefed instead of rediscovering the same baseline facts (backend/vendor identity, guest-checkout
  availability, etc.) from scratch.

Both are *active* inputs to the capture loop (see the `siteContext` bullet in Step 3's evidence-gathering),
not just background color repeated back in the report.

**Updating site notes.** At dive end (Step 4), append — never overwrite — a dated section to
`outputs/site-notes/<domain>.md` with confirmed, durable facts from this session: vendor/backend identity,
structural facts (guest checkout works, no reviews yet), and any `--context` the user supplied. Create the
file (and the `outputs/site-notes/` folder) if it doesn't exist. Site notes live under `outputs/` on
purpose: they're the user's own accumulated knowledge, kept local and gitignored — never write them into
the plugin's install directory, which is overwritten on update. Keep it to facts likely to still be true next dive — per-dive friction that's likely
to change (a specific broken CTA, a one-off error) belongs in the report, not site notes. Format:

```markdown
## <YYYY-MM-DD> dive
- <fact>
- <fact>
```

### Check-in gate (`--checkin[=N]`)

Applies on top of `--drive` (most useful there) or `--auto`. Instead of running to the end unsupervised:

- Every `N` stops (default 1), pause and post a short check-in: stops completed so far, the path taken, and
  any findings above LOW severity — then explicitly ask the user whether to continue automatically, take over
  driving themselves, or double-check something before you proceed.
- **Always check in immediately, regardless of `N`,** when you're about to log a HIGH-severity finding that
  came from an interaction you performed yourself (not a passive page read). This is the single most important
  gate: it's exactly the failure mode where an agent-driven dive can hand back a confident-sounding but wrong
  finding — a testing artifact (resized viewport, stolen focus, a race condition in your own tool calls)
  dressed up as a site bug, with nobody in the loop to catch it before the report goes out. Ask the user to
  either verify it themselves or greenlight logging it as unconfirmed.
- **Adaptive cadence.** Track consecutive *clean* check-ins (nothing above LOW to report) in the sidecar's
  `cleanCheckinStreak`. After each clean check-in, widen the effective interval by 1 stop beyond the base `N`
  (`checkinEvery`), capped at `N + 2` — e.g. `--checkin=1` loosens to every 2 stops after one clean check-in,
  then every 3 after two in a row. Any check-in with a finding above LOW, or the mandatory HIGH-severity pause
  above, resets the effective interval back to the base `N` immediately. This only relaxes the *routine*
  cadence — it never overrides the mandatory HIGH-severity pause, which fires regardless of where you are in
  the interval.
- A check-in is a real pause, not a rhetorical aside — wait for the user's response before taking the next
  action.

### Privacy (non-negotiable)

During checkout, never read, transcribe, or store the user's personal data, address, or payment details.
Observe *structure and friction* (field count, guest option, error handling), not the values typed. Skip any
screenshot that would expose entered personal/payment data. **Network requests count too:** on checkout
stops, use `read_network_requests` only to confirm which events fired (event name, endpoint host, status) —
never copy request URLs, query strings, or payloads into the journal, since they can carry email, address,
or order identifiers. **In `--drive` mode this also means: never create
a real account, never enter real payment/personal info, and stop at whatever step would require either** —
note it as "confirmation not reached," which is expected, not a failure. If the browser tooling itself denies
access to a page (e.g. an identity/login domain), respect that boundary rather than working around it.

### Live callout style

> **Stop 4 · PDP** — Add-to-cart is clear and sticky ✅. But no reviews above the fold and only 1 product
> image `[CONVERSION]`. Pattern: *missing social proof on PDP*.

One to three lines. Lead with the stage. Name the pattern. Tag the friction. Move on.

## Step 4 — Synthesize the dive report

When the dive ends, read back the full journal and produce the report with these sections, in order. Before
(or alongside) writing the report, update `outputs/site-notes/<domain>.md` per "Site context" under
Step 3 — append a dated section of durable, confirmed facts from this dive; create the file if it's the
domain's first dive.

### Shopping Dive: `<domain>`
**Dove:** `<date>` · **Stops:** `<n>` · **Path:** `Landing → Search → PDP → Cart → …` · **Lens:** conversion_architect
**Mode:** `<turn | auto | batch | drive>`, `checkin: <off | every N stops>`

**Shareable recap** — a compact block sized for pasting into Slack/email: the path walked, stop count, and
the **top 3 frictions** (one line each, tagged). First, so a skimmer gets the gist in five seconds. **Any
finding still marked unconfirmed per the viewport-stability guard should not occupy a top-3 slot** — rank it
by its confirmed severity, or below the confirmed findings if severity is itself uncertain.

**Summary** — 2–3 sentences: did the funnel get the shopper toward checkout, and where did it most resist?

**1. Funnel map** — the path actually walked, stop by stop, with a one-line read on momentum at each
(smooth / hesitation / stall / dead-end).

**2. Funnel storyboard** — a labeled visual sequence: for each stop, the screenshot
(`<screenshotDir>/stop-<n>.jpg`) with caption `Stop <n> · <Stage> · <one-line note>`. If the Claude-in-Chrome
`gif_creator` is available, also assemble an animated GIF of the walked funnel and link it. Where screenshots
are missing — whether by the checkout-privacy rule or because this environment doesn't persist screenshots to
disk — fall back to a **text storyboard** (captions alone), and say which reason applies.

**3. Findability** — how well search and navigation got the shopper to the right products. Cite search
behavior, facet usefulness, zero-result handling, nav clarity.

**4. Friction inventory** — every friction point, **ranked by estimated drop-off risk** (high → low). Each:
one line, a root-cause tag, and the stop it occurred at. Any unconfirmed/interaction-derived finding carries
its caveat inline, not just in the journal.

**5. Pattern callouts** — good patterns observed (reinforce) and anti-patterns (fix), named from
`references/pattern-library.md`, each with the stop and a root-cause tag.

**6. Drop-off risk points** — the 1–3 stops a real shopper would most likely abandon at, and why. Rank
unconfirmed findings below confirmed ones regardless of their nominal severity.

**7. Open questions** — the `?` quick-marks collected during the dive, each tied to its stop.

**8. Top recommendations** — prioritized by impact. Each names the specific page element / copy / flow to
change and carries a root-cause tag (the owning team). An unconfirmed finding gets a "verify first" action
item, not a "fix this" one.

Keep claims evidence-backed — point to the journal stops. Write the report to
`outputs/dive-report-<domain>-<YYYY-MM-DD-HHMM>.md` and give the user a 3–4 line summary plus the file path.
