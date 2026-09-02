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

**800 files / 224 MB total**, including 592 images, 79 stylesheets, 22 scripts, 18 RSS
feeds and the 7 Yoast sitemap files.

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
   over. They are redirects on the live site, so they were not turned into pages here:

   | From | To |
   |---|---|
   | `/home` | `/` |
   | `/contact/` | `/contact-us/` |
   | `/prjects/` | `/projects/` |

   (`/prjects/` is a typo slug that Yoast still lists in `page-sitemap.xml`. That listing was
   left untouched — it is exactly what the live sitemap says today.)

2. **`404.html` as the error document.**

3. **Trailing-slash directory indexes** — the host must serve `foo/index.html` for `/foo/`.
   Netlify, Cloudflare Pages, Vercel, GitHub Pages, Apache and nginx all do this by default.

4. **Feed content type.** Feeds are stored as `index.xml` inside their directory. To keep
   `/feed/` returning RSS rather than a directory listing, either add a rewrite from
   `/feed/` to `/feed/index.xml` or accept that feed readers must use the `.xml` URL.

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

## Verification performed on this copy

Every one of the 70 pages was fetched live and compared against its local file:

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
