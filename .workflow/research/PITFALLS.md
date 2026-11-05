# Pitfalls Research

## Summary

The site is healthy today: a local Hexo 7.3 build reproduces the live gh-pages output exactly (263 `index.html` pages, 333 files), and the latest `master` commit was deployed 34 s later. The dominant risk is **URL fragility**: permalinks are pinyin-ized from the *source folder path* plus the front-matter date, and no redirect mechanism exists, so routine file/category renames silently break every inbound link. Secondary risks are an EOL Node 12 CI pipeline pinned to floating action refs, a vendored 2017-era theme with CDN-hosted old JS, and unmanaged content assets/taxonomy. None require a rewrite; most are fixable with a few config and CI changes.

## Findings

### 1. Permalink/URL stability — every post URL is derived from folder path + date + pinyin (HIGH)
- **Evidence:** `source/_posts/数据库/linux-access-mysql-remote.md` publishes to `/2016/03/14/shu-ju-ku/linux-access-mysql-remote/`; the folder segment is part of Hexo's `:title` slug, then `hexo-permalink-pinyin@1.1.0` transliterates it. `_config.yml` says `:year/:month/:day/:title/`, which hides this coupling. No alias/redirect plugin is installed.
- **Triggers:** renaming/moving a `.md` file or its category folder, changing `date:`, changing title text, or upgrading/disabling the pinyin plugin (`^1.1.0`, last release old). The theme menu hardcodes `/1900/01/01/ge-ren-jian-li/` (`themes/yelee/_config.yml`), so renaming `个人简历.md` breaks "关于我" with no build error.
- **Impact:** 88 posts, 193 sitemap URLs, and all external backlinks/SEO equity; a single category rename can 404 dozens of pages.
- **Likelihood:** Medium-High. **Early warning:** post-deploy crawl returning 404s; Search Console 404 spike; the "关于我" menu 404.
- **Mitigation:** freeze folder/file/date names; commit a generated URL manifest and diff it in CI; adopt `hexo-generator-alias` (or committed meta-refresh stubs) for any intentionally changed URL; pin `hexo-permalink-pinyin` and its `transliteration` dependency exactly; add a CI internal-link + URL-diff check.

### 2. Deploy pipeline — Node 12 EOL with Hexo 7 (`engines: >=14`), unpinned installs (HIGH over time)
- **Evidence:** `.github/workflows/deploy.yml` pins `node-version: "12.x"` while `hexo@7.3.0`, `hexo-renderer-marked@6.3.0`, `hexo-util@3.3.0` all declare `node >=14`. A test build with Node 12.22.12 still succeeds today, so this is debt, not a current outage. The workflow uses `npm install` (not `npm ci`), installs unpinned `hexo-cli -g`, and references `actions/checkout@master` / `actions/setup-node@master`. Deployment relies on the `HEXO_DEPLOY_PRIVATE_KEY` secret and only runs on `master`; no post-deploy health check exists.
- **Impact:** silent deploy stop (build fails, site freezes at last good gh-pages commit) or non-reproducible output from an unpinned dependency bump.
- **Likelihood:** Medium now, High within a year. **Early warning:** no new `Site updated:` commit on `gh-pages`; Actions run red; site content older than `master`.
- **Mitigation:** move to Node 20/22, use `npm ci`, pin actions by tag/SHA, cache npm, add `engine-strict=true`, and end the job with `curl -fsS https://www.crazy-coder.cn/` plus a check that the deployed page contains the current commit hash.

### 3. Custom domain / CNAME and gh-pages branch — one file deletion takes the domain down (MEDIUM likelihood, HIGH impact)
- **Evidence:** `source/CNAME` contains `www.crazy-coder.cn` and is copied to `gh-pages`; there is **no `.nojekyll`** in the deployed tree. The custom-domain binding in GitHub repo settings is not versioned. `url` and CNAME both use `www`, but the apex `crazy-coder.cn` is not covered and the theme emits **no canonical tag** (0 in `public/index.html`).
- **Failure mode:** deleting/renaming `source/CNAME`, a force-push that recreates gh-pages without it, or a repo-settings change drops the custom domain; GitHub then serves `sunhao-java.github.io`, and `www` breaks. Missing `.nojekyll` means Jekyll still runs on every deploy and can choke on underscore-prefixed paths.
- **Early warning:** `curl -I https://www.crazy-coder.cn` shows `github.io`/404; gh-pages tree lacks `CNAME`.
- **Mitigation:** keep `source/CNAME` tracked; add `source/.nojekyll`; add a post-deploy `curl` assertion on the custom domain; document DNS + Pages settings; add canonical/`og:url` (already present) and decide on apex→www redirect.

### 4. Vendored theme + old CDN front-end dependencies (MEDIUM)
- **Evidence:** `themes/yelee` is a vendored copy (no `.git`, version `Yelee: "3.5"`, 828 KB) with no upstream merge path. `themes/yelee/_config.yml` loads jQuery 2.2.4, fancybox 2.1.5, Valine 1.5.3, animate.css 3.5.1, etc. from `cdn.jsdelivr.net` via protocol-relative URLs with no SRI. Hexo 7 + `hexo-renderer-marked@6` + `hexo-renderer-stylus@3` are a different generation from the theme's original Hexo 3 era.
- **Impact:** Hexo/renderer upgrades can break EJS helpers or Markdown HTML assumptions with no upstream fixes; jsDelivr is slow/blocked intermittently in mainland China, taking search, TOC, comments and fancybox down with it; old jQuery/Valine widen the XSS surface. Valine `app_id`/`app_key` are public in config (normal, but abusable).
- **Likelihood:** Medium. **Early warning:** browser console 404s/timeouts, unstyled or non-interactive pages, `hexo generate` deprecation warnings.
- **Mitigation:** vendor the CDN files into `themes/yelee/source` (or a China-friendly CDN) and add SRI; freeze Hexo/renderers to exact versions; snapshot the theme and test upgrades in a branch with a visual diff; move comments to a maintained service if Valine abuse appears.

### 5. Content/search/sitemap drift — hotlinked images, tag case splits, noisy sitemap (HIGH likelihood, MEDIUM impact)
- **Evidence:** 31 images are referenced by external URL, with 27 references to `https://imgs.lodsve.com:9000/...` (self-hosted MinIO, non-standard port, no local copies). 91 unique tags for 88 posts with 8 case-collision groups (`mysql`/`MySQL`, `oracle`/`Oracle`, `redis`/`Redis`, `solr`/`Solr`, `nginx`/`Nginx`, `hbase`/`HBase`, `devops`/`DevOps`/`DevOPS`, `mongodb`/`MongoDB`/`Mongodb`) → fragmented tag pages and duplicate sitemap entries. `search.xml` is 676 KB of full post bodies (loaded on demand because `search.onload: false`, still heavy on mobile). All 193 sitemap `<lastmod>` values equal the build date because a fresh clone resets mtimes; 103 of 193 URLs are tag/category/archive pages. `source/robots.txt` advertises `http://` sitemap URLs. Good news: all 22 unique absolute internal links resolve, and no drafts exist (`render_drafts: false`, empty `_drafts`).
- **Impact:** images vanish if the MinIO host moves; duplicate tag pages dilute SEO; sitemap freshness is meaningless; broken links can appear unnoticed.
- **Early warning:** image 4xx/timeouts in browser or uptime checks; tag pages proliferating; sitemap `lastmod` identical across all URLs.
- **Mitigation:** download hotlinked images into `source/imgs` and rewrite references; normalize tags to a lowercase whitelist (or use `tag_map`); add a scheduled link/image checker; set `updated_option: date` or explicit `updated:`; trim `search.xml` fields or switch to a JSON index; fix `robots.txt` to `https://`.

## Recommendations

1. **Lock URLs now:** generate `urls.txt` from `public/**/index.html`, commit it, and fail CI on removals without an alias. Never rename category folders or post files without adding a redirect.
2. **Fix CI first (cheap, high value):** Node 22, `npm ci`, pinned actions, `engine-strict`, post-deploy `curl` + commit-hash assertion.
3. **Harden the domain:** add `source/.nojekyll`, keep `CNAME` tracked, add a deploy-time domain check, add canonical tags/apex policy.
4. **De-risk assets:** localize `imgs.lodsve.com` images and jsDelivr libraries (with SRI), then upgrade Hexo only in a branch with visual regression.
5. **Clean taxonomy:** collapse the 8 case-duplicate tag groups, add a link checker, and make sitemap `lastmod` reflect real edits.
6. **Prefer `npm ci` + exact versions** for `hexo-permalink-pinyin`/`transliteration`, since pinyin output changes silently alter every Chinese URL.

## Open Questions

- Is the custom domain also configured in GitHub Pages settings and DNS (apex vs `www`), and is there an apex redirect?
- Are the `imgs.lodsve.com:9000` images backed up, and is that host intended to be long-lived?
- Is Node 12 kept deliberately for an old local/CI environment, or is upgrading to Node 20/22 acceptable?
- Do external backlinks/Search Console history matter enough to justify permanent redirect stubs for old URLs?
- Is Valine still the intended comment system, and is the public app key acceptable to the owner?
- Which posts (if any) are intentionally private/unlisted? The resume page exposes phone/WeChat publicly at a permanent URL.
