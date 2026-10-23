# Changelog

## v0.9.0

- **Release prep.** MIT license; marketplace install docs; usage docs for `--drive`, `--checkin`, `--context`.
- **Site notes moved to `outputs/site-notes/<domain>.md`** (local, gitignored) — they're the user's own
  knowledge and must not live in the plugin install directory, which is overwritten on update.
- **Conversion lens and severity criteria inlined** — no dependency on external rubric files.
- **Checkout network privacy** — checkout stops record which events fired, never request URLs, query strings,
  or payloads.
- Generic acceptance scenario in `test-scenarios/dive-scenario.md`.
- All of the above mirrored into `portable/`.

## v0.8.0

- **`--context "<text>"`** — brief a dive with known site facts; actively checked at the relevant stop.
- **Persistent site notes** — durable per-domain facts appended at dive end and loaded on the next dive.
- **Adaptive check-in cadence** — the `--checkin` interval widens after clean check-ins; a self-sourced HIGH
  finding still forces an immediate pause.

## v0.7.0

- **`--drive`** — the agent shops the site itself (stopping before account/payment), observed by the same
  capture loop.
- **`--checkin[=N]`** — pause every N stops, and always before logging a HIGH finding from the agent's own
  interaction.
- **Viewport-stability guard** — viewport drift caused by the tooling downgrades confidence on layout
  findings instead of reporting them as site bugs.
- Tracking check no longer assumes GTM — looks for the site's actual analytics vendor before calling a gap.

## v0.6

- **WebMCP readiness check (informational)** — dive/sweep now note whether a form the shopper used carries
  real WebMCP declarative attributes (`toolname`/`tooldescription`, spec: github.com/webmachinelearning/webmcp,
  Chrome 149 origin trial). New pattern: *agent-ready form*. Adoption is near-zero right now, so absence is
  neutral, not scored as friction — presence is a notable callout. (Not to be confused with the unrelated
  `webmcp.dev` browser-extension project — this checks the real W3C/Chrome/Edge spec.)

## v0.5

- **GTM/dataLayer tracking check** — at PDP/Cart/Checkout, dive and sweep now check network requests for a
  GTM container and whether the stage's ecommerce event (`view_item`/`add_to_cart`/`begin_checkout`/
  `purchase`) actually fired, not just whether the UI updated. New pattern: *silent tracking gap*.
- **B2B/B2B2C funnel support** — PDP/Cart/Checkout sections now cover quote-based CTAs, account-gated
  pricing, spec-heavy content, and net-terms/PO checkout as normal structure, not friction. New patterns:
  *quote-based CTA*, *price wall with no context*.
- Mirrored into `portable/` (SKILL.md references + flattened Gemini prompt) per the parity rule.

## v0.3

- **Skill-format migration** — `/dive` moved from the legacy `commands/dive.md` to `skills/dive/SKILL.md`
  (commands merged into skills in Claude Code v2.1.3+). Same `/dive` invocation; cleaner structure, richer
  frontmatter, auto-discovered (no `commands` array).

## v0.2

- **Quick-marks** — 1–3 char tagged notes (`!` / `+` / `?` / `*`) with inferred severity + root-cause tag.
- **Easier driving** — opt-in hands-free `--auto` (under `/loop`), pause/resume via a state sidecar, and
  `--batch` multi-tab capture.
- **Visual funnel storyboard + shareable recap** — per-stop screenshots assembled into the report (optional
  GIF), plus a Slack/email-ready recap block and an Open Questions roll-up.
- All of the above are mirrored in `portable/` for Gemini in Chrome and Copilot Cowork.
