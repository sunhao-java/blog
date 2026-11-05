# Stack Research

## Summary

The blog is a conventional Hexo static site: Hexo 7.3.0 + 14 npm dependencies, Markdown authored in `source/_posts` (88 `.md` files across 10 Chinese-named category folders), rendered through a **vendored** yelee 3.5 theme (EJS + Stylus, jQuery/RequireJS front end) and deployed from `master` to `gh-pages` by one GitHub Actions workflow. The stack is healthy today — I verified `npm ci && hexo generate` on Node 24.19.0: 333 files / 264 HTML in 2.55 s, no warnings, `npm audit` reports 0 vulnerabilities — but it is **pinned to a stale CI runtime and a stale lockfile contract**: the workflow installs Node **12.x** (EOL April 2022, ships npm 6) while the committed lockfile is `lockfileVersion: 3`, and all 203 `resolved` URLs point at `registry.npmmirror.com`. Hexo itself is one major behind (latest 8.1.2, requires Node ≥20.19). For a content-creation mainline this is fine to keep, provided the runtime skew is fixed before any dependency movement.

## Findings

### 1. Current stack and plugin inventory

- `package.json`: `private: true`, no `engines`, no `.nvmrc`; 4 scripts (`build/clean/deploy/server`); 14 production deps, **no devDeps, no test/lint tooling**.
- Plugins and what each actually does here:
  - `hexo` 7.3.0 — generator core; `hexo.version` field also set.
  - `hexo-renderer-ejs` 2.0.0 — required by the theme's 7 EJS layouts + ~20 partials.
  - `hexo-renderer-stylus` 3.0.1 (stylus 0.62 + nib) — compiles 28 `.styl` files, entry `themes/yelee/source/css/style.styl`.
  - `hexo-renderer-marked` 6.3.0 — Markdown; bundles `marked@4.3`, `jsdom@20`, `dompurify@3`; emits heading IDs (`<h2 id="一、什么是知识库">`) that the theme TOC links to.
  - `hexo-generator-index` / `archive` / `category` / `tag` (all v2/v3) — paginated routes; `per_page: 12` set both globally and in `index_generator`/`archive_generator`.
  - `hexo-generator-feed` 3.0.0 — `atom.xml`, limit 20, `content_limit: 140`.
  - `hexo-generator-sitemap` 3.0.1 + `hexo-generator-baidu-sitemap` 0.1.9 — `sitemap.xml`, `baidusitemap.xml`.
  - `hexo-generator-search` 2.4.3 — `search.xml` index consumed by the theme's client-side search (`theme.search.on`).
  - `hexo-permalink-pinyin` 1.1.0 — **load-bearing**: every post URL is pinyin-transliterated (`/2026/04/17/ai/rag-zhi-shi-ku-ji-zhu-fen-xiang/`).
  - `hexo-deployer-git` 4.0.0 + `hexo-server` 3.0.0 — local `hexo deploy` path; CI actually uses `hexo deploy` too, so this plugin is on the critical deploy path.
- Theme: `themes/yelee` is **vendored into the main repo** (103 tracked files, 828 KB, no `.git`, no `package.json`, no README), version declared only as `Yelee: "3.5"` in its `_config.yml`. Customization hooks that already exist: `source/css/_extend.styl` plus `_config.yml` (menu, Valine, pinyin menu entry, ICP number, CDN map). Front-end libs are loaded from jsDelivr at 2016-era versions: jQuery 2.2.4, fancybox 2.1.5, Valine 1.5.3, leancloud-storage 3.15.0, echarts 4.7.0, requirejs 2.2.0.
- Deployment: single job on push to `master` → checkout → setup-node 12.x → `npm install hexo-cli -g` + `npm install` → write deploy RSA key from secret → `hexo clean && hexo generate && hexo deploy` (branch `gh-pages`, remote `git@github.com:sunhao-java/blog.git`). `source/CNAME` carries `www.crazy-coder.cn`.

### 2. Currency and upgrade risk

- **Hexo 7.3.0 → 8.1.2**: Hexo 8 moves to `hexo-util@4`, `warehouse@6`, `hexo-fs@5` and requires Node ≥20.19. Theme-level API changes are the risk area for a 2017-era theme, not the CLI.
- **`hexo-renderer-marked` 6.3.0 → 7.0.1** is the single riskiest bump: marked 4 → 15 also moves jsdom 20 → 25 and requires Node ≥18. Heading-ID generation changed across marked majors; all 88 posts' in-page anchors, the theme TOC (`toc.js` reads `#toc .toc-link` hrefs) and any external deep links depend on the current IDs. Verify by diffing `<h2 id=` output before/after.
- **`hexo-generator-feed` 3 → 4** requires Node ≥20.19 (same gate as Hexo 8).
- **Unmaintained but working, leave alone**: `hexo-generator-search` 2.4.3 (last publish 2019, declares `engines: >=0.10`), `hexo-generator-baidu-sitemap` 0.1.9 (2018, never updated), `hexo-permalink-pinyin` 1.1.0 (2018). The last one must not be removed — it defines all existing URLs.
- **What breaks easily today**: the CI runtime. Node 12 ships npm 6, which cannot honor `lockfileVersion: 3`, so `npm install` re-resolves from semver ranges instead of the lockfile — CI and local builds are not the same build. It currently succeeds (live `atom.xml` `<updated>` is `2026-09-28T17:04:25Z`, matching the last commit's timestamp, and the deployed archive matches the newest post `2026-04-17`), but only because Hexo 7.3's dist happens to avoid post-Node-12 syntax. Any Hexo 8 / feed 4 / marked 7 upgrade hard-fails on `engines` and modern syntax.
- Other CI hygiene gaps: `actions/checkout@master` and `actions/setup-node@master` are floating refs; no `npm ci`, no npm cache, no `concurrency` group, no build job on pull requests, no post-build assertion that `atom.xml`/`sitemap.xml`/`search.xml` exist.
- Automation is abandoned rather than absent: no `.github/dependabot.yml`, but **5 stale `dependabot/npm_and_yarn/*` branches** still exist on `origin` (acorn, lodash, y18n, two multi-bumps) — dependency PRs were opened and never merged.
- Local/CI skew is real: this machine runs Node 24.19.0 / npm 12 (which blocks `hexo-util` and `ejs` postinstall scripts by default) and still builds fine; CI runs Node 12. No `.nvmrc`, no `engines`, so nothing declares which is intended.

### 3. Stay on Hexo, or migrate?

**Stay.** The site is 88 posts / 264 static pages built in 2.5 s with zero runtime — scale is nowhere near where a generator's performance matters. The real asset is the tuned yelee fork: Valine comment threads bound to a specific LeanCloud app, pinyin permalinks, Chinese-labeled category routes, client-side search, in-page TOC, ICP footer, custom Stylus. A migration would have to reproduce all of that and map 88 legacy URLs one-for-one, risking SEO and existing comment threads for no content-creation benefit. Hugo/Astro/Eleventy would buy faster builds and a modern authoring pipeline (components, image optimization), but those are feature work — contrary to the current mainline. Revisit only on a concrete trigger: theme becomes unmaintainable against Hexo 8, build time crosses ~30 s, or a posting workflow need appears that Markdown+EJS cannot serve.

### 4. Maintenance recommendations

1. **Fix the runtime first, before touching deps.** Add `.nvmrc` (`22`) and `"engines": { "node": ">=20.19" }`; in CI use `actions/setup-node@v4` with `node-version-file: .nvmrc` and `cache: npm`, switch `npm install` → `npm ci`, and pin `actions/checkout@v4`. That alone makes CI reproduce local builds.
2. **Decide the registry explicitly.** 203 lockfile `resolved` URLs are `registry.npmmirror.com`; `npm ci` will fetch from them regardless of machine locale. Either regenerate the lock against `registry.npmjs.org` for portable CI, or add an `.npmrc` documenting the mirror choice (fast in CN, single point of failure elsewhere). Do not leave it implicit.
3. **Cadence.** Monthly `npm outdated` review (read-only), quarterly batched minor/patch bump via one `npm ci`-verified PR. Majors land one at a time in this order: Node → `hexo@8` → `hexo-renderer-marked@7` → `hexo-generator-feed@4`. Each PR must pass a checklist: 88 post URLs unchanged, heading IDs/TOC intact, `atom.xml` + `search.xml` + both sitemaps present, Valine loads.
4. **Add a PR build gate.** A second workflow on `pull_request` running `hexo clean && hexo generate` plus assertions (HTML count, `grep -q '<h2 id='`, feed/sitemap presence). This is the cheapest protection for a content-first repo where breakage arrives via a stray Markdown construct, not via code.
5. **Theme fork hygiene.** Since the theme lives in-tree with no upstream binding, add `themes/yelee/UPSTREAM.md` recording the base (yelee 3.5, upstream author, base date) and every local edit; keep new CSS in `source/css/_extend.styl` and behavior/config in `_config.yml`, avoiding edits to `_partial/*.ejs` so future upstream diffs stay readable. If partial edits become frequent, split the theme into its own repo/submodule.
6. **Address the 2016 CDN surface separately.** jQuery 2.2.4 / fancybox 2.1.5 / Valine 1.5.3 / leancloud 3.15 are all EOL and third-party. Self-host at least jQuery + Valine (or move comments to a maintained system) so a CDN or LeanCloud policy change cannot blank the site; keep the CDN map in `_config.yml` as the single switch.
7. **Housekeeping.** Delete the 5 stale dependabot branches and either add a grouped weekly `dependabot.yml` (prod deps only, `npm` ecosystem) or explicitly rely on manual cadence — an empty middle state is the worst option. Keep `hexo-generator-baidu-sitemap` (still builds) but treat it as frozen.

## Open Questions

- Is `gh-pages` still produced by this workflow, or was the last deploy a local `hexo deploy`? The feed timestamp and commit time match, implying CI, but confirm in the Actions tab before rewriting `deploy.yml`.
- Intended Node baseline: 22 LTS (conservative, satisfies Hexo 8) or 24 (matches this machine)?
- Registry policy for CI: regenerate lock on npmjs.org, or formally adopt npmmirror?
- Should Valine/LeanCloud comments be migrated before any theme-level front-end cleanup, and where is that app region hosted?
- Is the `1900/01/01/ge-ren-jian-li/` resume-as-post URL still wanted as a menu entry, or should it become a page?
