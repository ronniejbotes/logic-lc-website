# logiclc.com — static mirror

A 1:1 static HTML copy of the live WordPress site at **https://logiclc.com/**, captured
**3 September 2026**.

Source stack that was mirrored: WordPress 7.1 + Elementor 4.2.3 / Pro Elements 4.2.2 +
`hello-elementor` theme + Yoast SEO 28.4 + Google Site Kit.

Nothing has been redesigned, rewritten, cleaned up or "improved". Copy first, optimise later.

---

## URL and file mapping

Every live URL keeps its exact path, so **no 301 redirects are needed for the migration
itself**.

| Live URL | File in this repo |
|---|---|
| `https://logiclc.com/` | `index.html` |
| `https://logiclc.com/services/` | `services/index.html` |
| `https://logiclc.com/category/projects/` | `category/projects/index.html` |
| `https://logiclc.com/wp-content/uploads/…` | `wp-content/uploads/…` (identical path) |
| `https://logiclc.com/feed/` | `feed/index.xml` |
| `https://logiclc.com/sitemap_index.xml` | `sitemap_index.xml` |
| 404 template | `404.html` |

Asset paths (`/wp-content/…`, `/wp-includes/…`) were deliberately left unchanged so that
every image URL Google has already indexed still resolves.

---

## What is in here

**70 HTML pages**, plus the 404 template:

| Type | Count |
|---|---|
| Published pages | 32 |
| Blog posts | 7 |
| Category archives | 13 |
| Tag archives | 11 |
| Author archives | 2 |
| Date (day) archives | 5 |

Nine of the category archives, seven of the tag archives and one author archive are **not
in the Yoast sitemap** (they hold no posts), but they return HTTP 200 and `index, follow`
on the live site, so they were mirrored too.

**897 files / 225 MB total**, including 592 images, 82 stylesheets, 93 scripts, 35 RSS
feeds and the 7 Yoast sitemap files.

> Originally 800 files. The second pass (below) added 70 Elementor runtime chunks, 3
> conditional CSS/JS assets, 17 uncaptured RSS feeds and 3 redirect stubs. See
> **Second pass — runtime assets**.

---

## The only two deliberate changes to the HTML

1. **Internal asset and link references were made root-relative.**
   `https://logiclc.com/wp-content/x.png` → `/wp-content/x.png`, `https://logiclc.com/services/`
   → `/services/`. This is what lets the site be previewed and worked on before cutover.

   **Explicitly left absolute, byte-for-byte as WordPress emitted them:** `<link rel="canonical">`,
   every `og:*` / `twitter:*` / `article:*` meta tag, the full Yoast `application/ld+json`
   schema graph, `rel="alternate"` RSS links, `shortlink`, `EditURI`, oEmbed links, and all
   `<loc>` values in the sitemaps and feeds. None of those SEO signals changed.

2. **Header and footer logo links.** On the live site both are `href="https://logiclc.com"`
   (no trailing slash). Stripping the host would have left `href=""`, which browsers resolve
   as *the current page* — a broken self-link on every page. They are now `href="/"`, the
   same destination.

That is the complete list. Verified below.

---

## Needs configuring on the new host

1. **Three 301 redirects that already exist on the live WordPress site** and must be carried
   over:

   | From | To | Why it matters |
   |---|---|---|
   | `/home` | `/` | This is the **“Home” link in the header nav on all 70 pages** |
   | `/contact/` | `/contact-us/` | Linked from 4 pages |
   | `/prjects/` | `/projects/` | A `<loc>` in `page-sitemap.xml`, so crawlers will request it |

   (`/prjects/` is a typo slug that Yoast still lists in `page-sitemap.xml`. That listing was
   left untouched — it is exactly what the live sitemap says today.)

   **Interim stubs are now in place** at `home/index.html`, `contact/index.html` and
   `prjects/index.html`. Each is a `noindex, follow` page carrying a canonical to the real
   destination plus a meta-refresh and a `location.replace()`. They exist so the nav does not
   404 while previewing. **They are not a substitute for a real 301** — configure the three
   redirects at the host and delete the stub directories at cutover.

2. **`404.html` as the error document.**

3. **Trailing-slash directory indexes** — the host must serve `foo/index.html` for `/foo/`.
   Netlify, Cloudflare Pages, Vercel, GitHub Pages, Apache and nginx all do this by default.

4. **Feed content type — needs a host rewrite.** All 35 feeds declared in the pages'
   `<head>` are stored as `index.xml` inside their directory, because `index.html` is what a
   static host serves for a directory. Without a rewrite, every `/…/feed/` URL advertised on
   the site will 404 on GitHub Pages (or return a directory listing under
   `python -m http.server`). Add a rewrite from `/*/feed/` → `/*/feed/index.xml` served as
   `application/rss+xml` (Netlify, Cloudflare Pages and nginx all support this; GitHub Pages
   does not). Copying each `index.xml` to `index.html` is **not** a fix — it would serve RSS
   as `text/html` and leave two divergent copies of every feed to maintain.

5. **`.nojekyll`** is present so GitHub Pages does not strip underscore-prefixed paths.

---

## Issues that exist on the LIVE site and were carried over unchanged

These were **not** introduced by the migration. They are broken on logiclc.com right now and
were left alone per the "copy exactly, fix later" brief.

- `/commercial-security-surveilance` — a misspelled internal link (missing an `l`) that
  returns 404 on the live site. Linked from the five date archive pages.
- `/wp-content/uploads/2026/07/Renewix-Dark-v3-scaled-1.png` and `…/Renewix-Light-v2-scaled-1.png`
  — referenced by `/style-guide/`, both 404 on the live site (leftover theme demo assets).
- **Elementor forms** (the contact form) submit to `/wp-admin/admin-ajax.php`. Once WordPress
  is gone there is no endpoint behind that. The form markup is preserved exactly; it will
  need a form handler before go-live.
- `wp-content/uploads/2026/08/250914_AF64_InSitu_Data_Center_v006.png` is **48.9 MB**. It is
  served on the live site as-is.

---

## Still loaded from third parties (unchanged from live)

Google Fonts (`fonts.googleapis.com` / `fonts.gstatic.com`) for Onest, Inter Tight and Jost;
the Trustindex reviews widget (`cdn.trustindex.io`); Google Tag Manager / Site Kit. These
were left external so the pages behave identically to the live site.

---

## Preview locally

```bash
python -m http.server 8000
# then open http://127.0.0.1:8000/
```

Root-relative paths mean the site must be served from a web root, not opened via `file://`.

---

## Second pass — runtime assets (4 September 2026)

The first pass captured everything the HTML *references*. That is not everything the site
*loads*.

**Elementor fetches its widget logic as lazy webpack chunks at runtime**, resolved by
`webpack.runtime.min.js` from a chunk-id → filename map inside the JavaScript. Those URLs
appear in no `<script src>` tag, so a reference-following crawler cannot see them. All **70**
were missing, and every one 404'd the moment a page loaded.

The result looked like a frozen screenshot of the page, because functionally that is what it
was — the markup was perfect and the behaviour was amputated:

| Symptom | Missing chunk |
|---|---|
| Counters stuck at `0` (the `data-to-value="30"` was always in the HTML) | `counter.*.bundle.min.js` |
| Header dropdowns dead on hover — **no route to 21 of the site's pages** | `nav-menu.*.bundle.min.js` |
| Partner-logo marquee frozen and overflowing its container | `image-carousel.*.bundle.min.js` |
| Video lightbox never opened | `lightbox.min.css` + `share-link.min.js` |
| Tabs, accordions, nested widgets inert | `nested-tabs.*`, `nested-accordion.*`, `shared-frontend-handlers.*` |

### What was added

| Added | Count | How it was found |
|---|---|---|
| Elementor / Pro lazy webpack chunks | 70 | Extracted the chunk map from `__webpack_require__.u` in both webpack runtimes |
| `conditionals/dialog.min.css` | 1 | 404 observed in a real browser session |
| `conditionals/lightbox.min.css`, `lib/share-link/share-link.min.js` | 2 | Only surface when a video lightbox is actually clicked |
| RSS feeds that were never captured | 17 | 35 feeds are declared in the pages' `<head>`; only 18 existed |
| Redirect stubs (`/home`, `/contact/`, `/prjects/`) | 3 | See "Needs configuring on the new host" |

All downloaded files were byte-size-verified against live.

### How this pass was verified

Reference-following was replaced with **browser-driven** verification — each page opened in
Chromium, scrolled top to bottom, every nav item hovered, every tab, accordion and carousel
control clicked, then live and mirror compared on measured values:

- **JavaScript errors across all 70 pages: 0** (previously 9 distinct `ChunkLoadError`s on the
  homepage alone).
- **Same-origin 404s across all 70 pages: 2** — the two `Renewix-*` images, which 404 on live
  too.
- **Counters** reach their exact target on every page that has them (home 30/500/100,
  `/about-us/` 30/48, `/services/` all eight).
- **Partner carousel** measured identical on both: 30 slides, 18 duplicates, 124 px slide
  width, **40 px gap**, autoplay running.
- **Dropdowns**: SmartMenus initialises with identical options on both
  (`showOnClick:false, showTimeout:250, hideTimeout:500`), and screenshots of the open About
  (4 items), Services (5) and Industries (14) menus are **md5-identical** between live and
  mirror.
- **Full-page screenshots**: 50 of 70 pixel-identical to live. The other 20 differ by ~0.1%
  of pixels, localised to the logo marquee — it is caught at a different point in its scroll
  cycle, i.e. the difference exists *because* the animation now runs.

### The live site drifted after the capture — Trustindex reviews widget

Worth understanding, because it will happen again the longer this snapshot sits.

The homepage Google-reviews widget rendered on live but was **blank on the mirror**, making the
page 293 px shorter. The cause was not the capture: **the Trustindex plugin was updated on
logiclc.com from 13.3.2 to 14.0 after 3 September.**

- **13.3.2** (what the mirror captured) emitted the widget as
  `<pre class="ti-widget" style="display:none"><template id="trustindex-google-widget-html">…`,
  relying on Trustindex's CDN `loader.js` to hydrate it.
- **14.0** (what live serves now) emits the finished widget markup inline, no hydration needed.

Trustindex serves **one** `loader.js` to every customer, and it has moved to 14.0 — so it no
longer hydrates the 13.3.2 template form. The mirror's markup was orphaned by a third-party
CDN update.

**Fixed** by re-capturing that one widget block on `index.html` from live and root-relativising
it, per the same convention as the rest of the mirror. Verified: mirror and live now both
report `docH=8093`, widget height `278 px`, and identical heights for all 12 homepage sections.

Two consequences to plan for:

1. The 8 reviews are now **frozen static HTML** and will not refresh. That is inherent to a
   static mirror — decide before go-live how reviews stay current.
2. This is the only post-capture drift found, but it is proof the snapshot ages. Re-capture
   close to cutover rather than relying on the 3 September copy.

### Known remaining divergences (deliberate)

- `/sitemap.xml` returns a 200 duplicate of `sitemap_index.xml`; live 301s. A meta-refresh
  stub would be worse — no XML parser follows one. Add a host redirect, or leave it.
- 141 `/wp-json/` REST and oEmbed URLs referenced in every `<head>` 404 here. No browser
  requests them; the head links were left byte-identical to live on purpose.
- Server-side-only routes (`/?s=` search, `/wp-admin/`, `/xmlrpc.php`) cannot work statically.
- **Forms post to `/wp-admin/admin-ajax.php` and will show a red "error"** on any static host.
  Deliberately left 1:1 with live rather than patched, so the markup stays faithful. Confirmed
  behaviour: a fully valid submission on the mirror POSTs to a non-existent endpoint, the user
  sees `.elementor-message-danger` reading "error", and **the enquiry is lost**. This is the
  site's primary conversion path (1 form on `/contact-us/`, 1 on `/coming-soon/`, 3 demo forms
  on `/style-guide/`) — **it needs a form handler wired up before go-live.** Client-side
  validation is identical to live, so the form looks healthy right up until submit.

---

## Third pass — removing WordPress plumbing (4 September 2026)

The site no longer runs WordPress, so markup that only exists to serve WordPress was
removed. **389 KB** in total.

The governing rule: Yoast *generated* the SEO markup, but that markup **is** the SEO. The
meta description, canonical, robots, `og:`/`twitter:`/`article:` tags and the JSON-LD
`@graph` all stay. Only the plumbing went.

### Removed

| Item | Scope |
|---|---|
| Yoast plugin HTML comments | 142 |
| `rel="https://api.w.org/"` REST discovery link | 71 |
| Per-page `/wp-json/` JSON alternate links | 64 |
| `EditURI` / `xmlrpc.php?rsd` links | 71 |
| oEmbed discovery links (JSON + XML) | 76 |
| `generator` metas — WordPress, Site Kit, Elementor | 213 |
| Emoji polyfill — settings JSON, inline loader, inline CSS | 213 |
| Site Kit content-events provider + its inline config | 71 |
| WordPress-internal `<body>` classes | 71 pages |
| Orphaned files: `wp-emoji-loader.min.js`, `wp-emoji-release.min.js`, the Site Kit no-op | 25,752 B |
| Yoast branding comments in `robots.txt` | — |

`robots.txt` **directives are unchanged** — only the comment block went. The emoji settings
JSON, inline loader and inline CSS had to be removed *together*: the loader begins
`throw new Error("Element missing: script#wp-emoji-settings")`, so removing either alone
would have thrown a JS error on all 70 pages.

Body classes: WordPress state (`wp-singular`, `wp-custom-logo`, `wp-theme-*`,
`page-template*`, `single-format-standard`) and numeric-ID duplicates (`page-id-144`,
`category-24`, `tag-19`, `postid-1489`) are gone. **Slug-based classes were kept**
(`category-projects`, `tag-house`, `author-kevincto`) — they are useful CSS hooks — along
with every `elementor-*` class and the 10 classes that CSS or JS actually references.

### Deliberately NOT removed — each was tested and found load-bearing

Every candidate was probed empirically: render the page, render it again with the asset
blocked, and compare pixels, computed styles, behavioural state and console errors. A
per-page noise floor was measured first (two unblocked renders) so a rotating carousel
could not fake a difference.

| Kept | Why — measured, not assumed |
|---|---|
| `block-library/style.min.css` | Removing it thins `.wp-block-separator` from **2px to 1px**. The content uses `wp-block-*` classes 600+ times. |
| `jquery-migrate.min.js` | **All 70 pages** log `JQMIGRATE: Number-typed values are deprecated for jQuery.fn.css("--menu-height")`. The caller is Elementor Pro's `nav-menu` chunk — dropping Migrate risks the mobile menu height. |
| `jquery/ui/core.min.js` | Blocking it throws a JS error and changes page dimensions on `/`. |
| `imagesloaded.min.js` | Blocking it throws a JS error on every archive page. |
| `dist/hooks.min.js`, `dist/i18n.min.js` | Elementor depends on both; blocking either throws. |
| `lib/dialog/dialog.min.js` | Looks orphaned to a text search — Elementor builds the URL at runtime. |
| `ElementorProFrontendConfig.urls.rest` (`/wp-json/`) | A config value the JS reads, not a link. |
| `wp-admin/admin-ajax.php` in the form config | Same: inert here, but the JS expects the key. |

**Most of the WordPress-core JavaScript is not removable** — Elementor is built on it.

### Verification

An SEO fingerprint of all 74 files — title, description, canonical, robots, every
`og:`/`twitter:`/`article:` tag, a normalised hash of the JSON-LD graph, plus stylesheet and
script counts — was captured before the cleanup and re-checked after each step:
**0 differences**.

### `_raw/` is intentionally untracked

`_raw/` (8.3 MB, 109 files) is the untouched capture, kept for verification. It is **not
committed** and is now listed in `.gitignore`, so a git-based deploy never ships it. It
exists only on the machine that made the capture — archive it elsewhere if it matters.

---

## Verification performed on this copy (first pass)

Every one of the 70 pages was fetched live and compared against its local file. Note that the
reference-resolution check below follows references *declared in the markup*, which is why it
reported clean while 70 runtime-fetched chunks were absent — see the second pass above:

- **Visible text** — word-for-word diff: **0 differences** on all 70 pages.
- **`<title>`, `<meta name="description">`, heading tree (h1–h6), image count** — **0 differences**.
- **Internal link sets** — identical apart from the two logo hrefs described above.
- **Rewrite fidelity** — each local file diffed against its unmodified live capture; the only
  changes are host-prefix removal and those two logo hrefs.
- **Reference resolution** — all 774 distinct local references (assets, pages, CSS `url()`,
  `srcset`, escaped JSON paths) were requested over HTTP against the served copy. Every one
  returns 200 except the five entries listed above, which are the live site's own redirects
  and 404s.
- **Inventory cross-check** — the WordPress REST API was enumerated (pages, posts, categories,
  tags, users, media) and every published, publicly reachable item has a corresponding file here.
