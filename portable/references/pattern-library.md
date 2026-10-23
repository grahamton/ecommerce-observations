# Pattern library

Named, recognizable ecommerce patterns. Call these out **by name** in live callouts and the report so
findings are concrete and comparable across dives. Each entry: what it is, the stage it usually appears at,
and the root-cause tag to use when noting it.

Use the exact pattern **name** (italicized) in output, e.g. *missing social proof on PDP*, *forced account
creation*. If you see a pattern not listed here, name it descriptively and tag it — the library is a
starting set, not a closed list.

---

## Good patterns (reinforce)

| Pattern | Stage | What it is | Tag when noting |
|---|---|---|---|
| *faceted search* | Category | Multi-select filters with result counts that narrow the grid | `[UX DESIGN]` |
| *search autocomplete* | Findability | Live suggestions / typo tolerance as the shopper types | `[CONVERSION]` |
| *persistent / sticky cart* | All | Cart count visible and reachable from every page | `[UX DESIGN]` |
| *sticky add-to-cart* | PDP | ATC stays visible on scroll | `[CONVERSION]` |
| *guest checkout* | Checkout | Buy without creating an account | `[CONVERSION]` |
| *checkout progress indicator* | Checkout | Clear step-of-N progress | `[UX DESIGN]` |
| *free-shipping-threshold nudge* | Cart | "Add $X for free shipping" progress | `[STRATEGY]` |
| *social proof on PDP* | PDP | Ratings + reviews prominent on the product page | `[CONVERSION]` |
| *recently-viewed* | All | Easy return to products already seen | `[STRATEGY]` |
| *breadcrumbs* | Category/PDP | Clear path back up the hierarchy | `[UX DESIGN]` |
| *transparent totals early* | Cart | Shipping/taxes estimated before checkout | `[CONVERSION]` |
| *address autocomplete* | Checkout | Reduces typing and errors | `[UX DESIGN]` |
| *clean dataLayer events* | PDP/Cart/Checkout | The stage's ecommerce event (GTM's `view_item`/`add_to_cart`/`begin_checkout`/`purchase`, or the site's actual analytics vendor's equivalent, e.g. Adobe's `scAdd`) fires in step with the UI action | `[FRONTEND INTEGRATION]` |
| *quote-based CTA* | PDP/Cart | B2B: *Add to Quote*/*Request Quote* clearly replaces Add to Cart, with a visible next step | `[STRATEGY]` |
| *agent-ready form* | Findability/PDP/Cart/Checkout | Form carries real WebMCP declarative attributes (`toolname`/`tooldescription`) so browser agents can complete it reliably | `[FRONTEND INTEGRATION]` |

---

## Anti-patterns (fix)

| Anti-pattern | Stage | What it is | Tag when noting |
|---|---|---|---|
| *forced account creation* | Checkout | No guest option; must register to buy, with no explanation offered | `[CONVERSION]` |
| *surprise fees* / *hidden shipping* | Checkout | Costs revealed only at the final step | `[CONVERSION]` |
| *zero-result dead end* | Findability | Empty search with no suggestions or recovery path | `[CONVERSION]` |
| *no search* | Findability | No search box at all | `[UX DESIGN]` |
| *funnel-blocking interstitial* | Landing | Popup/cookie/email wall before any content | `[CONVERSION]` |
| *broken back button* | All | Back loses state / breaks the flow | `[FRONTEND INTEGRATION]` |
| *missing order summary* | Checkout | No persistent recap of what's being bought | `[UX DESIGN]` |
| *excessive checkout steps* | Checkout | Too many pages/fields, no progress shown | `[UX DESIGN]` |
| *missing social proof on PDP* | PDP | No ratings/reviews on the decision page | `[CONVERSION]` |
| *thin product content* | PDP | Title-only or near-empty description, few images | `[DATA QUALITY]` |
| *opaque totals* | Cart | No shipping/tax estimate before checkout | `[CONVERSION]` |
| *add-to-cart not firing* | Cart | ATC doesn't register / wrong item lands | `[FRONTEND INTEGRATION]` |
| *facets without counts* | Category | Filters give no sense of result size | `[UX DESIGN]` |
| *checkout-blocking upsell* | Cart/Checkout | Upsell that interrupts the path to pay, or defaults to a pre-selected paid option with a de-emphasized decline path | `[CONVERSION]` |
| *deep nav maze* | Findability | Too many clicks to reach products | `[UX DESIGN]` |
| *silent tracking gap* | PDP/Cart/Checkout | The stage's expected ecommerce event never fires (GTM's or the site's actual analytics vendor's equivalent) — the UI works, the data doesn't | `[FRONTEND INTEGRATION]` |
| *price wall with no context* | PDP | Price hidden behind login/account with no explanation offered (B2B "price on request" done badly) | `[STRATEGY]` |
