# Funnel stages

The seven stages of a shopping dive. For each: what it is, how to classify a page into it, what to capture,
and the canonical friction to watch for. Capture what actually happens — dives skip stages and loop back.

Classification is a judgment call from **URL shape + page structure (`read_page`) + visible copy
(`get_page_text`)**, not URL alone. When ambiguous, pick the stage that matches what the shopper is *trying
to do* at that moment.

**Works for B2C and B2B/B2B2C alike.** Spec-heavy B2B catalogs (bulk ordering, account-gated pricing,
"request quote" instead of "add to cart") walk the same seven stages — just read the PDP/Cart/Checkout
sections below for the B2B variant of each before calling something a gap.

**Tracking check (every stage, via GTM/dataLayer — or the site's actual analytics stack).** At each stop,
skim `read_network_requests` for a Google Tag Manager container request (`googletagmanager.com/gtm.js` or
similar) and the ecommerce `dataLayer` push it should trigger for that stage: `view_item` on PDP,
`add_to_cart` on Cart, `begin_checkout`/`purchase` on Checkout. **Not every site runs GTM.** If no GTM
container appears at all, don't default to "tracking is broken" — that conflates two different things:
- *No GTM present, but an equivalent event fires through the site's actual stack* (Adobe Experience
  Cloud/AppMeasurement `scAdd`/`scOpen`, Segment, a custom `dataLayer.push`, etc.) — this is a **positive**
  finding, logged as *clean dataLayer events* even though the vendor isn't GTM. Look for the add-to-cart/
  checkout action's own network call (e.g. an Adobe beacon whose query string carries the SKU, quantity, and
  price) as the evidence.
- *No GTM, and no equivalent event either* — this is the real gap, and still worth flagging, but say
  explicitly that you checked for a non-GTM equivalent and found none, rather than just noting "no GTM."
Note whether the event fired, not just whether the UI action worked — a silent tracking gap (UI works, no
event) is its own finding regardless of vendor. See *silent tracking gap* in `pattern-library.md`.

**Viewport-stability guard (every stage — read this before flagging any layout/interaction bug).** Automation
tooling (screenshot capture, window management, extension quirks) can resize the browser viewport between
tool calls without you asking it to. A responsive site will reflow — sometimes hiding or repositioning header
elements, nav, or controls — purely because of that resize, with nothing wrong on the site's end. This looks
identical, from inside a dive, to a genuine frontend bug.

- **Record the `Viewport` line from every `read_page` call.** If it changes between two calls in the same
  stop, or between consecutive stops, without you having intentionally resized anything, treat that as a
  signal, not noise.
- If a layout-related finding (an element vanishing, a control not responding, content not where expected)
  coincides with a viewport change, **downgrade its confidence explicitly in the journal** — log it as
  observed, but flagged, rather than asserting it as a confirmed defect.
- Retrying the same interaction inside the same dive session is **not** independent confirmation if the
  viewport was already unstable — you're just repeating the same conditions that may have caused the false
  signal in the first place. Real confirmation is either (a) the user reproducing it manually outside the
  automation session, or (b) reproducing it yourself after confirming the viewport held steady across the
  whole interaction.
- **Motivating case:** a July 2026 dive of a B2B reseller's storefront found what looked like a HIGH-severity bug — typing
  into the header search box appeared to make the entire nav disappear and Enter stopped submitting the
  search. It reproduced twice inside the dive. The user could not reproduce it afterward on a manual retest.
  Reviewing the journal's own `Viewport` readings showed they had drifted from 1071×833 to 1428×1111 to
  2139×1046 across the same stretch of calls, unexplained — the far more likely explanation is a responsive
  header reacting to the automation resizing the window, not a real site defect. The finding had to be walked
  back from HIGH to unconfirmed after the fact. This guard exists so that correction happens *before* the
  report ships, not after.

**WebMCP readiness check (informational, not a friction score).** At any stop with a form the shopper acted
on — search box (Findability), add-to-cart (PDP), cart (Cart), checkout (Checkout) — use `read_page` to
check whether the `<form>` carries WebMCP's real declarative attributes: `toolname`, `tooldescription`,
`toolparamdescription` on its fields, `toolautosubmit` (spec: github.com/webmachinelearning/webmcp; Chrome
149 origin trial as of mid-2026 — an imperative `navigator.modelContext` registration is the JS-only
alternative and won't show up in the DOM, so absence of `toolname` isn't proof of absence). If present, note
it as *agent-ready form* with the tool name. This is near-zero adoption in the wild right now — treat a
"no" as neutral, not as a friction point; treat a "yes" as a notable, callout-worthy finding. See
*agent-ready form* in `pattern-library.md`.

---

## 1. Entry / Landing

**Is:** The shopper's first page — homepage, campaign/PPC landing page, or a deep link from search/social.

**Classify when:** URL is root `/` or a marketing path (`/landing`, `/promo`, UTM params present); page is
hero-led with broad navigation rather than a product grid or single product.

**Capture:** value proposition / headline, primary paths offered (search, nav, featured categories),
above-the-fold clarity, any interstitial/popup that appears before content, load feel.

**Friction to watch:** unclear value prop `[STRATEGY]`; funnel-blocking popup/cookie wall before any content
`[CONVERSION]`; no obvious path to products `[UX DESIGN]`; slow/janky first paint `[FRONTEND INTEGRATION]`.

---

## 2. Findability (search + navigation)

**Is:** The shopper trying to *get to* the right products — typing in search, using autocomplete, or
drilling through nav/mega-menu.

**Classify when:** URL is a search results path (`/search?q=`, `/s?k=`) or the user is interacting with the
nav; page shows a query + results or a category drill-down in progress.

**Capture:** the query typed, whether autocomplete/suggestions appeared, result relevance (do top results
match intent?), zero-result handling, nav structure depth, breadcrumb presence. Use `read_page` for the
search box + nav tree; `get_page_text` for zero-result/empty copy. **This stage is where interaction bugs are
easiest to misdiagnose** — a search box is one of the first things a dive touches, often right after the
initial page load/resize settles. Apply the viewport-stability guard above before escalating anything here
past MED.

**Friction to watch:** no search box `[UX DESIGN]`; no autocomplete `[CONVERSION]`; irrelevant top results
`[DATA QUALITY]`; **zero-result dead end** with no suggestions/alternatives `[CONVERSION]`; deeply nested nav
requiring many clicks `[UX DESIGN]`; search that ignores synonyms/typos `[DATA QUALITY]`.

---

## 3. Category / Listing

**Is:** A grid/list of products — category page, collection, or search results acting as a listing.

**Classify when:** URL is a category/collection path (`/c/`, `/collections/`, `/category/`); page shows
multiple product cards with facets/sort.

**Capture:** facets/filters available and whether they're usable (counts, multi-select, mobile behavior),
sort options, product card content (price, rating, badge, image), grid density/scannability, pagination vs.
infinite scroll. `read_page` for facet panel + cards.

**Friction to watch:** no/weak faceted filtering `[UX DESIGN]`; facets without result counts `[UX DESIGN]`;
missing ratings on cards `[CONVERSION]`; missing/incomplete prices `[DATA QUALITY]`; sort missing relevant
options `[UX DESIGN]`; cluttered or sparse grid `[UX DESIGN]`.

---

## 4. PDP (product detail page)

**Is:** A single product. The decision page.

**Classify when:** URL is a product path (`/p/`, `/product/`, `/dp/`, `/products/`); page shows one product
with gallery, price, variants, add-to-cart.

**Capture:** image count/quality, description depth (substantive vs. thin/title-only), variant selection
clarity (size/color), price + any sale framing, review/rating presence and prominence, add-to-cart clarity
and stickiness, cross-sell/upsell, shipping/returns reassurance. `read_page` for ATC button + variant
controls; `get_page_text` for description/reviews. Confirm `view_item` (or the site's equivalent — see the
tracking check above) fired in `read_network_requests`.

**B2B/B2B2C variant:** primary CTA may be *Add to Quote* / *Request Quote* / *Add to List* instead of Add to
Cart — that's not friction by itself. Capture whether pricing is shown outright, requires login
("price on request"), or requires a minimum quantity; whether spec tables / datasheets substitute for
marketing copy (normal for B2B — don't flag as thin content unless the specs themselves are sparse); and
whether bulk/tiered pricing is visible.

**Friction to watch:** thin or title-only description `[DATA QUALITY]`; few images `[UX DESIGN]`; **no social
proof / reviews** `[CONVERSION]`; unclear/buried add-to-cart (or add-to-quote) `[CONVERSION]`; confusing
variant selection `[UX DESIGN]`; no shipping/returns info on PDP `[CONVERSION]`; no cross-sell `[STRATEGY]`;
price hidden behind login with no stated reason `[STRATEGY]`; `view_item`-equivalent never fires `[FRONTEND
INTEGRATION]`.

---

## 5. Cart / Bag

**Is:** The cart — review before checkout, or a slide-out mini-cart.

**Classify when:** URL contains `/cart`, `/bag`, `/basket`; or a cart drawer opened after add-to-cart.

**Capture:** does the item land correctly (right variant/qty), editability (qty, remove), totals
transparency (subtotal, shipping estimate, taxes), free-shipping-threshold nudge, upsell, prominence and
clarity of the checkout CTA, save-for-later/persistence. `read_network_requests` to confirm the `add_to_cart`
dataLayer event (or the site's equivalent) fired, not just that the UI updated. Also note whether any
post-add-to-cart upsell modal defaults to a pre-selected option, and whether declining it is given equal
visual weight to accepting it — a pre-selected expensive option with a de-emphasized decline path is worth
flagging even if it doesn't technically block checkout.

**B2B/B2B2C variant:** may be a *quote cart* or *order pad* rather than a checkout-bound cart — capture
whether it supports bulk-line entry (multiple SKUs/qty at once), PO number / net-terms fields, and approval
routing (cart requires manager sign-off before it becomes an order). None of these are friction on their own.

**Friction to watch:** add-to-cart didn't fire / wrong item `[FRONTEND INTEGRATION]`; opaque totals, no
shipping estimate before checkout `[CONVERSION]`; no free-shipping progress when a threshold exists
`[STRATEGY]`; weak/hidden checkout CTA `[UX DESIGN]`; aggressive upsell blocking checkout `[CONVERSION]`;
pre-selected upsell with asymmetric accept/decline weighting `[CONVERSION]`; `add_to_cart`-equivalent event
silently missing while the UI updates `[FRONTEND INTEGRATION]`.

---

## 6. Checkout

**Is:** The purchase flow — contact, shipping, payment, review.

**Classify when:** URL contains `/checkout`, `/payment`, `/order`; page shows forms for contact/shipping/
payment and an order summary. **Also classify an immediate redirect to an identity/SSO provider
(`id.<domain>`, `accounts.<domain>`, an OAuth `authorize` endpoint, etc.) as reaching this stage** — a login
wall that appears *instead of* a checkout form is itself the finding.

**Capture (structure only — never the user's entered values):** number of steps and whether progress is
shown, **guest checkout vs. forced account creation**, field count/burden, address autocomplete, payment
options (cards, wallets, BNPL), whether shipping cost/taxes/fees appear *now or as a surprise*, error
handling, persistent order summary. Use `read_network_requests` for step transitions, fee-calculation
calls, and whether `begin_checkout` / `purchase` dataLayer events (or the site's equivalent) fire at the
right steps. **Do not read or store personal/payment data** — observe the form, not its contents. **If the
browser tooling itself denies you access to a page (common on identity/login domains as a safety boundary),
respect that denial — don't work around it, and note the denial itself as evidence that you stopped
appropriately.**

**B2B/B2B2C variant:** payment options may include *net-terms / invoice / PO* alongside or instead of card —
capture whether that's offered and whether it requires an account. A credit-application or approval step
before order placement is normal for B2B; note it as structure, not friction. **The distinction that still
counts as friction:** an account/login gate can be structurally normal for B2B procurement, but if nothing on
the cart or checkout-entry screen *explains why* an account is required before the shopper hits the gate,
that lack of explanation is still worth flagging — the gate itself isn't the problem, the silence about it is.

**Friction to watch:** **forced account creation** (no guest, where a guest option would make sense)
`[CONVERSION]`; **surprise shipping/fees** revealed only at the end `[CONVERSION]`; too many steps / no
progress indicator `[UX DESIGN]`; excessive or redundant fields `[UX DESIGN]`; no address autocomplete
`[UX DESIGN]`; limited payment options `[STRATEGY]`; order summary missing or not persistent `[UX DESIGN]`;
unclear validation errors `[FRONTEND INTEGRATION]`; `begin_checkout`/`purchase`-equivalent events missing
`[FRONTEND INTEGRATION]`; account/login gate presented with no explanation of why it's required `[CONVERSION]`.

---

## 7. Confirmation

**Is:** Post-purchase confirmation (or the point the dive stops short of a real purchase).

**Classify when:** URL contains `/confirmation`, `/thank-you`, `/order/complete`; page confirms an order.

**Capture:** order confirmation clarity, what-happens-next (shipping/tracking expectation), reassurance,
account-creation offer framed as benefit, any post-purchase cross-sell. *Most real dives stop before a true
purchase — note that and infer the confirmation experience only if reached.*

**Friction to watch:** thin confirmation with no next-steps `[UX DESIGN]`; no order number/email reassurance
`[CONVERSION]`; pushy post-purchase upsell `[STRATEGY]`.
