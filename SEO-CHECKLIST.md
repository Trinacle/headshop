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
- [ ] ⚪ **A6. Article publisher logo fix** — `snippets/section-article-template.liquid:254-262`
      uses `page_image` for publisher logo; should use shop logo.
- [ ] ⚪ **A7. VideoObject** — `featured-video`/`hero-video`/`promo-video` sections emit
      no video schema. Add if video content matters.
- ✅ Product schema (Product + Offer) present via `structured_data` filter.
- ✅ FAQPage schema present (`snippets/section-faq.liquid:34-54`).
- ✅ Article schema present (`snippets/section-article-template.liquid:226-266`).

## B. On-page SEO (title, headings, meta)

- [x] ✅ **B1. Remove duplicate H1 tags** — DONE. 16 custom-liquid `<h1>` → `<h2>`
      across 12 templates + Pro Blogger related-products title H1 → H3 in article.json.
      Every page now has exactly one H1.
- [ ] 🟡 **B2. Meta description fallback** — `layout/theme.liquid:20` only emits
      `<meta name="description">` when `page_description` exists. Add fallback to
      `product.description` / `collection.description` / `shop.description`.
- [ ] 🟡 **B3. Image alt text** — 6 empty `image_alt` fields in template JSON;
      several keyword-stuffed / mismatched alts in custom-liquid blocks.
      Audit and fix all `alt=""` and stuffed alts.
- ✅ Title tag dynamic per template with pagination/tag suffixes (`snippets/page-title.liquid`).
- ✅ Open Graph + Twitter cards complete (`snippets/social-meta-tags.liquid`).
- ✅ No accidental `noindex`/`nofollow` anywhere in theme (only password page).

## C. Crawlability & canonicalization

- [x] ✅ **C1. Canonical hardening** — NOT NEEDED. Verified live: Shopify's
      `canonical_url` already strips `sort_by`/`filter.v.*` and preserves `?page=N`
      correctly. No theme change required.
- [ ] 🟠 **C2. hreflang tags** — multi-market store (US/UK/Canada per `config/markets.json`)
      but zero `<link rel="alternate" hreflang>` in theme. Add in `layout/theme.liquid`
      via `localization.available_countries`.
- [ ] 🟡 **C3. Verify `view=ajax` not indexable** — AJAX renderer fetches `?view=ajax`
      (`assets/item-grid.js:335`). Confirm these aren't indexed.
- ✅ Infinite scroll is crawlable — real server-side pagination with `?page=N` links.
- ✅ Pagination is self-canonical (not collapsed to page 1).
- ✅ Internal links are real `<a href>` (product grid, blog, collections, search).

## D. Content rendering (blog)

- [ ] 🔴 **D1. Render `{{ article.content }}` in the theme** —
      `snippets/section-article-template.liquid` never outputs the article body; it's
      injected by the Pro Blogger app. If the app fails, crawlers see no body text.
      Add `{{ article.content }}` to the article template so content is always in HTML.

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
- [ ] 🟠 **E4. Fix `home-promo` above-the-fold images** — `sections/home-promo.liquid:184-193`
      uses `loading="lazy"` on top-of-page promo panels. Change first row to
      `loading="eager" fetchpriority="high"` + preload primary image. (Our section.)
- [ ] 🟡 **E5. Scope `modulepreload`** — `snippets/preload-js.liquid` preloads all ~60
      modules on every page. Limit to modules needed for current template.
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
