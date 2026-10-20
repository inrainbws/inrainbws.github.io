# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## What This Is

A personal academic website (Jekyll static site) for Zhuoqian Yang, a PhD candidate at EPFL. Deployed to GitHub Pages at https://inrainbws.github.io.

## Commands

```bash
# Serve locally with live reload
bundle exec jekyll serve

# Build static site into _site/
bundle exec jekyll build
```

## Visual System

The site uses an **editorial-academic** style: warm-grey paper background, Newsreader serif titles, Instrument Serif italic *only* for year numerals, Geist sans body, IBM Plex Mono caps. Color is intentionally quiet — deep purple `#5C2483` for links, chartreuse `#C8D300` reserved as a small punch mark (the author's own name in author lists, and `Oral`/`Spotlight`/`Best Paper` flags).

The canonical source of truth lives **outside this repo**, in the sibling directory `~/me/style-package/`:

- `STYLE_GUIDE.md` — tokens, type scale, color rules, component recipes, do/don't list.
- `APPLY.md` — the step-by-step migration recipe that was applied to this repo.
- `css/site.css` — the canonical stylesheet (already copied into `assets/css/site.css`).
- `templates/` — reference `head.html`, `default.html`, `index.html`, and `example-post.md`.

**Before changing visual styles, read `~/me/style-package/STYLE_GUIDE.md` first.** Hard rules to follow:

- Never invent new colors. Use the tokens declared on `:root` in `assets/css/site.css` (`--paper`, `--paper-bg`, `--card`, `--ink`, `--ink-70/55/40/25`, `--rule`, `--rule-hi`, `--purple`, `--chart`, `--chart-soft`).
- Instrument Serif (`--serif-cond`) is used **only** for `.year-band__year`. Newsreader (`--serif`) is the title face for everything else.
- No gradients, drop shadows, glassmorphism, or decorative SVGs. The system is flat and ruled.
- No emoji or icon-font glyphs.
- Two columns only — sticky sidebar left, content right. No third column.
- Sections are exactly: Publications, Talks, Academic Services, Misc. Don't add "News", "Highlights", "Research interests", etc.

If `STYLE_GUIDE.md` and `site.css` conflict, the CSS wins — fix the guide.

## Architecture

### Site Structure

- **`index.html`** — Main page. Renders four `<section class="section">` blocks (`§ 01` Publications, `§ 02` Talks, `§ 03` Academic Services, `§ 04` Misc). Publications use Liquid's `group_by_exp` to bucket posts by year into `.year-band` blocks; **you no longer maintain numbered `site.posts[N]` references**, so adding or reordering posts just works.

- **`_posts/`** — Each file is one publication card (not a blog post). Filename format: `YYYY-M-D-slug.md`. Front matter is `layout: post` + `title:`. The body is an HTML fragment using the new `.pub__*` markup (see "Adding a Publication" below).

- **`_layouts/default.html`** — `.site` CSS grid: a sticky `.sidebar` (left) and `.main` (right). The sidebar contains the profile photo, role cap, `<h1 class="intro__name">` (with `<em>Zack</em>` middle line), bio paragraph with supervisor links, a `Previously` block (M.S. / Shanghai AI Lab / Beihang), a `Reach` block with email + 2×2 tile grid of CV / GitHub / Scholar / LinkedIn, and an `.intro__quote` italic line. **All sidebar copy is hardcoded in this layout file** — not in `_config.yml`, not in `_data/`. Edit `_layouts/default.html` directly when sidebar text changes.

- **`_includes/head.html`** — Loads a single stylesheet (`/assets/css/site.css`) and Google Fonts (Newsreader + Instrument Serif + Geist + IBM Plex Mono). Also emits OG/Twitter/feed/favicon metadata.

- **`_includes/foot.html`, `_includes/imprint.html`, `_includes/projects.html`** — legacy includes from the previous design. Not currently referenced by `default.html` or `index.html`.

- **`project_pages/`** — Standalone project pages (`hg3d.html`, `mocanet.html`, `smn.html`, `transmomo.html`, `wsr.html`). These are self-contained HTML files with their own `<head>` and styles; they do **not** use `_layouts/default.html` or `site.css`. They are out of scope for the editorial style system. `smn.html` still references `assets/fonts/iwona.ttf` and `assets/fonts/rockwell.ttf`, so those font files are kept.

- **`assets/`**:
  - `assets/css/site.css` — the only stylesheet for the main site (legacy `main.css`, `custom.css`, `gridism.css`, `neumorphic.css`, `ionicons.css` were deleted).
  - `assets/fonts/iwona.ttf`, `assets/fonts/rockwell.ttf` — kept only for `project_pages/smn.html`. All other legacy fonts (ionicons, opendyslexic) were deleted; main site fonts come from Google Fonts.
  - `assets/images/` — paper thumbnails and the profile photo (`me_o.jpg`).
  - `assets/files/` — PDFs (CV, supplementary).
  - `assets/<project>/` — per-project subdirectories (`smn/`, `transmomo/`, `mocanet/`, `wsr/`).

- **`_data/projects.yml`** — exists but currently unused by `index.html` / `default.html`. Publications are driven entirely by `_posts/`.

### Adding a New Publication

1. Create `_posts/YYYY-M-D-shortname-post.md` with front matter `layout: post` and a `title:` field.
2. Add the paper thumbnail to `assets/images/`.
3. In the post body, use the `.pub__*` pattern below. **No edit to `index.html` needed** — `group_by_exp` picks it up automatically and slots it into the right year band.

Post body template:

```html
<div class="pub__thumb-wrap">
  <img class="pub__thumb" src="/assets/images/PAPER.png" alt="">
</div>
<div class="pub__body">
  <div class="pub__venue-row">
    <span class="cap cap--ink">VENUE YEAR</span>
    <!-- optional, chartreuse pill: <span class="pub__flag">Oral</span> -->
  </div>
  <h3 class="pub__title">Paper Title</h3>
  <p class="pub__authors">
    Coauthor A, <span class="pub__authors-me">Zhuoqian Yang</span>, Coauthor B
  </p>
  <!-- optional: <p class="pub__note">* equal contribution</p> -->
  <div class="pub__links">
    <a class="chip chip--primary" href="…">project page</a>
    <a class="chip" href="…">paper</a>
    <a class="chip" href="…">code</a>
  </div>
</div>
```

Rules when authoring a post body:

- Thumbnail column is exactly 220px (`.pub__thumb-wrap`; `.pub__thumb` has `aspect-ratio: 16/9`). Make thumbnail images 16:9; don't override.
- Every occurrence of `Zhuoqian Yang` in `.pub__authors` — including the starred form `Zhuoqian Yang*` — must be wrapped in `<span class="pub__authors-me">…</span>`.
- The **first** link in `.pub__links` is the primary chip (`chip chip--primary`). Usually "project page" if one exists, otherwise "paper". Only one primary per row.
- For papers without a thumbnail image, use a placeholder:
  ```html
  <div class="pub__thumb-wrap">
    <div class="pub__thumb pub__thumb--placeholder">VIDEO</div>
  </div>
  ```
- Journal venues: use the abbreviated `IEEE TIP · 28(5)` form with a middle-dot separator. Don't carry over inline color from the old markup.

### Adding a New Project Page

Self-contained HTML in `project_pages/`, separate from the editorial style system:

```yaml
---
permalink: /yourproject/
redirect_from:
  - /yourproject
---
```

Reference assets with relative paths like `../assets/<yourproject>/...`. These pages keep their own `<head>`, fonts, and CSS — don't try to share `site.css` with them.

Every project page must include the MapMyVisitors snippet just before `</body>` (same one as in `_layouts/default.html`). It's wrapped in an off-screen container so the globe widget stays invisible while the script still tracks on load:

```html
<div style="position:absolute;left:-9999px;top:-9999px;width:1px;height:1px;overflow:hidden;" aria-hidden="true">
  <script type="text/javascript" id="mapmyvisitors" src="//mapmyvisitors.com/map.js?d=OVas2h6GXLWYYpmkZv04q_lmp_6GJZ8tzpKkc6_qt88&cl=ffffff&w=a"></script>
</div>
```

Exception: pages that immediately redirect off-site (like `hg3d.html`) can't just drop in the async script — navigation cancels it before it loads, and MapMyVisitors has no simple pixel-ping endpoint. Don't use the plugin's `redirect_to` front matter (it discards the page body entirely); instead put the URL in a custom front-matter key and, in the body, inject `map.js` and trigger the redirect on its `onload` (after a short delay so the beacon can fire), with a timeout + meta-refresh fallback (see `hg3d.html`).

### Key Config (`_config.yml`)

- `site.title`, `site.description`, `site.author`, `site.url` — used in `head.html` and the footer.
- `site.google_scholar`, `site.github_username`, `site.linkedin` — used by the sidebar tile grid in `_layouts/default.html`.
- `site.instagram_link`, `site.accent`, `site.g_analytics`, `site.address` — declared but not currently consumed by the editorial layout.
- The sidebar bio, "Previously" entries, supervisor links, and italic quote are hardcoded in `_layouts/default.html`, **not** in `_config.yml`.
