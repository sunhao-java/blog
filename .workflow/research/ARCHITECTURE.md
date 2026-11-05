# Architecture Research

## Summary

The blog is a conventional Hexo 7.3 static site: 88 Markdown files, one directory per topic, a single global image pool, and URLs derived from date + directory + filename. Content organization is *consistent by accident* — categories are duplicated from directory names and happen to match 87/87 non-resume posts, but nothing enforces it. The weak points are (a) the directory is embedded in the URL through the post slug, (b) the taxonomy is unmanaged (91 tags, 59 used once), (c) there is no series/专题 model and no category index page, and (d) recent posts depend on an unreachable external image host. The publish path (push → Actions on Node 12 → `hexo deploy`) works today but is one runner deprecation away from breaking.

## Findings

### 1. Content organization: directory ≈ category, tags unmanaged

- 88 posts across 10 dirs: 中间件 20, 后端 17, 运维 15, 数据库 12, 工具 10, 大数据 5, 其他 3, AI 2, 个人感悟 2, 前端 1; plus `source/_posts/个人简历.md` at the root.
- Front-matter is minimal and uniform: only `title`, `date`, `tags`, `categories` (verified across all 88); no `description`, `keywords`, `updated`, or `series`. One post adds `password`.
- Front-matter delimiter style is split: 5 posts use `---`…`---`; 83 rely on Hexo's lenient form (no opening `---`, closing `---` only). Hexo 7's `hexo-front-matter` `rFrontMatterNew` parses both, so the build is fine — but a missing *closing* `---` would silently turn metadata into body text.
- `categories` equals the directory name in 87/87 posts that declare one (only the resume has none). This is a hand-maintained invariant, not a Hexo rule.
- Tagging is the main inconsistency: 91 distinct tags, 59 of them singletons (`系统`, `windows7`, `共享网络`, `yapi`…); case/word variants split pages (`Solr`/`solr`, `MySQL`/`mysql`, `MongoDB`/`Mongodb`, `Nginx`/`nginx`, `DevOps`/`DevOPS`, `HBase`/`hbase`); 20 posts omit the space after commas (`[Linux,Ubuntu,Redis]`). Language also mixes (`database` vs 数据库).
- Category boundaries are fuzzy: `中间件` absorbs OS-level installs (`linux_ubuntu_install_nginx`, `IDE安装wakatime插件`), `运维` absorbs Linux basics, `大数据` absorbs OpenResty/Flume/Kafka glue. Two near-duplicate posts exist: `中间件/Linux_Ubuntu安装Redis.md` vs `中间件/linux_ubuntu_install_redis.md` (121 vs 122 lines), and `mongodb_install` vs `linux_ubuntu_install_mongodb`.

### 2. URL / permalink architecture

- `_config.yml`: `permalink: :year/:month/:day/:title/` + `hexo-permalink-pinyin` (`separator: '-'`). Pages live in `source/tags/`, `source/categories/`, `source/404/`; per-page 12.
- A local build (`npx hexo generate`, Node 24) produced 333 files / 87 dated post pages. Actual output: `public/2016/03/18/shu-ju-ku/mysql-command/index.html`, i.e. **the directory name is part of the URL** — `:title` resolves to the post slug, and Hexo 7 derives the slug from the path under `_posts` (`数据库/mysql_command`), which the pinyin plugin then transliterates to `shu-ju-ku/mysql-command`.
- Consequence: three independent inputs move a URL — the front-matter `date`, the containing directory, and the filename. Nothing records or redirects old URLs (no redirect plugin in `package.json`). A pure re-organization of folders would break every inbound link and the Google index.
- Deployed `gh-pages` (commit `d966374`, 2026-09-28) matches a fresh local build, so there is no current drift. The URL shape is therefore de-facto frozen: `/YYYY/MM/DD/<pinyin-dir>/<pinyin-filename>/`.
- `source/categories/index.md` sets `type/layout: categories`, but yelee only ships `category.ejs` (singular). The rendered `/categories/` page is an empty article titled 归档 (verified in `public/categories/index.html`); category browsing only works at `/categories/<name>/`. The theme menu exposes 主页/所有文章/标签云/关于我 — no category entry. `/tags/` and `/tags/AI/` work.
- The menu hardcodes 关于我 → `/1900/01/01/ge-ren-jian-li/`, the pinyin of the resume post. That URL is stable only because `date: 1900-01-01` is pinned.
- SEO surface is thin: no `description` per post, no canonical/OG tags in the theme; `search.xml` is 676 KB (full post content) and `atom.xml` 31 KB.

### 3. Series / 专题 support

- No series mechanism exists: no `series` front-matter, no series index page, no "part N" navigation. `themes/yelee` layouts are index/post/page/archive/tag/category only; prev/next is chronological.
- Only one post claims a series in its title (`后端/跟我一起系列之读Spring注解源码.md`) and it has no siblings.
- The closest thing to a series today is the tag `xxl-job` (4 posts, 2020) and `spring`/`spring-cloud` (1 post each); a reader has no way to discover order or completeness.
- Hexo primitives available without new plugins: nested categories (`categories: [后端, Spring]`), `{% post_link <slug> %}`, and plain pages under `source/`. A `series` key plus a hand-curated `source/series/<slug>.md` index is the lowest-friction option because it needs zero theme changes.

### 4. Image / asset strategy

- `post_asset_folder: false`; assets are global under `source/imgs/<topic>/` (24 files: `hexo/`, `mysql/`, `xxl-job/`, `big_data/`, `history/`, `build_blog_by_travis/`, `ai/`, plus `coder.jpg`, `logo.png`). Posts reference them absolutely (`![](/imgs/hexo/1.png)`, 22 refs), so renaming a post never breaks its images — but nothing associates an image with its post and nothing prunes orphans.
- 27 image refs point at `https://imgs.lodsve.com:9000/...` (4 recent posts: RAG, mkcert, wakatime, 转载). The host is unreachable from the public internet right now (curl exit 56/timeout, even for `/`), so those posts likely render with broken images.
- 4 legacy refs point at `img.blog.csdn.net` / `img.my.csdn.net` (301/302 → anti-hotlink risk).
- The theme itself reads `source/imgs/coder.jpg` for avatar/favicon and `source/background/` for backgrounds, coupling theme config to the global image pool.

### 5. Write → publish pipeline

- `npm run build|clean|deploy|server` wrap the Hexo CLI; scaffold (`scaffolds/post.md`) emits `title/date/tags` only — no `categories`, and `hexo new` writes to `source/_posts/<title>.md` (root), so the author must move the file into a topic dir by hand or the post gets no category and a root-level URL.
- `.github/workflows/deploy.yml` runs on push to `master` with `actions/checkout@master`, `actions/setup-node` pinned to **Node 12** (EOL; Hexo 7 requires Node ≥ 14), then `hexo clean && hexo generate && hexo deploy` using an SSH deploy key. It works only because the runner tolerates the old Node; a runner image update breaks publishing.
- No CI checks on content: no front-matter validation, duplicate-slug check, broken-link/image check, or HTML build assertion beyond `hexo generate` exit code.
- Local build succeeds in ~2.3 s for 333 files, so adding validation is cheap.

## Recommendations

### Target content architecture (keep directories, make them authoritative)

1. **Freeze the URL contract.** Keep `permalink: :year/:month/:day/:title/` and never rename `date`, folder, or filename of a published post. If a folder must change, add `hexo-generator-redirect`-style aliases (or a static `_redirects`/meta-refresh page per old path) in the same commit. Document this in `README.md` as a "published post is immutable" rule.
2. **One directory = one category, enforced.** Keep the 10 top-level dirs as the canonical taxonomy, but tighten semantics: `中间件` = runtime/middleware (Redis, MQ, Nginx, Solr, FastDFS), `运维` = OS/ops/deploy, `工具` = developer tooling, `大数据` = data pipeline. Move `IDE安装wakatime插件` → 工具, `nginx_url` → 前端 or 中间件, Linux install posts stay in their product category. Allow nested categories only for real series (`categories: [后端, Spring]`).
3. **Controlled tag vocabulary.** Cap tags at ~40 evergreen terms, English product names in canonical casing (`MySQL`, `Redis`, `Kubernetes`, `Spring`), Chinese only for cross-cutting concepts (`源码`, `性能`, `排障`, `转载`). Ban one-off tags; merge the existing case duplicates. This turns `/tags/` from 91 near-empty pages into a usable index.
4. **Add `description` (and optionally `updated`) to front-matter** and surface it in `head.ejs` for SEO/social previews. Keep `search.xml` but consider `field: post` + `content: false` style trimming if size grows.
5. **Fix category browsing.** Either add `themes/yelee/layout/categories.ejs` (mirror the tag-cloud partial with `list_categories`) and link 分类 in the menu, or repurpose `source/categories/index.md` as a hand-curated 专题/分类导航 page. Also fix its title (currently 归档) and the duplicated 分类 & 标签 title on `/tags/`.
6. **Series model (low friction).** Use `series: xxl-job` + `series_order: 2` in front-matter, and one plain page per series at `source/series/<slug>.md` listing parts with `{% post_link ... %}`. Add a `series` block to `scaffolds/post.md` so starting a series is one edit. This needs no theme changes and keeps the machine-readable key for a future generator. Retrofit: XXL-JOB (4 posts), Spring 注解源码, Kubernetes/Harbor, Redis/MongoDB 高可用 clusters.
7. **Images: keep the global pool, add a convention.** Stay on `post_asset_folder: false` (avoids coupling assets to the URL-bearing folder and matches existing content), but standardize `source/imgs/<post-slug>/<n>.png` and always use root-absolute `/imgs/...`. Migrate the 27 `imgs.lodsve.com` refs and 4 CSDN refs into local dirs in the same pass. Add a CI script that fails on external image hosts and missing local files.
8. **Pipeline hardening.** Update Actions to `actions/checkout@v4` + `actions/setup-node@v4` with Node 20 and npm cache; add `npm run new -- <category> <slug>` wrapping `hexo new post --path <category>/<slug>.md` so the file lands in the right folder with `categories` pre-filled; add `npm run check` running a front-matter linter (required keys, tag whitelist, duplicate slug/URL, dead image refs) and wire it before `hexo generate` in CI. Optionally keep `hexo-deployer-git`; a `peaceiris/actions-gh-pages` step is the lower-maintenance alternative.
9. **De-duplicate** the two Redis-install and two MongoDB-install posts (canonicalize one, redirect or unpublish the other) before any taxonomy migration.

## Open Questions

- Is URL stability or folder tidiness more important? A one-time reorganization is feasible now (traffic is low) but requires a redirect map; after that, treat published paths as immutable.
- Should `中间件` be split (e.g. `中间件` vs `基础设施`), or is the current 10-category set the intended long-term taxonomy?
- Is `imgs.lodsve.com` permanently retired (then migrate all 27 refs) or just temporarily down (then keep and monitor)?
- Do series need in-article prev/next navigation (requires a small theme/tag plugin), or is a curated index page enough?
- Should the resume stay a dated post at `1900-01-01`, or become a real `page` (`/about/`) with a redirect from the old URL?
- Keep the date in the permalink (`/2026/04/17/...`) for chronological context, or move to a flatter `/:category/:title/` (all URLs change, so only worth doing with redirects)?
