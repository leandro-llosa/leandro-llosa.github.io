# NotionGit — Personal Site & Blog

A free, minimalist personal website template powered by **Notion** and **GitHub Pages**.

Write everything in Notion — your home page, about, now page, blog posts, any section you want — and it appears on your site within ~10 minutes. No servers, no paid tools, no code required after setup.

---

## How it works

```
Notion databases  →  GitHub Action (cron)  →  Jekyll files  →  GitHub Pages
```

Two Notion databases drive the entire site:

| Database | Controls |
|---|---|
| **Pages** | Your home page, nav structure, and every site section (About, Now, Contact, etc.) |
| **Posts** | Your blog posts |

A scheduled GitHub Action queries both databases every 10 minutes, converts content to Markdown, and commits the files. GitHub Pages rebuilds automatically.

---

## Setup (≈ 20 minutes)

### Step 1 — Create two Notion databases

Duplicate the Notion workspace template *(link)*, which gives you both databases pre-configured. Or create them manually with the schemas below.

**Pages database** — one row per section of your site:

| Property | Type | Notes |
|---|---|---|
| `Title` | Title | Page name (also shown in nav) |
| `Slug` | Text | URL path, e.g. `about`, `now`, `blog` |
| `Type` | Select | `home`, `blog-list`, or `markdown` |
| `Nav Order` | Number | Sort position in the navbar (1 = first) |
| `Show in Nav` | Checkbox | Whether to appear in the header nav |
| `Status` | Select | `Draft` or `Published` |
| `Description` | Text | Optional meta description |
| `Name` | Text | Your display name — shown on the home page, and used for the browser tab title and RSS feed (home type only) |
| `Profile Picture` | Text | External image URL — home page only |
| `Tagline` | Text | One-line bio — home page only |
| `Social Links` | Text | One `Name: URL` per line — home page only |

**Posts database** — one row per blog post:

| Property | Type | Notes |
|---|---|---|
| `Title` | Title | Post title |
| `Slug` | Text | URL identifier, e.g. `my-first-post` |
| `Status` | Select | `Draft` or `Published` |
| `Publish Date` | Date | Date shown on the post |
| `Tags` | Multi-select | Optional |
| `Cover Image` | Files & media | Use an **external URL** (Notion-hosted URLs expire) |
| `Description` | Text | Optional excerpt / meta description |

### Step 2 — Create a Notion integration

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations).
2. Click **+ New integration**, give it a name (e.g. *Site Sync*), select your workspace.
3. Copy the **Internal Integration Token** — you'll need it in Step 4.
4. Back in Notion, open **each database**, click `···` → **Add connections**, and connect your integration.

### Step 3 — Use this GitHub template

1. Click **"Use this template"** at the top of this repository.
2. Name your repo `username.github.io` (for a root site) or anything else.
3. Clone or open the new repo.

### Step 4 — Add secrets to GitHub

Go to **Settings → Secrets and variables → Actions → New repository secret** and add:

| Secret name | Value |
|---|---|
| `NOTION_TOKEN` | Your integration token from Step 2 |
| `NOTION_PAGES_DATABASE_ID` | ID of your Pages database |
| `NOTION_POSTS_DATABASE_ID` | ID of your Posts database |

**Finding a database ID:** Open the database in Notion and look at the URL:
```
https://www.notion.so/yourworkspace/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx?v=...
                                   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                                   This 32-character string is the database ID
```

### Step 5 — Configure the site

Edit `_config.yml` and update at minimum:

```yaml
url: "https://username.github.io"
author:
  email: "you@example.com"
```

`title:` and `author.name` are set for you: every sync points them at the `Name`
property of the home page in your Pages database, so the browser tab title and the
RSS feed follow whatever you call yourself in Notion. Change your name there, not here.

### Step 6 — Enable GitHub Pages

1. Go to **Settings → Pages**.
2. Under **Source**, select **Deploy from a branch**.
3. Select branch `master` (or `main`) and folder `/ (root)`.
4. Click **Save**.

### Step 7 — Set up your home page in Notion

In your Pages database, create a row with:
- `Type` = `home`
- `Status` = `Published`
- `Profile Picture` = a URL to your photo (use an external host, not Notion upload)
- `Tagline` = a one-line description of yourself
- `Social Links` = one link per line in `Name: URL` format:
  ```
  GitHub: https://github.com/username
  Twitter: https://twitter.com/handle
  Email: mailto:you@example.com
  ```
- Write your longer bio/intro in the **page body** — it's rendered as Markdown below your tagline.

### Step 8 — Add site sections

Create rows in the Pages database for each section you want. Common setup:

| Title | Slug | Type | Nav Order | Show in Nav |
|---|---|---|---|---|
| About | `about` | `markdown` | 1 | ✓ |
| Blog | `blog` | `blog-list` | 2 | ✓ |
| Now | `now` | `markdown` | 3 | ✓ |
| Contact | `contact` | `markdown` | 4 | ✓ |

Write the content of each page in the Notion page body.

### Step 9 — Publish your first post

1. In your Posts database, create a new row.
2. Fill in `Title`, `Slug`, and `Publish Date`.
3. Write the post content in the page body.
4. Set `Status` to **Published**.
5. Wait up to 10 minutes, or trigger manually: **Actions → Sync Notion → Run workflow**.

---

## Page types

| Type value | What it renders |
|---|---|
| `home` | Profile picture, name, tagline, social links, bio, recent posts. Always at `/`. |
| `blog-list` | Chronological list of all blog posts. |
| `markdown` | Clean content page — headings, text, code, images. Use for About, Now, Contact, etc. |

More types (e.g. `photos`) can be added by creating a matching `_layouts/{type}.html` and handling the value in `typeToLayout()` in `scripts/sync-notion.js`.

---

## Supported Notion content

| Notion block | Markdown output |
|---|---|
| Paragraph | Plain paragraph |
| Heading 1 / 2 / 3 | `#` / `##` / `###` |
| Bulleted list | `- item` |
| Numbered list | `1. item` |
| To-do list | `- [ ] item` / `- [x] item` |
| Code block | ` ```lang ``` ` with syntax highlighting |
| Quote | `> blockquote` |
| Callout | `> emoji callout text` |
| Divider | `---` |
| Image | `![caption](url)` |
| Video | Link |
| Bookmark / link preview | Inline link |
| Toggle | `<details><summary>...</summary>` |

---

## Customisation

### Site title, description, author
Edit `_config.yml`.

### Home page content
Edit the `home` row in your Notion Pages database. No code changes needed.

### Navigation
Add or reorder pages in the Notion Pages database. Anything with **Show in Nav** checked and a **Nav Order** appears in the header automatically.

### Theme colours and fonts
All design tokens are CSS custom properties in the `:root` block at the top of `assets/css/main.css`. Change colours, fonts, and spacing there. Dark mode is handled automatically via `@media (prefers-color-scheme: dark)`.

### Custom domain
1. Add a `CNAME` file to the repo root with your domain (e.g. `yourname.com`).
2. Configure your DNS to point to GitHub Pages.
3. Update `url:` in `_config.yml`.

---

## Unpublishing content

Set `Status` to `Draft` in Notion. On the next sync:
- A **post** is removed from `_posts/` and disappears from the blog.
- A **page** is removed from `_pages/` and its URL returns 404.

---

## Local development

```bash
# Install Ruby dependencies
bundle install

# Serve locally with hot-reload
bundle exec jekyll serve

# Run the sync script locally
NOTION_TOKEN=secret_xxx \
NOTION_PAGES_DATABASE_ID=xxx \
NOTION_POSTS_DATABASE_ID=xxx \
node scripts/sync-notion.js
```

---

## Repository structure

```
├── _posts/                   ← generated by sync (do not edit manually)
├── _pages/                   ← generated by sync (do not edit manually)
├── _data/
│   ├── home.yml              ← generated: home page profile data
│   └── nav.yml               ← generated: navigation items
├── _layouts/
│   ├── default.html          ← base HTML shell
│   ├── home.html             ← profile + bio + recent posts
│   ├── blog.html             ← chronological post list
│   ├── post.html             ← individual blog post
│   └── page.html             ← generic markdown content page
├── _includes/
│   ├── head.html             ← <head> meta + CSS
│   ├── header.html           ← site nav (built from _data/nav.yml)
│   ├── footer.html
│   └── post-card.html        ← post preview card
├── assets/css/main.css       ← full theme, plain CSS, dark-mode ready
├── scripts/
│   └── sync-notion.js        ← Notion → Markdown conversion + file writing
├── .github/workflows/
│   └── sync-notion.yml       ← scheduled GitHub Action (every 10 min)
├── _config.yml
└── index.html                ← home page entry point (layout: home)
```

---

## FAQ

**Q: How often does the sync run?**
Every 10 minutes via GitHub Actions cron. Trigger it manually anytime from the Actions tab. GitHub may delay scheduled runs a few minutes under high load.

**Q: Do I need both databases?**
No. Set only `NOTION_POSTS_DATABASE_ID` to run a posts-only blog (no home/pages sync). Set only `NOTION_PAGES_DATABASE_ID` for a site with no blog. Set both for the full experience.

**Q: What if one page/post fails to sync?**
The script logs detailed output and continues processing remaining items. One failure won't block others. Check the Actions run log for details.

**Q: Are Notion image URLs permanent?**
No — Notion-hosted file URLs expire after ~1 hour. Always use **external** image URLs (your own CDN, Unsplash, etc.) for `Profile Picture` and `Cover Image`.

**Q: How do I add a new custom section type (e.g. Photos)?**
1. Add a new option to the `Type` select in your Notion Pages database (e.g. `photos`).
2. Create `_layouts/photos.html`.
3. Add a case to `typeToLayout()` in `scripts/sync-notion.js`.
4. Style it in `assets/css/main.css`.

**Q: Will GitHub Pages build correctly?**
Yes. The theme uses only GitHub Pages–safe plugins (`jekyll-feed`, `jekyll-seo-tag`, `jekyll-sitemap`) and standard Jekyll features — no custom plugins.

---

## License

MIT — free to use, modify, and deploy.
