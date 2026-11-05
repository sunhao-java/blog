# Features Research

## Summary
The blog already ships a complete "read a post" surface: local search, Atom feed, sitemaps, OG/Twitter meta, TOC, code highlighting, fancybox images, prev/next navigation, year archives with a post calendar, tag cloud, categories page, 404 page, and a working Actions deploy. Gaps are not in publishing basics but in *discovery between posts* (no series grouping, no related posts, no reading time) and *feedback* (zero analytics configured). Three liabilities also stand out: a Node 12 build for a Hexo 7 site, ~26 references to jsdelivr/bootcss CDNs, and an enabled-but-unmaintained Valine comment stack with app keys committed. Given the mainline (low-friction writing, not feature maximalism), the highest-value work is a small number of build-time, JS-free additions plus build-pipeline hardening — not new interactive features.

## Findings

### Confirmed capabilities (repo evidence)
| Area | What exists | Evidence |
|---|---|---|
| Content | Hexo 7.3, 88 Markdown posts in 10 category folders; 75/88 use `<!-- more -->`; 0 use a `description:` field | `package.json`, `find source/_posts -name '*.md'` |
| Pagination | Index 12/page; archives 12/page with yearly+monthly grouping and an ECharts post calendar | `_config.yml`, `layout/_partial/archive.ejs`, `post-calendar.ejs` |
| Navigation | Menu: home/archives/tags/about; prev+next post links; floating prev/next plus a 999-entry mini archive; tag cloud page; categories page | `themes/yelee/_config.yml`, `post/nav.ejs`, `post-nav-button.ejs`, `tag-cloud-page.ejs` |
| Search | Build-time `search.xml` (full post content) + client UI, index fetched lazily (`onload: false`) | `_config.yml`, `source/js/search.js`, `left-col.ejs` |
| Feeds/SEO | Atom feed (20 items), `sitemap.xml`, `baidusitemap.xml`, `robots.txt`, 404 page, `open_graph()` emitting og:* + twitter:card with description fallback (excerpt → first 200 chars) | `_config.yml`, `head.ejs`, `source/robots.txt`, `source/404/index.md` |
| Reading | TOC (max depth 3, collapsible, `list_number: false`), highlight.js with line numbers, copyable copyright block, fancybox lightbox, password-protected posts | `toc.ejs`, `_config.yml`, `post.ejs` |
| Engagement | Valine comments enabled (app id/key in repo), busuanzi visit counters, reading progress bar, scroll animations, GitHub repo widget | `themes/yelee/_config.yml`, `after-footer.ejs`, `article.ejs` |
| Analytics | **None active** — `google_analytics` and `baidu_tongji` are empty | `themes/yelee/_config.yml`, `head.ejs` |
| Deploy | GitHub Actions: `hexo clean && generate && deploy` on push to master, Node 12, no cache | `.github/workflows/deploy.yml` |
| Theme ownership | Theme is vendored (103 tracked files, no submodule), so patching is frictionless | `git ls-files themes/yelee` |

### Gaps that matter for an 88-post archive
- **No series/collection concept.** Categories are coarse topic buckets (中间件 20, 后端 17, 运维 15, 数据库 12, 工具 10) and tags are free-form; nothing links the posts that belong to one sequence (e.g. RAG, HBase, xxl-job).
- **No related posts.** Readers only move via chronologically adjacent posts or search.
- **No reading time / no per-post summary.** 13 posts render in full on list pages; `description` is absent everywhere, so list excerpts and OG descriptions depend on `<!-- more -->` placement and truncated body text.
- **No canonical link, no `<html lang>`, no JSON-LD.** `head.ejs` sets `lang` only when `page.lang` exists and never emits a canonical URL; social previews work, structured data does not.
- **No image lazy-loading.** 53 Markdown images, zero `loading=`/`lazyload` in theme output; `post_asset_folder: false` means images live outside post folders.
- **Search is a linear substring scan** over a full-text XML with no empty-state or match highlighting; acceptable at 88 posts, not extensible.
- **No feedback loop.** Without analytics there is no evidence about which of the 88 posts are actually read.
- **Third-party fragility:** 16 CDN assets from `cdn.jsdelivr.net` plus a pace theme from `cdn.bootcss.com`; Valine depends on a LeanCloud China endpoint and ships on every post page.

## Recommendations

Effort: **S** ≤ half day, **M** 0.5–2 days. "Serves" marks whether it reduces writing→publishing friction (W) or improves reader discovery/reading (R).

| # | Change | Effort | Impact | Serves |
|---|---|---|---|---|
| 1 | Fix CI: Node 12 → 20, `npm ci`, cache `~/.npm` and `node_modules`; add a build-fails-loudly check | S | High — a broken deploy blocks every publish | W |
| 2 | Add a `hexo new` scaffold script (title/date/categories/tags/series/description prompts) + a tiny pre-push front-matter check | S | High — removes the main per-post authoring tax | W |
| 3 | Series index: adopt `series:` front-matter, generate `/series/:name/` pages and a series box (all parts + prev/next within series) in the post layout. No plugin, no client JS | M | High — the single biggest discovery win for a 88-post archive | R |
| 4 | Related posts at build time by tag/category overlap (top 5, deduped) in the post footer | S–M | Medium-high — keeps readers inside a topic without JS | R |
| 5 | Reading time + word count from `post.content`, and a front-matter `description` used for list cards and OG | S | Medium — better scanning, better social/search snippets | W+R |
| 6 | SEO polish: canonical URL, `<html lang="zh-CN">`, default OG image, `BlogPosting` JSON-LD | S | Medium — long-tail Chinese search traffic | R |
| 7 | `loading="lazy" decoding="async"` + intrinsic dimensions on rendered images (marked renderer hook or a post-filter), and enable `post_asset_folder` for new posts | S | Medium — image-heavy posts on mobile | R |
| 8 | Lightweight analytics: self-hosted Umami/Plausible or Baidu Tongji behind a config flag; keep it off the critical path | S | Medium — tells you what to write next | W |
| 9 | De-risk assets: self-host the handful of theme CSS/JS files or add a fallback for jsdelivr; remove the dead `cdn.bootcss.com` pace reference | S–M | Medium — pages currently depend on CDNs that are unreliable in mainland China | R |
| 10 | Search UX: empty state, keyword highlighting, and index excerpts instead of full text | S | Low-medium — polish, not a gap until ~200 posts | R |

### Explicitly not worth it (given the mainline)
| Candidate | Why not |
|---|---|
| Comments / community (Valine) | Unmaintained stack, extra JS and cold-start on every post, keys committed, spam surface. It serves discussion, not writing or reading. Disable unless historical comments are actively answered. |
| Social share buttons | Baidu share endpoints are legacy/HTTP; share volume for a personal Chinese tech blog is negligible. |
| Algolia/Meilisearch, instant-search service | Build-time local search is adequate at this scale and adds an external dependency plus index sync. |
| Headless CMS / SPA rewrite | Directly contradicts the "Markdown in git → Actions → Pages" mainline. |
| Newsletter, membership, PWA/offline, i18n | No audience signal or content need; each adds maintenance with no writing-friction payoff. |
| More theme JS (widgets, animation frameworks, chat) | Reading performance is already taxed by jQuery/require/fancybox/pace/scrollreveal; adding is negative value. |

## Open Questions
1. Are the existing Valine comments read or answered? If yes, keep as read-only archive; if no, remove the section and its assets rather than migrate.
2. Is `categories` the intended series axis, or is a separate `series` field preferable (categories are coarse and multi-purpose)?
3. Does the Actions deploy currently succeed on Node 12 with Hexo 7, or is `deploy.yml` already failing silently? (No `node_modules` locally to verify a build.)
4. Which decision should analytics inform — topic selection, series planning, or nothing? If nothing, skip analytics entirely.
5. Is mainland-China page load (jsdelivr/bootcss) an observed problem or a theoretical one? This decides whether self-hosting assets is P1 or P3.
