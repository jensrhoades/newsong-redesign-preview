# NewSong Full Site Rebuild — Claude Code handoff

## Mission

Recreate the **entire** live site at **https://newsongatl.com** inside this Hugo
project, in the already-approved new design. Every page, all the real content
(copy, photos, sections, buttons/CTAs). This is a content + templating job on top
of the design system that already exists in this repo — **not** a redesign.

**Work locally. Build the whole site, verify it, then STOP for review.**
Do NOT commit or push — Jens will review the local build with you and commit together.

Permissions: this session is expected to run with `--dangerously-skip-permissions`.
Work autonomously; only stop to ask if you hit a genuine content/scope decision
(see "When to check in").

---

## Where things stand (what already exists)

This repo was just ported from two flat HTML files into a working Hugo site.
Read these first — they define the conventions you must follow:

- `hugo.toml` — site config, contact **params**, and the shared `neighborhoods` list.
- `assets/css/main.css` — **all** styles, single source of truth. Brand vars in `:root`.
- `layouts/_default/baseof.html` — HTML shell; sets the `<body>` class.
- `layouts/partials/{head,header,footer,switcher}.html` — shared chrome.
- `layouts/index.html` — homepage sections.
- `layouts/_default/single.html` — the reusable **service-page** template (front-matter driven).
- `layouts/_default/list.html` — section index (`/services/`).
- `content/_index.md`, `content/services/_index.md`, `content/services/kitchens.md`.

Currently built pages: **Home** and **Services → Kitchens** (plus the `/services/` list).
`static/images/` holds **112** real NewSong photos already (see "Image map" below).

Also read the two memory files (auto-loaded via `MEMORY.md` in this project):
`newsong-redesign-deploy` and `newsong-redesign-gotchas`.

Environment: Windows, PowerShell + Git Bash. Hugo **extended v0.162.1**. Git repo,
branch `main`, remote `github.com/jensrhoades/newsong-redesign-preview`.
Deployed via GitHub Actions → Pages (build_type=workflow) at
`https://jensrhoades.github.io/newsong-redesign-preview/`.

Run locally:
```
hugo server
```
Full production build + link check:
```
hugo --gc --minify --baseURL "https://jensrhoades.github.io/newsong-redesign-preview/"
```
`public/` and `resources/` are gitignored build artifacts — never commit them.

---

## The design system contract (FOLLOW THIS EXACTLY)

### 1. Contact info lives in params — never hardcode it

`hugo.toml [params]`: `phone`, `phoneHref`, `addressLine1`, `addressLine2`, `rating`,
`serviceArea`, `showSwitcher`, `neighborhoods[]`. Reference as `{{ .Site.Params.phone }}`
etc. Header and footer already do this. Any new page must too.

### 2. Body-class scoping (this is how the CSS stays faithful)

`baseof.html` sets `<body class="page-home|page-service|page-default">`. `main.css`
scopes layout differences under those classes (`.page-home .intro .wrap` vs
`.page-service .intro .wrap`, heading scale, `section` padding, `.cta` sizing).

**Consequence you must respect:** `.page-default` pages do NOT get the `.intro` grid,
the service heading scale, etc. When you build new page types, you will extend the
body-class logic. Recommended change to `baseof.html`: also emit the section name so
each type can be scoped, e.g.:
```
{{- $bodyClass := "page-default" -}}
{{- if .IsHome -}}{{ $bodyClass = "page-home" }}
{{- else if eq .Section "services" -}}{{ $bodyClass = "page-service" }}
{{- else if .Section -}}{{ $bodyClass = printf "page-%s" .Section }}
{{- else if .File -}}{{ $bodyClass = printf "page-%s" .File.ContentBaseName }}{{- end -}}
```
Then any new section-specific layout rules go in `main.css` scoped under that class,
using the SAME merge discipline used already: if a class is reused with a different
layout on another page type, scope BOTH under their body class; watch specificity and
source order (later rule of equal specificity wins).

### 3. URL/path helpers — the subpath rule (do not regress this)

The site runs on a **subpath** now (`/newsong-redesign-preview/`) and moves to a root
custom domain later. So EVERY internal path must go through Hugo, never be hardcoded:

- Images/assets: `{{ "images/foo.jpg" | relURL }}`
- Link to home or home-anchors: `{{ site.Home.RelPermalink }}` then append `#anchor`.
  **Do NOT use `{{ "/" | relURL }}`** — it returns bare `/` and drops the subpath prefix.
  (This was a real bug; `site.Home.RelPermalink` is the fix.)
- Link to another page: `{{ (site.GetPage "/services/bathrooms").RelPermalink }}` or
  `{{ "services/bathrooms/" | relURL }}`.
- In `main.css`, image URLs are relative to `/css/`, so use `../images/...`
  (already done for `bg-26.png`).

### 4. Reusable CSS component vocabulary (compose new pages from these)

`main.css` already styles all of these. Reuse them to build faithful pages before
writing any new CSS:

- Layout: `.wrap` (max-width container), `.center`, `section` (vertical rhythm).
- Headings/eyebrows: `.kicker` (magenta uppercase label), `.sec-title`, `.sec-sub`.
- Buttons: `.btn` + `.btn-yellow` / `.btn-plum` / `.btn-outline` / `.btn-outline-plum`.
- Home hero: `.hero`, `.hero-img`, `.hero-rating`, `.hero-actions`.
- Overlapping cards: `.quicklinks`, `.ql-grid`, `.ql`, `.ql-img`, `.ql-body`, `.go`.
- Split image+text band: `.intro` (+ `.intro-img`, `.intro ul/li/.tick`).  ← scoped by body class
- Plum feature grid: `.services`, `.svc-grid`, `.svc-card` (`.ico` for the yellow icon chip).
- Yellow callout band: `.showcase` (`.pic` + `.txt`).
- Stats strip: `.stats`, `.stat` (`b` = big yellow number).
- Testimonials: `.tstm`, `.tgrid`, `.tcard` (`.stars`, `.who`).
- FAQ accordion: `.faq`, native `<details><summary>` + `.ans`.
- Service hero: `.svc-hero` (`img.bg`), `.crumb` breadcrumb.
- Sticky call rail: `.rail`.
- Gallery grid: `.gal`, `.gg` (image links, 3-up).
- Numbered process: `.process`, `.steps`, `.step` (auto-numbered via CSS counter).
- Location chips: `.areas`, `.chips`, `.chip`.
- Final CTA band: `.cta` (bg image + plum scrim).
- Architectural fade on white sections: add class `arch-fade` to any `<section>`.

Only add new CSS when a live section genuinely has no match. When you do: put it in
`main.css`, keep brand vars, scope it by body class if it could collide, and don't
introduce new colors/fonts — reuse `:root` vars (`--plum #594A5F`, `--yellow #EED82C`,
Bree Serif headings, Roboto body).

### 5. Service-page front-matter contract (`single.html`)

A new service page = one `content/services/<slug>.md`. Copy `kitchens.md` and edit.
Fields consumed by `layouts/_default/single.html`:

```yaml
title:            # H1 + <title>
crumbLabel:       # breadcrumb last segment (e.g. "Bathrooms")
description:      # meta description
ctaAnchor:        # header "Schedule a Call" target on this page (e.g. "#quote")
heroImage:        # "images/..." path under static/
lead:             # hero subhead sentence
introKicker:      # kicker above intro heading
introHeading:     # intro section H2
railText:         # sticky call-rail blurb
galleryKicker:    # e.g. "Recent Bathrooms"
galleryHeading:   # e.g. "A look at our work."
gallery:          # list of "images/..." paths (any length)
faqKicker:
faqHeading:
faq:              # list of {q, a}
areasHeading:     # H2 for the neighborhoods chips section
ctaKicker:
ctaHeading:
ctaText:
ctaImage:         # "images/..." for the final CTA band
# body (Markdown after front matter) = intro paragraphs
```
Generic bits (4-step process, call rail, neighborhood chips) are rendered by the
template and stay consistent across services automatically.

---

## Step 1 — Inventory the live site (firecrawl)

Firecrawl tools are available (may be deferred → load via ToolSearch:
`firecrawl_map`, `firecrawl_scrape`).

1. Map every URL:
   ```
   firecrawl_map url="https://newsongatl.com" sitemap="include" limit=300
   ```
   If the map is thin, retry with `sitemap="only"` and also a plain crawl of the nav.
2. Write the raw URL list to `scratch/site-map.txt` (create a `scratch/` dir; it's
   gitignored via `.claude/`? No — add `scratch/` to `.gitignore`) so you have a
   durable checklist. Mark each URL done as you build it.
3. For each real page (skip tag/category/pagination/feed noise), scrape:
   ```
   firecrawl_scrape url="<page>" formats=["markdown","links"] onlyMainContent=false
   ```
   Add `formats=["screenshot"]` (fullPage) for ONE representative page per template
   type (home, service, about, contact, our-work, service-area) to judge layout,
   photo placement, section order, and button styles. Don't screenshot every page.
4. From each scrape capture: page title, meta description, H1, the ordered list of
   sections with their headings + copy, every CTA (button label + destination),
   image URLs and what each is for, and any repeated data (service lists, FAQs,
   testimonials, neighborhoods, team members, awards).

**Use the real copy.** Do not paraphrase or invent marketing text — this is a faithful
reproduction. Preserve headings, body copy, testimonials, FAQ Q&A, phone/address
verbatim. Fix only obvious typos.

---

## Step 2 — Predicted inventory & image map (confirm against the real map)

The existing `static/images/` filenames strongly imply this structure. Treat as a
hypothesis; the firecrawl map is the source of truth.

| Likely page | Route | Key existing images |
|---|---|---|
| Home | `/` | `newsong-home-hero-image`, `newsong-home-about-image`, `newsong-make-us-different-image`, `newsong-design-build-company-image` (already built with different hero — reconcile to live) |
| About | `/about/` | `newsong-about-hero-image`, `about-us-title-1`, team: `newsong-member-{david,gabe,jason}-headshot`, `newson-expertise-image` |
| Services (overview) | `/services/` | `newsong-services-hero-image`, `newsong-services-{kitchen,bathroom,interiors,exteriors,porches}-{thumb,featured}` |
| Kitchens | `/services/kitchens/` | DONE |
| Bathrooms | `/services/bathrooms/` | `newsong-our-work-bathrooms-1..11`, `newsong-services-bathrooms-featured` |
| Interiors / Whole-home | `/services/interiors/` | `newsong-our-work-interiors-1..22`, `newsong-services-interiors-featured` |
| Exteriors | `/services/exteriors/` | `newsong-our-work-exteriors-1..10`, `newsong-services-exteriors-featured` |
| Porches | `/services/porches/` | `newsong-our-work-porches-1..8`, `newsong-services-porches-featured` |
| Our Work / Portfolio | `/our-work/` | `newsong-our-work-hero-image` + all `newsong-our-work-*` by category |
| Service Area | `/service-area/` | `newsong-service-area-hero-image` + `neighborhoods` param |
| Process | `/process/` (or a home section) | `newsong-process-image` |
| Guarantees | `/guarantees/` (or a home section) | `newsong-guarantees-image` |
| Contact | `/contact/` | `iboy-contact-hero` |
| Idea Book (lead magnet) | `/idea-book/` maybe | `newsong-idea-book-cover` |
| Footer CTA image | (partial) | `newsong-schedule-call-footer-image` |
| Awards/associations | (about/footer) | `NARI-white-200`, `nahb-200x140-1`, `apb_logo-1`, `inset-awards` |

Verify the actual service names/routes on the live site (they may differ, e.g.
"Kitchens & Baths" as one page). Match the live URL structure so redirects aren't
needed later. `homeschooling*.png`, `error-404.png`, `building.png`, `destination.png`
look like theme cruft — ignore unless a live page uses them.

---

## Step 3 — Build the templates

Reuse `index.html` (home) and `single.html` (services) as-is where possible. Create
new layouts for the other page types. In Hugo, a non-home single page resolves to
`layouts/_default/single.html` unless you give it its own type/layout. Options:

- **Distinct page types** (about, contact, service-area, our-work): create
  `layouts/_default/<name>.html` and set `layout: "<name>"` in that page's front
  matter (or use section folders + section layouts). Each `{{ define "main" }}...{{ end }}`
  composes the page from the CSS component vocabulary above, driving variable copy
  from front matter where it helps reuse.
- **Generic content page** (privacy policy, thank-you, simple text): make a plain
  `layouts/_default/page.html` with `.Content` inside a `.wrap` + `section`, and set
  `layout: "page"`.
- **Our Work / portfolio**: a gallery page grouping `newsong-our-work-<category>-*`
  images. Reuse `.gal`/`.gg`; consider a `data/` file or front-matter lists per category.

Extend `baseof.html` body-class logic (see §2) so each new type can be scoped in CSS.

Keep header/footer nav in sync: as you add pages, update `layouts/partials/header.html`
and `footer.html` link lists to match the live site's navigation. Nav/footer are edited
ONCE and apply everywhere — that's the point of the port, so get the live nav right.

Blog/news: if the map reveals a blog, replicate posts as `content/blog/<slug>.md` with
a `blog/single.html` + `blog/list.html`. If there are many (>~20), build the templates
and port a first batch, then flag the rest for Jens rather than silently truncating.

---

## Step 4 — Images: reuse first, fetch what's missing

1. Match live-site images to existing `static/images/` files by filename — most of the
   media library is already here (they came from `/wp-content/uploads/`).
2. For any image referenced on a live page that is NOT already in `static/images/`,
   download it into `static/images/` keeping a sensible filename:
   ```
   curl -L -o "static/images/<name>.jpg" "<live-image-url>"
   ```
   Prefer the site's original upload URL (firecrawl returns image src URLs).
3. Never route images through any MCP push; they're committed to git normally.
4. Reference every image via `{{ "images/<name>" | relURL }}`.

---

## Step 5 — Build & verify loop (repeat until clean)

After each batch of pages:
```
hugo --gc --minify --baseURL "https://jensrhoades.github.io/newsong-redesign-preview/"
```
- Build must report **0 errors** (warnings like languageCode/CRLF are fine).
- Grep the built `public/` for broken references:
  - No `src="images/` (bare relative) — everything should be `/newsong-redesign-preview/images/...`.
  - No bare `href=/#` links (subpath-dropping bug).
- Spot-check with `hugo server` + the in-app Browser pane (navigate to
  `http://localhost:1313/newsong-redesign-preview/...`) and read `read_network_requests`
  to confirm **no 404s** on CSS/images for each new page. Screenshot if the pane is visible.
- Confirm each new page's `<body class>` is what you intended and the design matches the
  live page's section order and photos.

Definition of done: every real page from the live site exists locally, builds clean,
renders on-brand with real content and working images, nav/footer updated, no broken
links or missing assets. Then STOP and write a summary for Jens (page list, any
judgment calls, anything you couldn't source).

---

## Guardrails & gotchas

- **Design is locked.** Plum `#594A5F`, yellow `#EED82C`, Bree Serif + Roboto,
  heart-in-hex logo, plum top bar, overlapping quick-link cards, alternating
  white/plum/yellow bands, `arch-fade` on white sections. Reuse `:root` vars; add no
  new colors/fonts. This is a faithful rebuild in the existing look, not a restyle.
- **Real content only.** Reproduce the live copy; don't invent or heavily rewrite.
- **Subpath discipline.** Always `relURL`/`site.Home.RelPermalink`; never hardcode `/…`.
  Re-check the "`/` | relURL drops the prefix" rule (§3).
- **Don't commit or push.** Local build for review. (Jens will commit with you.)
- **Pages stays on GitHub Actions source.** Don't touch Pages settings. The
  `README.md` landmine is neutralized as long as build_type=workflow.
- **When to check in:** genuine scope calls only — e.g. a large blog, a page whose
  purpose/content is ambiguous, a service whose URL/naming doesn't match, or content
  that can't be sourced from the live site. Otherwise proceed autonomously.

## Suggested task order

1. Read the existing files listed above; internalize `single.html` + `main.css`.
2. firecrawl_map → write `scratch/site-map.txt` checklist.
3. Scrape home + one of each page type; sketch the section/template mapping.
4. Extend `baseof.html` body classes; build the shared nav/footer to match live.
5. Build service pages (bathrooms, interiors, exteriors, porches) via `single.html` md files.
6. Build About, Services-overview, Our-Work, Service-Area, Contact, Process/Guarantees.
7. Scrape remaining pages, build them; fetch any missing images.
8. Full build + link/404 verification; fix.
9. Summarize for Jens. Do not commit.
```
