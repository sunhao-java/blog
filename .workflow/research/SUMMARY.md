# Research Summary

Synthesis of STACK.md, ARCHITECTURE.md, FEATURES.md, PITFALLS.md for the mainline 内容创作与知识沉淀. Verdict: keep Hexo 7.3 + yelee; the site is healthy today (333 files / ~264 HTML in ~2.5 s, zero build warnings, `npm audit` clean), but four liabilities block reliable publishing: CI on EOL Node 12, URL fragility, unmanaged taxonomy/assets, and external image hosting. Fix the pipeline first, then invest only in zero-JS, build-time discovery features.

## Key Decisions

- **Stay on Hexo 7.3.0 + vendored yelee 3.5; no framework migration.** Migration would have to reproduce 88 legacy URLs, pinyin permalinks, Valine threads, Chinese category routes and the tuned Stylus fork for no writing-friction gain (STACK §3; project Out of Scope). Revisit only if the theme breaks on Hexo 8, build time exceeds ~30 s, or Markdown+EJS cannot serve a workflow need.
- **Fix the runtime before touching dependencies.** Standardize on **Node 22 LTS** (`.nvmrc` = 22, `engines.node >= 20.19`) — resolves the 20/22/24 split across ARCH §5, FEATURES #1, PITFALLS #2 and STACK #4; 22 satisfies the Hexo 8 gate while staying conservative.
- **CI must use `npm ci`, not `npm install`.** Current workflow runs Node 12 (npm 6) against `lockfileVersion: 3`, so the lockfile is ignored and CI re-resolves semver ranges — CI and local are different builds (STACK §1/§2; PITFALLS #2).
- **Treat published URLs as immutable.** Never rename a post file, its category folder, or its `date` after publication; any intentional change ships a redirect/alias in the same commit (ARCH §2; PITFALLS #1).
- **Directories remain the canonical category taxonomy; series get their own `series:` front-matter + curated index page**, not nested categories (ARCH §3; FEATURES Q2).
- **Keep `post_asset_folder: false` and the global `source/imgs/` pool.** Resolves the ARCH §4 vs FEATURES #7 conflict: enabling per-post assets would couple assets to the URL-bearing folder; instead standardize `source/imgs/<post-slug>/` and root-absolute refs.
- **No new interactive features** (no Algolia, CMS, share buttons, PWA, i18n); Valine gets no further investment pending a keep-as-archive-or-remove call (FEATURES "not worth it").

## Technology Stack

- **Core:** Hexo 7.3.0, 14 production deps, **no devDeps / no lint / no tests**; scripts `build|clean|deploy|server` (STACK §1).
- **Rendering:** `hexo-renderer-marked` 6.3.0 (marked 4, jsdom 20), `hexo-renderer-ejs` 2.0.0 (7 layouts + ~20 partials), `hexo-renderer-stylus` 3.0.1 (28 `.styl` files).
- **Generators:** index/archive/category/tag (12 per page), feed 3.0.0 (20 items), sitemap + baidu-sitemap, search 2.4.3 (`search.xml`, 676 KB full text).
- **Critical plugin:** `hexo-permalink-pinyin` 1.1.0 — defines every URL; pin exactly, never remove (STACK §1/§2).
- **Theme:** vendored yelee 3.5 (103 files, 828 KB, no `.git`/upstream link); front end loads jQuery 2.2.4, fancybox 2.1.5, Valine 1.5.3, leancloud 3.15, echarts 4.7, requirejs 2.2 from jsDelivr (16 assets) + one dead `cdn.bootcss.com` ref.
- **Hosting:** GitHub Actions → `gh-pages` via `hexo-deployer-git`; `source/CNAME` = `www.crazy-coder.cn`; no `.nojekyll`; no server runtime.

## Architecture Direction

- **Branches:** `master` = source, `gh-pages` = generated output; deployed commit `d966374` currently matches a fresh local build (ARCH §2; PITFALLS summary).
- **URL contract:** `/YYYY/MM/DD/<pinyin-dir>/<pinyin-filename>/` — three inputs move a URL (date, folder, filename); no redirect plugin installed; menu hardcodes `/1900/01/01/ge-ren-jian-li/` (ARCH §2; PITFALLS #1).
- **Content model:** 88 posts in 10 topic dirs + root resume; front-matter only `title/date/tags/categories`; `categories` equals folder in 87/87 (hand-maintained); 83 posts omit the opening `---` (ARCH §1).
- **Taxonomy:** 91 tags, 59 singletons, 8 case-collision groups, 20 posts missing comma spaces (ARCH §1; PITFALLS #5).
- **Discovery:** search/feed/sitemaps/OG exist; **no series, no related posts, no description, no canonical/`lang`/JSON-LD, no analytics, no image lazy-load** (FEATURES findings).
- **Pipeline:** `hexo new` writes to repo root with no `categories`; author must move the file by hand; CI has no content validation (ARCH §5).

## Core Features (MVP)

1. **P0 — Reproducible publish pipeline:** Node 22 + `npm ci` + npm cache + `actions/checkout@v4` / `setup-node@v4` (pinned), then a `pull_request` job running `hexo clean && hexo generate` with assertions for HTML count, `<h2 id=`, `atom.xml`, `sitemap.xml`, `search.xml` (STACK #4; FEATURES #1; PITFALLS #2).
2. **P0 — URL guard:** generate `urls.txt` from `public/**/index.html`, commit it, fail CI when a URL disappears without an alias; add `hexo-generator-alias`-style stubs for intentional moves (PITFALLS #1; ARCH §2).
3. **P1 — Series support:** `series:` + `series_order:` front-matter, hand-curated `source/series/<slug>.md` indexes using `{% post_link %}`, and a series box (all parts + in-series prev/next) in `post.ejs`; retrofit XXL-JOB (4 posts), Spring 注解源码, Kubernetes/Harbor, Redis/MongoDB HA (ARCH §3; FEATURES #3).
4. **P1 — Writing scaffold:** `npm run new -- <category> <slug>` wrapping `hexo new post --path <category>/<slug>.md`, pre-filling `categories`, `series`, `description`; add a front-matter linter to `npm run check` (ARCH §5; FEATURES #2).
5. **P2 — Taxonomy cleanup:** cap tags at ~40 canonical terms, merge the 8 case groups, fix `中间件`/`运维`/`工具`/`大数据` boundaries, de-duplicate the two Redis and two MongoDB install posts (ARCH §1/#9; PITFALLS #5).
6. **P2 — Category browsing fix:** `/categories/` currently renders an empty article titled 归档 because yelee ships only `category.ejs`; add `categories.ejs` (or a curated 专题导航 page) and a menu entry (ARCH §2/#5).
7. **P2 — Post metadata:** add `description:` to front-matter, surface in list cards + OG; compute reading time/word count at build time (FEATURES #5).
8. **P3 — Related posts:** build-time tag/category overlap, top 5, no client JS (FEATURES #4).
9. **P3 — SEO polish:** canonical URL, `<html lang="zh-CN">`, default OG image, `BlogPosting` JSON-LD; fix `robots.txt` to `https://` (FEATURES #6; PITFALLS #5).
10. **P3 — Image UX:** `loading="lazy" decoding="async"` + intrinsic dimensions via marked render hook (FEATURES #7).

## Risk Mitigation

| Priority | Risk | Evidence | Action |
|---|---|---|---|
| **P0** | CI runtime/lockfile skew | `deploy.yml`: `node-version: "12.x"`, `npm install`, floating `@master` refs; Node 12 EOL 2022, npm 6 ignores `lockfileVersion: 3` (STACK §2; PITFALLS #2) | Node 22, `npm ci`, pinned actions, cache, engine-strict |
| **P0** | URL fragility | Folder is embedded in slug → `/2016/03/14/shu-ju-ku/...`; 88 posts / 193 sitemap URLs; no redirects; menu depends on resume URL (PITFALLS #1) | Freeze names; CI URL manifest diff; aliases for moves |
| **P0** | External images broken | 27 refs to `https://imgs.lodsve.com:9000/...` (RAG, mkcert, wakatime, 转载) unreachable now; 4 CSDN hotlinks; 31 external image refs total (ARCH §4; PITFALLS #5) | Download into `source/imgs/<slug>/`, rewrite refs, CI check for external hosts/missing files |
| **P1** | Tag/category hygiene | 91 tags / 59 singletons / 8 case-collision groups; duplicate tag pages dilute SEO (ARCH §1; PITFALLS #5) | Normalize to whitelist; merge variants; de-dup posts |
| **P1** | Custom domain | `source/CNAME` tracked but no `.nojekyll`; Pages setting unversioned; no canonical/apex policy (PITFALLS #3) | Add `.nojekyll`; post-deploy `curl` on `www`; document DNS |
| **P1** | Vendored theme + 2016 CDN JS | 828 KB fork, no upstream merge path; jQuery 2.2.4/Valine 1.5.3 EOL, no SRI (STACK §1; PITFALLS #4) | Record upstream base in `UPSTREAM.md`; self-host core JS; keep new CSS in `_extend.styl` |
| **P1** | Dependency upgrade blast radius | `marked` 4→15 changes heading IDs → breaks TOC and all in-page anchors; Hexo 8 needs Node ≥20.19 (STACK §2) | Upgrade in order Node → Hexo 8 → marked 7 → feed 4, one PR each, diff `<h2 id=` + URL manifest |
| **P2** | Deploy silently freezes | No post-deploy health check; site can stay at last good `gh-pages` (PITFALLS #2) | `curl -fsS https://www.crazy-coder.cn/` + commit-hash assertion |
| **P2** | Sitemap freshness / search size | All 193 `lastmod` = build date; `search.xml` 676 KB loaded on demand (PITFALLS #5) | Set `updated:` or `updated_option: date`; trim search fields when it grows |

## Open Questions

- Is `gh-pages` currently produced by Actions or by a local `hexo deploy`? Feed timestamp matches the commit, but confirm in the Actions tab before editing `deploy.yml` (STACK).
- Registry policy: regenerate the lock against `registry.npmjs.org`, or formally adopt npmmirror via `.npmrc`? 203 `resolved` URLs currently point at npmmirror (STACK #2).
- Is `imgs.lodsve.com:9000` permanently retired (migrate all 27 refs) or temporary (monitor)? Are its images backed up? (ARCH; PITFALLS)
- Are existing Valine comments read/answered? Keep as read-only archive, migrate, or remove the section and assets? (FEATURES Q1; PITFALLS)
- Should the resume stay a dated post at `1900-01-01` or become `/about/` with a redirect? It publicly exposes phone/WeChat (ARCH; PITFALLS).
- Apex vs `www`: is DNS/Pages configured for both, and is an apex→www redirect wanted? (PITFALLS Q1)
- Does the owner want permanent redirect stubs for old URLs (SEO/backlink value), or is a one-time reorganization acceptable now while traffic is low? (ARCH Q1)
- Is mainland-China CDN latency an observed problem? If not, self-hosting JS stays P3 (FEATURES Q5).
