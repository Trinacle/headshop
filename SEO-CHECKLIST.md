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
- [x] ✅ **A4. AggregateRating / Review on products** — DONE. Product schema now
      appends an `AggregateRating` node from Judge.me metafields
      (`product.metafields.judgeme.badge`) when reviews exist. Emitted as a separate
      JSON-LD node that Google merges with the Product block by URL. Enables review
      stars in SERPs. (If no products have reviews yet, the block is silently omitted.)
- [x] ✅ **A5. ItemList on collection pages** — DONE. CollectionPage schema now
      includes an `ItemList` enumerating products on the current page (url + name).
- [x] ✅ **A6. Article publisher logo fix** — DONE. Now uses `settings.logo`
      (shop logo); falls back to `page_image` only if no logo set.
- [ ] ⚪ **A7. VideoObject** — LOW PRIORITY. Video sections (`featured-video`,
      `hero-video`, `promo-video`) exist only on 2 product templates
      (`product.product-landing`, `product.puffco-plus-v3`) and there are no videos
      on the homepage. Add VideoObject schema only if video content is actively used
      on those templates.
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
- [ ] 🟡 **E6. Split `components.css` (335KB)** — DEFERRED. The preload pattern
      (`preload: true`) already makes it non-blocking. Splitting would require
      identifying per-section CSS blocks and creating per-template stylesheets —
      high effort with regression risk. Low marginal gain over the jQuery removal
      and modulepreload scoping already done. Revisit if PageSpeed still flags it.
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

**Phase 1 — Stop the bleeding (SEO-critical):** ✅ DONE
1. ✅ D1 — article content verified OK (Pro Blogger renders in HTML, no change needed)
2. ✅ B1 — fixed duplicate H1s (16 H1→H2 + Pro Blogger H1→H3)
3. ✅ A1 + A2 — Organization + WebSite/SearchAction schema added
4. ✅ C1 — canonical verified OK (Shopify handles it)

**Phase 2 — Speed wins:** ✅ DONE
5. ✅ E1 + E2 — removed jQuery (~88KB) + custom deferral system
6. ✅ E4 — home-promo LCP images (eager + fetchpriority)
7. ✅ E3 — deferred ajaxinate.js
8. ✅ E5 — scoped modulepreload to used modules only

**Phase 3 — Rich results + polish:** ✅ DONE
9. ✅ A3 — BreadcrumbList schema
10. ✅ A4 — AggregateRating from Judge.me metafields
11. ✅ A5 — ItemList on collection pages
12. ✅ A6 — article publisher logo fix
13. ✅ B2, B3 — meta description fallback + alt text fixes
14. ✅ C2, C3 — hreflang (admin issue) + view=ajax (verified OK)

**Phase 4 (separate project, do later):**
15. ⬜ F1 — theme update to 9.1.0 (after rankings stabilize)
16. ⬜ E6 — split components.css (high effort, preload already mitigates)
17. ⬜ A7 — VideoObject (only if video content is actively used)
18. ⬜ Admin: fix UK market (`/en-gb/` redirects to `/`) in Shopify Settings → Markets
