# Project: 孙昊(Hulk)的博客

## What This Is

基于 Hexo 7 的个人技术博客，使用定制的 yelee 主题，以 Markdown 写作，通过 GitHub Actions 构建并部署到 `gh-pages`，绑定自有域名 [crazy-coder.cn](https://www.crazy-coder.cn/)。内容面向中文技术读者，现有 88 篇文章，覆盖后端、数据库、中间件、大数据、AI、运维、前端、工具与个人感悟等分类。

## Core Value

**从「想法」到「可被检索的线上文章」的路径必须足够低摩擦** —— 写作、构建、发布三步不能成为负担；同时所有历史内容必须长期可访问、可搜索、可归档。其他一切（主题花哨程度、功能数量）都让位于这一点。

## Requirements

### Validated

<!-- 已上线并由现有代码/提交历史确认有价值。 -->

- [x] Hexo 站点可在本地构建并预览（`hexo generate` / `hexo server`，`package.json` + `package-lock.json` 锁定依赖）
- [x] GitHub Actions 自动构建并部署到 GitHub Pages（`.github/workflows/deploy.yml`）
- [x] 通过 `hexo-deployer-git` 推送到 `gh-pages` 分支（`_config.yml` deploy 配置）
- [x] 绑定自定义域名 `www.crazy-coder.cn`（README 中的 CNAME 流程）
- [x] 定制 yelee 主题（`themes/yelee`：EJS 布局 + Stylus 样式）
- [x] 分类 / 标签 / 归档 / 首页分页（`hexo-generator-category|tag|archive|index`）
- [x] 站内搜索（`hexo-generator-search` 生成 `search.xml`）
- [x] RSS/Atom 订阅与 Sitemap、百度 Sitemap
- [x] 中文标题永久链接转拼音（`hexo-permalink-pinyin`）
- [x] Markdown 渲染与代码高亮（`hexo-renderer-marked` + highlight.js）
- [x] 88 篇已发布文章，按主题目录组织于 `source/_posts/`

### Active

<!-- 当前正在推进的假设，发布后才能转为 Validated。 -->

- [ ] 建立稳定的写作—发布节奏：选题 → 草稿 → 构建验证 → 发布 → 归档
- [ ] 将零散笔记整理为专题/系列，并为系列补索引页或导读
- [ ] 主题与依赖的可维护性：Hexo/主题升级前有构建与页面回归检查
- [ ] 持续优化可发现性：搜索、Sitemap、文章元数据（标题/描述/关键词）
- [ ] 把可复用的写作与排障经验沉淀到 `.workflow/specs/` 与领域词表

### Out of Scope

- 多用户账号、评论社区、运营活动 —— 定位是个人知识输出，不引入运营与审核负担
- 后端服务与数据库 —— 纯静态站点，无服务端运行时
- 迁移到其他框架（Next.js/Nuxt 等） —— 现有 Hexo 工作流稳定，迁移收益不抵成本
- 复杂的交互式前端应用功能 —— 与「低摩擦写作」的核心价值无直接关系

## Context

- 仓库同时是博客源码与部分个人履历类内容的载体，近年提交以文章与简历更新为主。
- 分支模型：`master` 保存源码，`gh-pages` 保存生成的静态页面。
- 仓库已有 `.codegraph/` 索引，可直接复用做代码/结构查询，无需重建。
- 依赖由 `package-lock.json` 锁定；Node.js 环境通过 GitHub Actions 与本地 nvm 提供。
- 文章语言以中文为主（`language: zh-CN`），涉及代码块与英文术语。

## Constraints

- **托管平台**: GitHub Pages 静态托管 —— 无服务端逻辑，构建产物必须完全静态
- **构建工具**: Node.js + Hexo 7.x —— 升级需同步验证主题与插件兼容性
- **部署链路**: push 触发 GitHub Actions → `gh-pages` —— 构建失败即无法发布
- **内容语言**: 中文为主，永久链接转拼音 —— 改 permalink 规则会影响既有 URL
- **仓库范围**: `.workflow/` 之外的源码与文章仅在明确任务中修改

## Tech Stack

- **Language**: JavaScript（Node.js），内容为 Markdown
- **Framework**: Hexo 7.3 + yelee 主题（EJS / Stylus）
- **Database**: 无（静态站点，搜索索引为生成的 `search.xml`）

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| 继续使用 Hexo + yelee 主题 | 现有内容与工作流已稳定，迁移成本高于收益 | — Pending |
| 源码与生成页面分分支（master / gh-pages） | 保持仓库历史干净，部署产物不污染源码 | — Pending |
| 永久链接启用拼音转换 | 中文标题生成可读、可分享的 URL | — Pending |
| 工作流主线聚焦内容创作与知识沉淀 | 站点能力已基本满足写作需求，边际收益在内容而非功能 | — Pending |

## Stakeholders

- 孙昊(Hulk) —— 作者与唯一维护者
- 中文技术读者 —— 通过搜索与订阅访问文章

---
*Last updated: 2026-11-05 after initialization*
