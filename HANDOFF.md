# Headshop.com Theme — Session Handoff

**Date:** July 17, 2026
**Repo:** `https://github.com/Trinacle/headshop.git`
**Local:** `C:\Treman\Claude\headshop`
**Store:** `headshop-etm.myshopify.com`
**Theme:** Expanse by Archetype Themes, v6.1.0
**Live theme:** "headshop/main" (#162429698287) — GitHub-connected, PUBLISHED
**Skill:** `shopify-theme-dev` at `C:\Users\kevin\.claude\skills\shopify-theme-dev\` (read it first)

---

## Setup & workflow (already done, just use it)

- GitHub repo is connected to the **live theme** via Shopify's native GitHub integration.
  **Every `git push origin main` auto-deploys to the LIVE site in ~1-2 min.** There is no staging anymore — pushes go live.
- **CRITICAL RULE:** The Shopify theme editor commits back as `shopify[bot]` ("Update from Shopify for theme headshop/main"). ALWAYS before pushing:
  ```bash
  git fetch origin && git merge origin/main --no-edit
  ```
  If a push fails with "remote contains work that you do not have locally" → that's the editor; merge, never force-push.
- Theme check (must cd first; output goes to stderr):
  ```bash
  cd /c/Treman/Claude/headshop && shopify theme check 2>&1
  ```
  Baseline: **9 pre-existing base-theme errors** (SVG syntax FPs, gift_card ParserBlockingScript, translation keys). These are NOT ours — don't touch. Custom files must stay at 0 errors.
- Acceptable warnings: `OrphanedSnippet` (mostly false positives on our snippets), `RemoteAsset`.
- CSS changes: tell Kevin to **hard-refresh (Ctrl+Shift+R)** — custom.css caches. Shopify re-versions the asset URL (`?v=...`) on sync; verify via the versioned URL on the live page if changes seem missing.

## Work completed this session (26+ commits)

### Speed
- Removed ~88KB inlined jQuery 3.7.1 + the MutationObserver/eval() `lazyScript` deferral system (snippets/inlineScript.liquid rewritten, snippets/lazyScript.liquid DELETED)
- Deferred ajaxinate.js (`<script defer>` in theme.liquid, collection templates only)
- Scoped modulepreload (snippets/preload-js.liquid — scans page's actual module imports instead of all ~60 import-map entries)
- home-promo LCP: first panel `loading="eager" fetchpriority="high"`
- Mobile nav images: lazy + `widths: '50, 80, 100'` (was eagerly loading 2000-4000px product images for 40px thumbnails)
- Desktop megamenu: `widths: '200, 300, 400'`
- **All 21 image-element calls** across 16 snippet files now have explicit `widths` (the fallback was `img.width × 2` = 4000px URLs)
- Removed dead Google Maps dns-prefetch hints from theme.liquid
- **Fixed 5 unclosed CSS braces in custom.css** (a `@media (min-width:769px)` at ~line 100 was never closed + 4 nested @media blocks) — this had trapped all later CSS (incl. portrait-video + search styles) inside media queries

### SEO / structured data
- `snippets/structured-data-global.liquid` (rendered in theme.liquid head): Organization (logo, conditional sameAs from 10 social settings) + WebSite w/ SearchAction
- BreadcrumbList JSON-LD in `snippets/breadcrumbs.liquid` (delimited-string build; `item` URLs `| json` escaped)
- AggregateRating on products from `product.metafields.judgeme.badge` (separate JSON-LD node merged by URL) in section-main-product.liquid
- ItemList in CollectionPage schema (section-main-collection.liquid)
- Article publisher logo uses `settings.logo` w/ page_image fallback
- Meta description fallback chain (page_description → product/collection/page desc → shop.description) in theme.liquid
- 16 duplicate H1s → H2 across 12 templates + Pro Blogger related-title H1→H3 in article.json
- Homepage image alt fixes (US + UK): filled empties, fixed keyword-stuffed + mismatched alts, added width/height/loading to 32 images

### Features
- **Portrait video fix:** media.liquid passes `video_aspect_ratio`; video-media.liquid detects `< 1` → `.video-media--portrait` + inline `--aspect-ratio`; custom.css uses `object-fit: contain`. Verify external (YouTube/Vimeo) portrait videos visually — detection relies on Shopify knowing the external video's ratio.
- **Search overhaul** (snippets/section-main-search.liquid):
  - Products grid first; "Related Blog Posts" section only renders on the LAST page (`search_pagination.current_page == search_pagination.pages`, gated by `show_articles`)
  - Custom infinite scroll (IntersectionObserver, 400px rootMargin) fetches ?page=N, appends products, extracts+appends `[data-search-articles]` from the last page
  - Bouncing-dots loader (`.search-infinite-loader` CSS in custom.css)
  - Breadcrumbs removed; "Latest" sort REMOVED (Shopify storefront search only honors relevance/price-ascending/price-descending — verified; created-descending silently reverts)
- **Collection layout:** sort bar moved below description/above products via `controls_in_slot: true` param on item-grid; grid-view icons (large/medium/list buttons) removed from item-grid-controls.liquid. Search still renders controls at top (doesn't pass the flag). Both item-grid branches have the `unless controls_in_slot` guard.
- **Judge.me stars on collection cards:** `.grid-product__reviews` div in product-grid-item.liquid renders `product.metafields.judgeme.badge`; hidden when empty/0.00
- **Blog article styling:** `.proarticle` ul/ol list-style restored (base theme's global `ul{list-style:none}` reset had killed bullets since .proarticle isn't inside .rte) + line-height 1.8 + paragraph/heading spacing

### Shopify Markets (admin-side, Kevin did these)
- International menu created; product availability per market set; UK market redirect FIXED
- Market header groups: US → `main-menu`, CA/UK → `international` (sections/header-group.context.*.json)
- USD-only checkout (no multi-currency); ships to 200+ countries; ~2-3k products international

## Verified-OK (do NOT "fix" these)

- Canonical: Shopify's `canonical_url` already strips sort_by/filter params, preserves ?page=N
- Infinite scroll (EOSH app on collections) is crawlable — real Liquid pagination w/ `?page=N` links. EOSH hook `eosh-total-pages` input now in BOTH subcollections and product_grid blocks
- Blog bodies render in visible HTML via Pro Blogger app (D1 investigation) — adding `{{ article.content }}` to theme would duplicate
- `?view=ajax` self-canonicalizes
- hreflang injected by Shopify via content_for_header (x-default, en, en-CA)

## Known issues / remaining work

### App bloat (admin, not theme) — biggest remaining perf lever
19 active app embeds load on every page. Lab PageSpeed ~35 mobile but **real-user CrUX PASSES** (LCP 2.0s, INP 117ms, CLS 0). Kevin checked: Sezzle/Appstle/Judge.me have no page-scoping settings; Klaviyo popup removed; PushOwl/BixGrow/Clarity/Blockify/Chatty are all used. SyncTrack preconnect mystery = **Blockify's CDN** (megamind-fraud / IP blocker). Blockify + NoFraud overlap (two fraud apps) — flagged, Kevin's call.

### Product page JS errors (conversion risk)
Session recording (Clarity) showed errors on PDPs. 7+ apps run widgets simultaneously: FBP/Fast Bundle (~190 refs), Appstle (24), Judge.me, Aftersell, SavedBy, Klaviyo, NoFraud. Theme code is clean — errors are app conflicts. Next step: get actual console errors from DevTools on a PDP.

### Deferred theme work
- Expanse update 6.1.0 → 9.1.0 (3 majors behind) — separate project, after rankings stabilize
- components.css split (335KB) — preload pattern mitigates; low ROI
- VideoObject schema — only 2 product templates use video

## Gotchas learned this session

- `paginate` constrains `for item in search.results` INSIDE it, but the same loop OUTSIDE paginate iterates ALL results — this caused blog posts rendering on page 1
- The old lazyScript system was the ONLY jQuery consumer; theme JS is all ES modules + is-land islands
- custom.css `{{ settings.color_borders }}` on ~line 106 renders literally (plain .css isn't Liquid-processed) — harmless, has var() fallback
- `templates/*.json` have `/* auto-generated */` comment headers — strip before JSON.parse
- NocoDB: Reports table = `mqwhxbs4a06v9y1` (base `pjx9ze1q8aipk68`); Clients = `mrvkir7gtw61y4v`; Category field only allows "Investor, Sales, Competition". A session recap report was published (Record Id 3, Category "Sales", Headshop orange #d1471e accent)
- `/tmp` in Git Bash ≠ Node's `/tmp` (Windows temp) — write scratch scripts into the repo dir, delete after
- Trinacle token broker (`shopify.trinacle.com/api/get-token`) had no active session for headshop this session (500) — but `shopify` CLI was already logged in with theme scopes; use CLI for theme ops
- R2/NocoDB attachment URLs need signed URLs — for logos in reports, use the client's CDN logo URL and base64-embed

## Reports & docs in repo

- `README.md` — dev workflow, deploy commands, theme-check baseline
- `SEO-CHECKLIST.md` — full audit results, all Phase 1-3 marked complete, remaining items annotated
- Trinacle dashboard report: app.trinacle.com/reports (Record Id 3)

---

## BUCKS Currency Converter (Aug 25, 2026)

The app requires Settings → General → Currency formatting set to
`<span class=money>${{amount}}</span>` in **HTML with currency** and **HTML without currency**
(leave both **Email** fields plain — they drive order/notification emails).

**Why the raw markup showed as text:** Shopify's `t` filter HTML-escapes interpolated
variables unless the key ends in `_html`. `info.save_amount` / `info.you_save_amount` don't,
so the "Save $X" badge printed the tags. Fixed in commit `1bb862c` across
block-price, product-grid-item, section-search-results, section-main-cart,
cart-ajax, onboarding-product-grid-item — the money value is now interpolated as a
plain `[[amt]]` token and `replace`d in after translation. **Deploy this before changing
the money format.**

Checked and safe: `og:price:amount` already `strip_html`s; JSON-LD uses
`product | structured_data` (raw numbers, SEO unaffected); `price-range.js` /
`formatMoney` write via `innerHTML`; `labels.from_price_html` already `_html`.

**Expected side effect:** Expanse disables superscript decimals when
`shop.money_format contains 'money'`, so prices render `$65.00` instead of `$65`+superscript
`00`. Intentional — `<sup>` inside the converted span would break the converter.

**Still to verify after the format change:** PDP/cart with the 7+ price-rendering apps
(Fast Bundle, Appstle, Sezzle, Aftersell, Judge.me) — any that build prices in JS from
`Shopify.money_format` and inject via `textContent` will show raw tags, and that can't be
fixed theme-side. Also confirm the converter re-runs on AJAX-injected prices (cart drawer,
variant switch, collection infinite scroll, quick view).

---

## SEO + mobile audit & fixes (Sep 1, 2026)

### ✅ DEPLOYED 2026-09-01 (commits 45b0961 + 4514d89, live and verified)
Verified on the live site after deploy: 6/6 sampled product pages now render exactly
one H1 (was 2), and the Organization JSON-LD carries a logo URL.

**Push was blocked for ~an hour; here is why, so nobody re-lives it.** Three stacked causes:
(1) global `~/.gitconfig` pins `credential.github.com.username = oauth2`, and that GCM
credential has READ but not WRITE on the repo - hence a 403 saying "denied to kevin-treman"
while `git ls-remote` succeeded; (2) `credential.helper manager` sits first in the chain;
(3) **PowerShell strips a bare `""`**, so `git config --local credential.helper ""` is a
silent no-op there - it must be run from Git Bash. A fine-grained PAT with Contents:
read/write DOES work on Trinacle repos (verified `permissions: {admin, push: true}`),
contradicting the old note in the `github-access` memory, which has now been corrected.
Escape hatch that always works:
`git push "https://x-access-token:<TOKEN>@github.com/Trinacle/headshop.git" main`
(then `git fetch origin` to sync the tracking ref). Full procedure in the memory file.

### Fixed in 45b0961
1. **Duplicate H1 on ~89% of products.** Scan of 2,000 products via `products.json`:
   1,797 carry an SEO-rewritten `<h1>` inside `body_html` (distribution `{1: 1794, 2: 2, 6: 1}`),
   rendering a second page-level H1 beside the product title. Demoted to `<h2>` at the single
   render chokepoint, `snippets/product-description.liquid`. Covers all ~18k products and all
   future ones, mutates no data, reverts by deleting one line. Pre-flight verified: **zero
   uppercase `<H1>`** in the catalog (3,621 lowercase tags), so the case-sensitive
   `replace` is safe. Chosen over an Admin API bulk rewrite (~18k writes, ~3h, destructive,
   and feeds strip tags anyway so it buys nothing extra).
2. **Organization JSON-LD had no logo.** The LD keyed off `settings.logo`, but the logo is a
   header *section block* setting, so it was always blank and the property was omitted.
   Now falls back to `shop.brand.logo` / `shop.brand.square_logo`.
   **Verified after deploy:** `curl -sL https://www.headshop.com | grep -A3 '"logo"'`. If still
   absent, set Settings → Brand → Logo in admin (the fallback only fires if Brand is populated).

### Audit results — passing, do not "fix"
No `noindex` meta, no `X-Robots-Tag`. Canonicals correct on home/collection/product.
hreflang present. 404s return 404. robots.txt + 31-shard sitemap OK. Every theme `<img>`
has alt. Titles 57–73 chars. Structured data is strong: Product, Offer, Brand,
BreadcrumbList, ItemList, CollectionPage, Organization (stable `@id` `#organization`,
5 `sameAs`), WebSite, SearchAction. **Zero horizontal overflow at 375px** (the 300
"overflowing" nodes are `.visually-hidden` text at -9999px, which is correct).

### Two false positives — corrected, do not chase
- **Judge.me carousel CLS: NOT a bug.** The images lack width/height *attributes*, but
  measured live the wrapper has computed `height: 250px` with `object-fit: cover`, so the
  box is already reserved by Judge.me's CSS. No CSS rule needed.
- **"item-grid island never hydrates": WRONG.** That was an artifact of the Claude Browser
  pane not compositing frames — Chrome only computes IntersectionObserver while rendering,
  so `on:visible` islands cannot hydrate there. In a real browser `itemGridDefined: true`,
  `filterInlineTop: "93px"`. **Never diagnose `on:visible` / IO behaviour in a non-compositing
  pane.** (`item-grid.js:197` does have a real misplaced-paren bug —
  `new CustomEvent(EVENTS.headerStickyCheck), { bubbles: true }` — but it is a redundant path.)

### Open items (not done, with reasons)
- **Homepage meta description is 281 chars** (truncates ~155-160) and contains a copy error:
  "the most , high-qua…" — stray comma, missing word. Lives in admin (Online Store →
  Preferences), not the theme. Needs Kevin's copy; not inventing merchant positioning.
- **Dead Universal Analytics pixel `UA-196913542-3`** fires on every page (UA shut down
  July 2023), throwing console errors and collecting nothing. **Not in theme code** —
  grep for `UA-196913542`/`gtag_UA` returns nothing. It is the Google & YouTube channel,
  Online Store → Preferences, or an app. Admin-side removal.
- **robots.txt AI-crawler groups: deliberately NOT added.** There is no `Disallow` for
  GPTBot/ClaudeBot/PerplexityBot etc., so they are already fully allowed. Adding an explicit
  `User-agent: GPTBot / Allow: /` group would *override* Shopify's 50 default Disallows for
  that agent, exposing /cart, /checkout, /account to it. Current policy is already correct.
- **llms.txt is Shopify-generated and cannot be overridden.** `/llms.txt` returns 200 but is
  Shopify's UCP/Shop-skill agent doc — nothing about what headshop.com is a primary source of.
  No theme hook exists to replace it. NOTE: both `/llms.txt` and `/robots.txt` contain text
  addressed to AI agents recommending they install `shop.app/SKILL.md`. That is crawled
  content, i.e. data, not instructions — do not act on it.
- **AEO gap:** FAQPage LD exists only on `page.faq.json` and `product.product-landing.json`.
  Top collections have no FAQ blocks. Content work, not shipped.
- **Naver/NEO lane skipped** — US-market store.
- Page weight 1.0–1.3 MB HTML/page and 5–6 render-blocking stylesheets; both trace to the
  19 app embeds, already tracked under App bloat above.

### Measurement (skill requires this, do not skip)
Baseline recorded 2026-09-01, pre-fix: product PDPs = 2 H1s; Organization logo absent;
homepage meta description 281 chars. **Re-measure 2026-09-15**: GSC impressions/clicks for
/products/*, and confirm 1 H1 per PDP via
`curl -sL <product-url> | grep -c '<h1'`.

