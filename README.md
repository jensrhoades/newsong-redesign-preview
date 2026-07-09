# NewSong Renovations — Site Rebuild (Direction C: Faithful Refresh)

Working files for the NewSong website rebuild. Direction C keeps the current brand
(plum #594A5F, chartreuse-yellow #EED82C, Bree Serif, heart-in-hex logo, overlapping
plum quick-link cards) and refreshes structure, spacing, and photography.

Final build target: Hugo. These HTML files are the approved design reference.

## Files

- `index.html` — homepage
- `kitchens.html` — service page template (kitchens); other services will follow this pattern
- `images/` — real NewSong photography goes here (see manifest below)

The pill switcher (top-right) links the two pages together for review. It can be
removed before the Hugo port.

## Image manifest

Images are referenced by path, not embedded. Drop real photos into `images/` using
these exact filenames. Suggested source files from the current site's media library
(`/wp-content/uploads/2024/12/`) are listed, but any on-brand replacement is fine.

### Homepage (index.html)

| Filename | Slot | Suggested source |
|---|---|---|
| `hero-exterior.jpg` | Hero background (historic home) | `newsong-our-work-exteriors-5.jpg` |
| `card-kitchen.jpg` | Quick-link card: Kitchens | `newsong-our-work-kitchens-1.jpg` |
| `card-bathroom.jpg` | Quick-link card: Bathrooms | `newsong-our-work-bathrooms-4.jpg` |
| `card-wholehome.jpg` | Quick-link card: Whole Home | `newsong-our-work-interiors-5.jpg` |
| `card-exterior.jpg` | Quick-link card: Exteriors | `newsong-our-work-exteriors-2.jpg` |
| `about.jpg` | "Who We Are" split image | `newsong-our-work-exteriors-7.jpg` |
| `featured-project.jpg` | Yellow featured-project callout | `newsong-our-work-interiors-8.jpg` |
| `cta-interior.jpg` | Bottom CTA background | `newsong-our-work-interiors-13.jpg` |

### Kitchen service page (kitchens.html)

| Filename | Slot | Suggested source |
|---|---|---|
| `kitchen-hero.jpg` | Hero background | `newsong-our-work-kitchens-1.jpg` |
| `kitchen-gallery-1.jpg` … `kitchen-gallery-6.jpg` | Gallery grid | any six kitchen/interior shots |
| `cta-interior.jpg` | Bottom CTA (shared with homepage) | `newsong-our-work-interiors-13.jpg` |

Recommended: resize hero/CTA images to ~1500px wide, cards/gallery to ~800px, JPEG
quality ~70, to keep the page fast.

## Brand notes (confirm before final commit)

The plum and yellow here are pulled from the live website's CSS. Before locking the
build, verify these match NewSong's printed collateral (vehicle wraps, cards, signage).
If they differ, update the CSS variables at the top of each file:

```
--plum:#594A5F;
--yellow:#EED82C;
```

## Other directions (for reference)

Directions A (Warm Editorial) and B (Modern Craft) were also explored. C was chosen.
A and B files exist in the review outputs if needed later.
