# headshop.com — Shopify Theme

The production theme for **headshop.com** (`headshop-etm.myshopify.com`).

- **Base theme:** Expanse by Archetype Themes, v6.1.0
- **Store:** `headshop-etm.myshopify.com`
- **Staging theme:** `Trinacle | 6.1.0 Theme 7/16` (#162429042927)
- **Live theme:** `Trinacle | 6.1.0 Theme Update-Optimize-24-4-2025` (#150622896367)

## Local dev

Clone + run the dev server (hot-reload preview at `127.0.0.1:9292`):

```bash
git clone https://github.com/Trinacle/headshop.git
cd headshop
shopify theme dev --store headshop-etm.myshopify.com
```

> The `shopify theme dev` login is interactive — run it in your own terminal, not
> the agent sandbox. After edits, **hard-refresh the preview (`Ctrl+Shift+R`)** —
> CSS assets cache aggressively.

## Deploying to the staging theme

```bash
shopify theme push --store headshop-etm.myshopify.com --theme 162429042927
```

Use `--theme 162429042927` (the staging copy) so nothing goes live by accident.
The live theme (#150622896367) only gets updated when you're ready to publish.

## Theme check

Must run from inside this directory (else it scans the parent and reports other
projects' errors). The report goes to **stderr**:

```bash
cd /c/Treman/Claude/headshop && shopify theme check 2>&1
```

Baseline: 11 pre-existing errors from the Archetype base theme (SVG syntax
false-positives, `ParserBlockingScript` in layout, translation keys). These are
**not ours** — leave the base plumbing alone. Custom files must stay at 0 errors.

## GitHub ↔ Shopify sync

If you connect this repo to the staging theme via Shopify admin
(Online Store → Themes → Add theme → Connect from GitHub), the theme editor
commits back as `shopify[bot]`. **Always fetch + merge before pushing** — never
force-push over editor edits:

```bash
git fetch origin && git merge origin/main
```

## Custom sections

| Section | Purpose |
|---------|---------|
| `sections/home-promo.liquid` | 4-panel promo banner, full editor control (50 settings). Prefixed `homepromo-`. |
