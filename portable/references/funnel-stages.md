# Funnel stages

The seven stages of a shopping dive. For each: what it is, how to classify a page into it, what to look at,
and the canonical friction to watch for. Capture what actually happens — dives skip stages and loop back.

Classification is a judgment call from **URL shape + what's on the page + the visible copy**, not URL alone.
When ambiguous, pick the stage that matches what the shopper is *trying to do* at that moment.

> Observation note (platform-neutral): wherever this says "inspect the page," use whatever your environment
> gives you — reading the live tab's structure and text, or the page content / screenshots the user shares.
> If you can see network/analytics activity (e.g. add-to-cart events firing), note it; if not, skip it.

---

## 1. Entry / Landing

**Is:** The shopper's first page — homepage, campaign/PPC landing page, or a deep link from search/social.

**Classify when:** URL is root `/` or a marketing path (`/landing`, `/promo`, UTM params present); page is
hero-led with broad navigation rather than a product grid or single product.

**Look at:** value proposition / headline, primary paths offered (search, nav, featured categories),
above-the-fold clarity, any interstitial/popup that appears before content, load feel.

**Friction to watch:** unclear value prop `[STRATEGY]`; funnel-blocking popup/cookie wall before any content
`[CONVERSION]`; no obvious path to products `[UX DESIGN]`; slow/janky first paint `[FRONTEND INTEGRATION]`.

---

## 2. Findability (search + navigation)

**Is:** The shopper trying to *get to* the right products — typing in search, using autocomplete, or
drilling through nav/mega-menu.

**Classify when:** URL is a search results path (`/search?q=`, `/s?k=`) or the user is interacting with the
nav; page shows a query + results or a category drill-down in progress.

**Look at:** the query typed, whether autocomplete/suggestions appeared, result relevance (do top results
match intent?), zero-result handling, nav structure depth, breadcrumb presence.

**Friction to watch:** no search box `[UX DESIGN]`; no autocomplete `[CONVERSION]`; irrelevant top results
`[DATA QUALITY]`; **zero-result dead end** with no suggestions/alternatives `[CONVERSION]`; deeply nested nav
requiring many clicks `[UX DESIGN]`; search that ignores synonyms/typos `[DATA QUALITY]`.

---

## 3. Category / Listing

**Is:** A grid/list of products — category page, collection, or search results acting as a listing.

**Classify when:** URL is a category/collection path (`/c/`, `/collections/`, `/category/`); page shows
multiple product cards with facets/sort.

**Look at:** facets/filters available and whether they're usable (counts, multi-select, mobile behavior),
sort options, product card content (price, rating, badge, image), grid density/scannability, pagination vs.
infinite scroll.

**Friction to watch:** no/weak faceted filtering `[UX DESIGN]`; facets without result counts `[UX DESIGN]`;
missing ratings on cards `[CONVERSION]`; missing/incomplete prices `[DATA QUALITY]`; sort missing relevant
options `[UX DESIGN]`; cluttered or sparse grid `[UX DESIGN]`.

---

## 4. PDP (product detail page)

**Is:** A single product. The decision page.

**Classify when:** URL is a product path (`/p/`, `/product/`, `/dp/`, `/products/`); page shows one product
with gallery, price, variants, add-to-cart.

**Look at:** image count/quality, description depth (substantive vs. thin/title-only), variant selection
clarity (size/color), price + any sale framing, review/rating presence and prominence, add-to-cart clarity
and stickiness, cross-sell/upsell, shipping/returns reassurance.

**Friction to watch:** thin or title-only description `[DATA QUALITY]`; few images `[UX DESIGN]`; **no social
proof / reviews** `[CONVERSION]`; unclear/buried add-to-cart `[CONVERSION]`; confusing variant selection
`[UX DESIGN]`; no shipping/returns info on PDP `[CONVERSION]`; no cross-sell `[STRATEGY]`.

---

## 5. Cart / Bag

**Is:** The cart — review before checkout, or a slide-out mini-cart.

**Classify when:** URL contains `/cart`, `/bag`, `/basket`; or a cart drawer opened after add-to-cart.

**Look at:** does the item land correctly (right variant/qty), editability (qty, remove), totals
transparency (subtotal, shipping estimate, taxes), free-shipping-threshold nudge, upsell, prominence and
clarity of the checkout CTA, save-for-later/persistence. If you can see it, confirm the add-to-cart event
fired.

**Friction to watch:** add-to-cart didn't fire / wrong item `[FRONTEND INTEGRATION]`; opaque totals, no
shipping estimate before checkout `[CONVERSION]`; no free-shipping progress when a threshold exists
`[STRATEGY]`; weak/hidden checkout CTA `[UX DESIGN]`; aggressive upsell blocking checkout `[CONVERSION]`.

---

## 6. Checkout

**Is:** The purchase flow — contact, shipping, payment, review.

**Classify when:** URL contains `/checkout`, `/payment`, `/order`; page shows forms for contact/shipping/
payment and an order summary.

**Look at (structure only — never the user's entered values):** number of steps and whether progress is
shown, **guest checkout vs. forced account creation**, field count/burden, address autocomplete, payment
options (cards, wallets, BNPL), whether shipping cost/taxes/fees appear *now or as a surprise*, error
handling, persistent order summary. **Do not read or store personal/payment data** — observe the form, not
its contents.

**Friction to watch:** **forced account creation** (no guest) `[CONVERSION]`; **surprise shipping/fees**
revealed only at the end `[CONVERSION]`; too many steps / no progress indicator `[UX DESIGN]`; excessive or
redundant fields `[UX DESIGN]`; no address autocomplete `[UX DESIGN]`; limited payment options `[STRATEGY]`;
order summary missing or not persistent `[UX DESIGN]`; unclear validation errors `[FRONTEND INTEGRATION]`.

---

## 7. Confirmation

**Is:** Post-purchase confirmation (or the point the dive stops short of a real purchase).

**Classify when:** URL contains `/confirmation`, `/thank-you`, `/order/complete`; page confirms an order.

**Look at:** order confirmation clarity, what-happens-next (shipping/tracking expectation), reassurance,
account-creation offer framed as benefit, any post-purchase cross-sell. *Most real dives stop before a true
purchase — note that and infer the confirmation experience only if reached.*

**Friction to watch:** thin confirmation with no next-steps `[UX DESIGN]`; no order number/email reassurance
`[CONVERSION]`; pushy post-purchase upsell `[STRATEGY]`.
