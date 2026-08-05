# NewSong Redesign - Hugo Port Plan (Claude Code handoff)

## What this is

Convert the approved flat-HTML preview (`index.html`, `kitchens.html`) into a proper
Hugo site with a shared header and footer, so future edits touch one file instead of
every page. This is the one-time scaffolding job. After it's done, adding pages and
editing copy happens in normal chat via github_push and patch_file.

Repo: `jensrhoades/newsong-redesign-preview`
Current state: two flat HTML files at repo root, plus `images/` folder with real photos.
Target: Hugo project that builds those same two pages from shared layouts, ready for more service pages.

Design is locked. Do NOT restyle. Preserve the exact look: plum #594A5F, yellow #EED82C,
Bree Serif + Roboto, heart-in-hex logo, plum top bar, overlapping quick-link cards,
alternating white/plum/yellow bands, architectural-drawing fade (bg-26.png) on white sections.
Colors live as CSS variables at the top of the stylesheet; keep them there so they stay easy to match to print collateral.

---

## Start clean

Per standing practice, begin by removing any local copy of the repo dir, then clone fresh
over HTTPS (not SSH):

```
rm -rf newsong-redesign-preview
git clone https://github.com/jensrhoades/newsong-redesign-preview.git
cd newsong-redesign-preview
```

The repo currently has: `index.html`, `kitchens.html`, `images/`, `README.md`,
and `media-images.zip` (the full-res photo archive; leave it or move it out of the web root).

---

## Target structure

```
newsong-redesign-preview/
├── hugo.toml                      # site config
├── archetypes/
│   └── default.md
├── assets/
│   └── css/
│       └── main.css               # ALL the CSS extracted from the current <style> blocks
├── layouts/
│   ├── _default/
│   │   ├── baseof.html            # the HTML shell: <head>, header partial, {{ block "main" }}, footer partial
│   │   ├── single.html            # service/content pages
│   │   └── list.html              # (optional) section index pages
│   ├── index.html                 # homepage layout (the home-specific sections)
│   └── partials/
│       ├── head.html              # <head> contents: meta, fonts, CSS link
│       ├── header.html            # top bar + sticky nav (edit once, applies everywhere)
│       ├── footer.html            # footer (edit once, applies everywhere)
│       └── switcher.html          # the preview page-switcher pills (remove before real launch)
├── content/
│   ├── _index.md                  # homepage content/front matter
│   └── services/
│       └── kitchens.md            # kitchen service page content
├── data/
│   └── services.toml              # (optional) the 6 service cards as data, so the grid is generated
├── static/
│   └── images/                    # move the real photos here (Hugo serves /static at web root)
└── .github/
    └── workflows/
        └── hugo.yml               # GitHub Pages build + deploy
```

---

## Step-by-step

### 1. Init and config

`hugo new site . --force` (into the cloned repo), then write `hugo.toml`:

```toml
baseURL = "/"
languageCode = "en-us"
title = "NewSong Renovations"

[params]
  phone = "678-325-2436"
  phoneHref = "6783252436"
  address = "641 Schuyler Ave SE, Atlanta, GA 30312"
  rating = "130+ Google reviews"
  serviceArea = "Serving Intown Atlanta & the North Georgia Mountains"

[markup.goldmark.renderer]
  unsafe = true   # allow raw HTML in content where needed
```

Putting phone/address in params means a phone-number change is one edit in hugo.toml,
not a find-replace across pages. Reference as `{{ .Site.Params.phone }}`.

### 2. Move images

Move `images/` to `static/images/`. Hugo serves `static/` at the web root, so existing
`src="images/..."` paths keep working unchanged. Move `media-images.zip` out of the web
root (or delete from the deploy; it's the archive, not a site asset).

### 3. Extract CSS

Both HTML files share one big `<style>` block (nearly identical). Extract it to
`assets/css/main.css` as the single source of truth. Diff the two blocks first: the
kitchens page has a few page-specific rules (`.svc-hero`, `.rail`, `.gg`, `.steps`,
`.chip`) that the homepage lacks. Merge everything into one main.css; page-specific rules
are harmless when unused. Keep the `:root` variables block at the very top.

Link it in `head.html` via Hugo asset pipeline:
```
{{ $css := resources.Get "css/main.css" | resources.Minify | resources.Fingerprint }}
<link rel="stylesheet" href="{{ $css.RelPermalink }}">
```

### 4. Build the partials (the whole point of this port)

**head.html** — everything currently between `<head>` and `</head>`: charset, viewport,
title (use `{{ .Title }} — NewSong Renovations`), the Google Fonts preconnect + Bree
Serif/Roboto link, and the CSS link from step 3.

**header.html** — the `.topbar` + `<header>` nav block, verbatim from the current pages.
Replace hardcoded phone/area with `{{ .Site.Params.phone }}` etc. This is the file Jens
edits when the nav or phone changes, once, forever.

**footer.html** — the `<footer>` block, verbatim. Replace the services list and contact
details with param/data references where practical. This is the file that fixes the
"had to edit the footer on every page" problem.

**switcher.html** — the `.switcher` pills. Keep as a partial so it's trivial to delete
before the real site goes live. Include it only while previewing.

### 5. baseof.html

The shell that pulls it together:
```
<!DOCTYPE html>
<html lang="en">
<head>
  {{ partial "head.html" . }}
</head>
<body>
  {{ partial "switcher.html" . }}
  {{ partial "header.html" . }}
  {{ block "main" . }}{{ end }}
  {{ partial "footer.html" . }}
</body>
</html>
```

### 6. Homepage: layouts/index.html

Everything between `header` and `footer` on the current `index.html` (hero, quick-links,
intro/about, services, showcase, stats, testimonials, faq, cta) goes inside:
```
{{ define "main" }}
  ... homepage sections ...
{{ end }}
```
Content/copy that Jens will edit (hero headline, lead, stats) can either stay inline for
now or move to `content/_index.md` front matter. Recommendation: keep the homepage
sections in the layout for this pass; migrate to front matter later only if he wants to
edit homepage copy without touching HTML. Don't over-engineer it now.

### 7. Service pages: single.html + content

`layouts/_default/single.html`:
```
{{ define "main" }}
  ... the service-page structure (svc-hero, intro+rail, gallery, process, faq, areas, cta) ...
{{ end }}
```
Drive the variable bits from front matter so each new service is just a content file.
`content/services/kitchens.md` front matter example:
```
---
title: "Kitchen Remodeling in Atlanta"
heroImage: "images/newsong-our-work-kitchens-1.jpg"
lead: "From cabinets and countertops to walls-out additions..."
gallery:
  - images/newsong-our-work-kitchens-2.jpg
  - images/newsong-our-work-kitchens-3.jpg
  - images/newsong-our-work-kitchens-4.jpg
  - images/newsong-our-work-kitchens-5.jpg
  - images/newsong-our-work-kitchens-7.jpg
  - images/newsong-our-work-kitchens-9.jpg
---
Body copy / intro paragraphs here.
```
Once this template exists, a NEW service page (bathrooms, whole-home, custom homes) is a
single new .md file in chat. That's the payoff.

### 8. Services grid as data (recommended)

Put the 6 homepage service cards in `data/services.toml` so the grid is generated by a
`range` and edits are one file:
```toml
[[card]]
title = "Kitchens & Baths"
blurb = "The rooms your home revolves around, finished with the detail luxury deserves."
icon = "kitchen"   # maps to an SVG partial or inline switch

[[card]]
title = "Custom Home Building"
blurb = "Ground-up custom homes designed and built to an exacting standard."
icon = "home"
# ... etc, in display order; Historic Restorations last
```
Icons: keep the existing inline SVGs, selected via a `{{ if eq .icon "home" }}` switch in
a small partial. If that's fiddly, leaving the six cards as static HTML in index.html is
acceptable for now, the data approach is the nice-to-have, not the blocker.

### 9. Deploy: GitHub Pages workflow

`.github/workflows/hugo.yml` — standard Hugo Pages action (peaceiris or the official
actions/deploy-pages). Build on push to main, publish. Confirm Pages is enabled on the
repo pointing at GitHub Actions. baseURL may need to be the Pages URL if served from a
subpath; if using a custom domain or root, "/" is fine.

### 10. Verify before finishing

- `hugo server` locally: homepage and /services/kitchens/ both render identical to the
  current flat pages (same look, same photos, arch-fade visible, logo present).
- Edit `footer.html` once, confirm the change shows on BOTH pages. That proves the port did its job.
- Check mobile breakpoint (900px) still collapses correctly.
- Confirm images load from /images/ (i.e. static/images/).

---

## Guardrails

- Keep all six current service cards and the exact copy already written. Don't rewrite content.
- Preserve the plum/yellow/Bree-Serif identity exactly; this is a structural port, not a redesign.
- Large/binary work (images already committed) stays in git; don't route images through any MCP push.
- Once this is merged and building, STOP. Further pages and copy edits happen in normal chat.

## After the port (these are chat jobs, not Claude Code)

- New service pages: bathrooms, whole-home, exteriors, basements/ADUs (each a content .md).
- Custom Homes page: needs its own content plan (different buyer: land, architecture,
  build process, ground-up portfolio) before building.
- Photo swap into hero/prominent slots when Jason sends the new set.
- Location pages later.
