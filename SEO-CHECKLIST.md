# headshop.com — SEO & Speed Checklist

Goal: top-ranking, fast, crawlable store. Baseline audit done 2026-07-16 against
Expanse theme v6.1.0. Items marked 🔴 are likely ranking-loss contributors.

Legend: 🔴 critical · 🟠 high · 🟡 medium · ⚪ low · ✅ verified OK

---

## A. Structured data / schema (rich results + entity understanding)

- [x] ✅ **A1. Organization schema** — DONE. `snippets/structured-data-global.liquid`,
      rendered in `layout/theme.liquid`. Name, url, logo, sameAs (conditional social).
- [x] ✅ **A2. WebSite + SearchAction schema** — DONE. Same snippet. SearchAction
      targets the search page for sitelinks searchbox eligibility.
- [x] ✅ **A3. BreadcrumbList JSON-LD** — DONE. `snippets/breadcrumbs.liquid` now
      emits BreadcrumbList JSON-LD alongside the visible nav for all template types.
- [ ] 🟡 **A4. AggregateRating / Review on products** — `snippets/section-main-product.liquid:73`
      uses Shopify's `structured_data` filter (Product + Offer) but no ratings. Judge.me
      is installed — configure it to inject `AggregateRating`, or add manually.
- [ ] 🟡 **A5. ItemList on collection pages** — `snippets/section-main-collection.liquid:97`
      emits thin `CollectionPage`. Add `ItemList` enumerating products.
- [x] ✅ **A6. Article publisher logo fix** — DONE. Now uses `settings.logo`
      (shop logo); falls back to `page_image` only if no logo set.
- [ ] ⚪ **A7. VideoObject** — `featured-video`/`hero-video`/`promo-video` sections emit
      no video schema. Add if video content matters.
- ✅ Product schema (Product + Offer) present via `structured_data` filter.
- ✅ FAQPage schema present (`snippets/section-faq.liquid:34-54`).
- ✅ Article schema present (`snippets/section-article-template.liquid:226-266`).

## B. On-page SEO (title, headings, meta)

- [x] ✅ **B1. Remove duplicate H1 tags** — DONE. 16 custom-liquid `<h1>` → `<h2>`
      across 12 templates + Pro Blogger related-products title H1 → H3 in article.json.
      Every page now has exactly one H1.
- [x] ✅ **B2. Meta description fallback** — DONE. Falls back through
      `page_description` → product/collection/page description → `shop.description`.
- [x] ✅ **B3. Image alt text** — DONE. Filled 4 empty `image_alt` + 2 empty
      `logo_alt` fields in `index.json` home-promo panels. No stuffed alts found.
- ✅ Title tag dynamic per template with pagination/tag suffixes (`snippets/page-title.liquid`).
- ✅ Open Graph + Twitter cards complete (`snippets/social-meta-tags.liquid`).
- ✅ No accidental `noindex`/`nofollow` anywhere in theme (only password page).

## C. Crawlability & canonicalization

- [x] ✅ **C1. Canonical hardening** — NOT NEEDED. Verified live: Shopify's
      `canonical_url` already strips `sort_by`/`filter.v.*` and preserves `?page=N`
      correctly. No theme change required.
- [x] ✅ **C2. hreflang tags** — NOT a theme fix. Shopify Markets already injects
      hreflang via `content_for_header` (x-default, en, en-CA confirmed live). UK
      market (en-GB) tag is missing because `/en-gb/` redirects to `/` — the UK
      market needs fixing in **Shopify admin → Settings → Markets**, not the theme.
      Adding theme-level hreflang would duplicate Shopify's auto-injected tags.
- [x] ✅ **C3. Verify `view=ajax` not indexable** — VERIFIED OK. `?view=ajax` returns
      a full page but its canonical points to the clean collection URL, so it
      self-corrects. Low SEO risk.
- ✅ Infinite scroll is crawlable — real server-side pagination with `?page=N` links.
- ✅ Pagination is self-canonical (not collapsed to page 1).
- ✅ Internal links are real `<a href>` (product grid, blog, collections, search).

## D. Content rendering (blog)

- [x] ✅ **D1. Article content rendering** — VERIFIED OK (no change needed). Pro
      Blogger server-renders the article body in visible HTML (~9,800 chars, 26 `<p>`).
      Google can read it. Adding `{{ article.content }}` to the theme would duplicate.
      Left as-is; revisit only if blog rankings don't recover.

## E. Speed / Core Web Vitals

- [x] ✅ **E1. Remove inlined render-blocking jQuery** — DONE. ~88KB jQuery 3.7.1
      removed from `snippets/inlineScript.liquid`. Biggest first-paint blocker gone.
- [x] ✅ **E2. Retire the homegrown MutationObserver+`eval()` deferral system** — DONE.
      `snippets/lazyScript.liquid` deleted; MutationObserver removed from inlineScript.
      Apps now load as Shopify intends (no 8s delay).
- [x] ✅ **E3. Defer `ajaxinate.js`** — DONE. `theme.liquid` now loads it with
      `<script defer>`. Resolved the theme.liquid ParserBlockingScript error.
- [x] ✅ **E4. Fix `home-promo` above-the-fold images** — DONE. First panel image+logo
      now `loading="eager" fetchpriority="high"`; rest stay lazy.
- [x] ✅ **E5. Scope `modulepreload`** — DONE. `snippets/preload-js.liquid` now scans
      the page's actual `<script type="module">` imports and preloads only those,
      instead of all ~60 import-map entries.
- [ ] 🟡 **E6. Split `components.css` (335KB)** — audit for unused section styles.
- ✅ JS architecture is modern (ES modules, import maps, `is-land` islands).
- ✅ Images use Shopify CDN resizing, `srcset`, `width`/`height` (low CLS).
- ✅ Fonts use `font-display: swap` + preconnect (no FOIT).
- ✅ `theme.js` deferred; third-party scripts (Audiohook, AddShoppers) async.

## F. Theme version

- [ ] ⚪ **F1. Update Expanse 6.1.0 → 9.1.0** — 3 major versions behind. DO THIS LAST,
      after SEO fixes stabilize rankings, as a separate clean project. Major versions
      restructure sections and can overwrite customizations. Review release notes:
      https://help.archetypethemes.co/hc/en-us/articles/46027072690835-Expanse-release-notes

---

## Recommended order of attack

**Phase 1 — Stop the bleeding (SEO-critical, low risk):**
1. D1 — render article content in theme (blog indexation)
2. B1 — fix duplicate H1s (on-page relevance)
3. A1 + A2 — add Organization + WebSite/SearchAction schema
4. C1 — canonical hardening (duplicate content)

**Phase 2 — Speed wins:**
5. E1 + E2 — remove jQuery + custom deferral system
6. E4 — fix home-promo LCP images
7. E3 — defer ajaxinate

**Phase 3 — Rich results + international:**
8. A3 — BreadcrumbList schema
9. C2 — hreflang
10. A4 — product ratings schema (via Judge.me config)
11. B2, B3 — meta description fallback, alt text

**Phase 4 (separate project):**
12. F1 — theme update to 9.1.0
