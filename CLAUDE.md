# CLAUDE.md — Project Context

This file provides context for AI assistants (Claude) working on this codebase.

---

## What this project is

A Jekyll-based personal website + blog template where **all content is managed through Notion**. Non-technical users fork the repo, add two secrets, and their entire site — home page, navigation, sections, and blog — is driven by Notion databases synced via a GitHub Action.

The design philosophy is BearBlog-style minimalism: fast, readable, no JavaScript, dark-mode ready, fully themeable via CSS custom properties.

---

## Architecture

### Data flow

```
Notion Pages DB  ─┐
                   ├─→  scripts/sync-notion.js  ─→  _data/  +  _pages/  ─→  Jekyll  ─→  GitHub Pages
Notion Posts DB  ─┘                              ─→  _posts/
```

The sync script runs as a scheduled GitHub Action every 10 minutes and commits any changed files back to the repo.

### The two Notion databases

**1. Pages database** (`NOTION_PAGES_DATABASE_ID`)
Controls the site structure. One row = one section/page of the site.

Properties read by the sync script:
- `Title` (Title) — page name used as the nav label and page heading
- `Slug` (Text) — URL path, e.g. `about`, `blog`, `now`
- `Type` (Select) — determines which Jekyll layout to use (see Page types below)
- `Nav Order` (Number) — sort order in the header navbar
- `Show in Nav` (Checkbox) — whether to include in the navbar
- `Status` (Select) — must be `Published` to sync
- `Description` (Text) — optional meta description
- `Name` (Text) — display name shown on the home page (home type only); falls back to `Title` if not set.
  Also written into `_config.yml` as `title` and `author.name`, which is what jekyll-seo-tag
  and jekyll-feed use for the browser tab title, the feed title and the feed author
- `Profile Picture` (Text) — external image URL, used by the `home` type
- `Tagline` (Text) — one-liner bio, used by the `home` type
- `Social Links` (Text) — newline-separated `Name: URL` pairs, used by the `home` type

The **Notion page body** is converted to Markdown and used as the page content.

**2. Posts database** (`NOTION_POSTS_DATABASE_ID`)
Blog posts. Falls back to `NOTION_DATABASE_ID` for backwards compatibility.

Properties: `Title`, `Slug`, `Status`, `Publish Date`, `Tags`, `Cover Image`, `Description`, `Canonical URL`, `Featured`.

### Page types → Jekyll layouts

| Notion `Type` value | Jekyll layout | URL |
|---|---|---|
| `home` | n/a (writes `_data/home.yml`) | `/` via `index.html` |
| `blog-list` or `blog` | `blog` | `/{slug}/` |
| `markdown` (default) | `page` | `/{slug}/` |

Adding a new type: add a case in `typeToLayout()` in `scripts/sync-notion.js` and create the corresponding `_layouts/{type}.html`.

### Generated files

| File | Generated from |
|---|---|
| `_data/home.yml` | The `home` type page in Pages database |
| `_config.yml` (`title`, `author.name` only) | The `Name` property of the `home` type page |
| `_data/nav.yml` | All Pages with `Show in Nav: true`, sorted by `Nav Order` |
| `_pages/{slug}.md` | Each non-home published page in Pages database |
| `_posts/{date}-{slug}.md` | Each published post in Posts database |

---

## File structure

```
├── _data/
│   ├── home.yml          ← profile pic, name, tagline, social links, bio markdown
│   └── nav.yml           ← [{title, url}] sorted by Nav Order
├── _layouts/
│   ├── default.html      ← base HTML shell (header + main + footer)
│   ├── home.html         ← reads _data/home.yml; shows profile + bio + recent posts
│   ├── blog.html         ← chronological post list using post-card.html
│   ├── post.html         ← individual blog post with cover image + tags
│   └── page.html         ← generic markdown content page
├── _includes/
│   ├── head.html         ← <head>: charset, viewport, {% seo %}, CSS link, feed meta
│   ├── header.html       ← sticky nav built dynamically from _data/nav.yml
│   ├── footer.html       ← year + site name + "Built with Jekyll / Notion"
│   └── post-card.html    ← reusable post preview (date + tags + title + excerpt)
├── _pages/               ← managed by sync, do not edit manually
├── _posts/               ← managed by sync, do not edit manually
├── assets/css/main.css   ← complete theme: design tokens, reset, all components
├── scripts/
│   └── sync-notion.js    ← Node.js sync script (CommonJS, @notionhq/client only)
├── .github/workflows/
│   └── sync-notion.yml   ← cron every 10 min; stages _posts/ _pages/ _data/
├── _config.yml           ← Jekyll config; includes _pages collection definition
├── Gemfile               ← github-pages gem + jekyll-feed/seo-tag/sitemap
├── package.json          ← {dependencies: {"@notionhq/client": "^2.2.15"}}
└── index.html            ← layout: home (entry point for home page)
```

---

## CSS design system

All tokens are CSS custom properties in `:root` in `assets/css/main.css`. Key ones:

```css
--bg, --surface         /* page and card backgrounds */
--text, --text-secondary, --text-muted
--border
--accent, --accent-hover  /* links, interactive states */
--font-sans, --font-mono
--container: 720px      /* max content width */
--radius, --radius-lg
--s-1 through --s-16    /* spacing scale (0.25rem increments) */
```

Dark mode is handled automatically via `@media (prefers-color-scheme: dark)` — no JS toggle needed.

### Component classes

Home page: `.home-hero`, `.home-avatar`, `.home-avatar-placeholder`, `.home-name`, `.home-tagline`, `.home-social`, `.home-bio`, `.home-posts`, `.home-post-row`

Blog list: `.blog-header`, `.posts-list`, `.post-card`, `.post-card-title`, `.post-excerpt`, `.post-meta`, `.post-date`, `.tag`

Post: `.post-header`, `.post-title`, `.post-info`, `.post-cover`, `.post-content`, `.post-footer`, `.back-link`

Page: `.page-header`, `.page-title`, `.page-content`

Layout: `.site-header`, `.header-inner`, `.site-logo`, `.site-nav`, `.main`, `.container`, `.site-footer`

---

## sync-notion.js internals

**Entry point:** `main()` calls `syncPages()` then `syncPosts()` depending on which env vars are set.

**Shared utilities:**
- `richTextToMarkdown(richText[])` — handles bold, italic, code, strikethrough, links
- `blockToMarkdown(block)` — converts one Notion block to a Markdown string
- `blocksToMarkdown(blocks[])` — converts a block array, handles list grouping
- `fetchAllBlocks(blockId)` — paginated fetch of all children
- `fetchPublished(databaseId)` — queries DB filtering `Status == Published`
- `titleToSlug(title)` — URL-safe slugification
- `yamlStr(value)` — JSON.stringify wrapper for safe inline YAML strings

**Pages sync (`syncPages()`):**
1. Reads existing `_pages/*.md`, builds `notionIdToFile` map from `notion_id:` front matter
2. Fetches all Published pages from `NOTION_PAGES_DATABASE_ID`
3. For each page: extracts metadata, fetches blocks, converts to Markdown
4. If `type === 'home'`: accumulates profile data, writes `_data/home.yml`, and calls
   `syncConfigIdentity()` to point `_config.yml`'s `title` / `author.name` at the `Name`.
   That file is edited line by line, not YAML round-tripped, so its comments and ordering
   survive; unrecognised shapes are left alone and warned about
5. Otherwise: writes `_pages/{slug}.md` with appropriate layout front matter
6. Removes `_pages/*.md` files whose Notion pages are no longer Published — via
   `applyDeletions()`, which aborts the whole sync (exit 1, nothing committed) if the
   deletion looks like a bad read rather than a real edit
7. Writes `_data/nav.yml` from pages with `showInNav: true`, sorted by `navOrder`

**Posts sync (`syncPosts()`):** Same pattern as pages, but writes to `_posts/{date}-{slug}.md` with post-specific front matter.

**Social links format** (the `Social Links` Notion property):
```
GitHub: https://github.com/username
Twitter: https://twitter.com/handle
Email: mailto:user@example.com
```
Parsed by `parseSocialLinks()` into `[{name, url}]` stored in `_data/home.yml`.

---

## Jekyll config highlights

```yaml
collections:
  pages:
    output: true
    permalink: /:slug/

defaults:
  - scope:
      path: "_pages"
      type: "pages"
    values:
      layout: "page"
```

The `_pages` collection outputs each file at `/{slug}/`. The default layout is `page` but each file's own front matter overrides it (e.g. `layout: blog` for the blog list page).

---

## Adding a new custom section type

Example: adding a `photos` gallery type.

1. In Notion, add `photos` as a new option to the `Type` select property.
2. In `scripts/sync-notion.js`, add to `typeToLayout()`:
   ```js
   case 'photos': return 'photos';
   ```
3. Create `_layouts/photos.html` — it receives `{{ content }}` (the Notion page body as Markdown) and can use any custom HTML structure.
4. Add CSS for the new layout in `assets/css/main.css`.
5. If the layout needs special front matter (e.g. a list of image URLs), add properties to the Pages database and extract them in `extractPageMeta()`, then write them to the front matter in `buildPageFrontMatter()`.

---

## Local development

```bash
# Install Ruby gems
bundle install

# Serve locally (hot-reload on file changes)
bundle exec jekyll serve

# Run sync against real Notion data
NOTION_TOKEN=secret_xxx \
NOTION_PAGES_DATABASE_ID=xxx \
NOTION_POSTS_DATABASE_ID=xxx \
node scripts/sync-notion.js
```

Jekyll serves at `http://localhost:4000` by default.

---

## GitHub Actions workflow

`.github/workflows/sync-notion.yml`:
- Trigger: cron `*/10 * * * *` + manual `workflow_dispatch`
- Steps: checkout → Node 20 → `npm install` → run sync → `git add _posts/ _pages/ _data/ _config.yml` → commit + push if changed
- Required secrets: `NOTION_TOKEN` + (`NOTION_PAGES_DATABASE_ID` and/or `NOTION_POSTS_DATABASE_ID`)
- Legacy: `NOTION_DATABASE_ID` still works as a posts-only fallback

---

## Important constraints

- **No JS in the browser** — the site is pure HTML + CSS. Keep it that way.
- **GitHub Pages safe mode** — only use plugins listed in `github-pages` gem. No custom Ruby plugins.
- **Notion image URLs expire** — always use external URLs for profile pictures and cover images. Warn users in docs.
- **`_pages/` and `_posts/` are managed by sync** — never edit these manually; changes will be overwritten.
- **`_data/home.yml` and `_data/nav.yml` are managed by sync** — same rule.
- **`_config.yml` is hand-maintained except `title` and `author.name`** — those two track the
  home page's `Name` on every sync. Everything else in the file is yours; edit it freely.
- **The sync refuses suspicious bulk deletes** — if a run would delete more than
  `MAX_DELETE_RATIO` (default 0.5) of the tracked files in `_pages/` or `_posts/`, or if
  Notion returns zero published rows, the sync aborts with exit 1 and commits nothing.
  A dropped `Status` option or a revoked integration share is indistinguishable from a
  real unpublish, and this sync pushes straight to master. To push a genuine mass
  unpublish through, re-run with `ALLOW_BULK_DELETE=true`.
