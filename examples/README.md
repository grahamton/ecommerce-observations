# Example dives

Real `/dive` output from public storefronts, unedited apart from the privacy pass below. Each folder holds the
journal, the synthesized report, and the per-stop screenshots that make up the storyboard.

| Folder | Store | Type | What it shows |
|---|---|---|---|
| `b2c-target/` | Target | Mass-market B2C | Search-heavy findability, promo/fee disclosure, guest checkout, tracking events across a big retail stack |
| `b2b-uline/` | Uline | B2B | Quantity pricing, catalog-number search, spec-heavy PDPs, account vs. guest checkout — the B2B patterns that generic audit tools misread as friction |

> These are independent observations of publicly accessible pages, made for illustration. They aren't
> affiliated with or endorsed by the stores named, and they reflect the site as it was on the dive date.

## How these were produced

```
/dive https://www.uline.com/
/dive https://www.target.com/
```

Ride-along mode (a human shopping, `/dive` observing), stopping before account creation or payment.

## Privacy pass before committing

- Use a logged-out browser profile with no saved addresses or payment methods.
- Check every screenshot for personal data (name, email, address, order numbers, location/store pickers
  that reveal where you live) and crop or drop it.
- Checkout stops should show structure only. If a screenshot shows typed values, remove it and keep the
  text storyboard entry.
- Strip the sidecar (`.dive-state-*.json`) and any `site-notes/` file. They're working state, not examples.
