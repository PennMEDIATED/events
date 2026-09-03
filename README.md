# Penn MEDIATED — Events

The events page for the [Center on Media, Technology and Democracy](https://infodem.upenn.edu) — the Fall 2026 seminar schedule and past public events. Static HTML/CSS, no build step.

- `index.html` — page markup
- `styles.css` — all styling (design tokens live at the top in `:root`)
- `assets/` — the Fall 2026 schedule PDF and its rendered preview image, plus event photos (the Media Fragmentation panel tile, and the two Information & Democracy kickoff photos)

This repo's design system is copied from [`about`](https://github.com/PennMEDIATED/about) — see that repo's README for the canonical spacing/color/type tokens and component conventions. Don't redefine a token or component pattern here that already exists there; pull the value from `about` instead so the pages don't drift apart. `--pad-x` here is responsive (32px under 900px, 20px under 480px), same as `home`/`grants`/`team-leadership`. Grid and flex children shrink below their content: grid tracks are `minmax(0, 1fr)` rather than `1fr`, and flex items that hold text carry `min-width: 0`. Without those, a track or item is pinned to its widest child and pushes the page wider than the viewport on small screens.

This page does **not** end in the shared Newsletter + Supporters closing block that `about`, `home`, and `grants` use — it was built, then deliberately removed here along with the "Amy Gutmann Hall" venue blurb, so the page now ends with Past Events. If a closing block is wanted back later, copy it fresh from one of those three repos rather than trying to restore it from this repo's git history, since the design system may have moved on by then.

## Updating content

This page was built from the content on the live `infodem.upenn.edu/events/` page. Nothing is left as a placeholder: the two kickoff photos in the "Information and Democracy Research Seminar" card and the Media Fragmentation panel tile all use real assets (`assets/Amir-seminar-presentation.jpg`, `assets/engler-introduction.jpg`, `assets/Media-Fragmentation.jpg`).

The "Democratic Repercussions of Media Fragmentation" card (`.event-card__media`) links out to the `@PennMediated` YouTube channel rather than embedding an inline player or a separate "watch" link — a deliberate choice, not a placeholder. If a direct link to that specific video's recording is ever wanted, update the `href` on `.event-card__media` to the video's own URL (and consider re-adding a link to `.event-card__links` alongside the Daily Pennsylvanian coverage).

To update the Fall 2026 schedule: replace `assets/fall-2026-seminar-schedule.pdf` with the new flyer, then re-render its preview image at the same path —

```
pdftoppm -png -r 300 assets/fall-2026-seminar-schedule.pdf page
python3 -c "from PIL import Image; im = Image.open('page-1.png').convert('RGB'); w,h = im.size; nw = 1100; im.resize((nw, int(h*nw/w))).save('assets/fall-2026-seminar-schedule-preview.jpg', quality=88, optimize=True)"
rm page-1.png
```

— and commit both files together. The PDF is the single source of truth for the schedule's details (dates, speakers, affiliations, room, time, virtual-link policy); nothing in `index.html` duplicates them, so there's no separate markup to keep in sync.

To add a new past event: copy a `.event-card` block inside `.past-events__grid` (a two-up grid; a third card wraps to its own row under the 900px breakpoint, so nothing needs to change there for a growing list — though a fast-growing archive may eventually want its own "past events" sub-page rather than living entirely on this one).

## Typography

Sitewide convention. The `--fs-*`/`--lh-*` block at the top of `styles.css` is canonical and identical in every page repo.

**Two families, no third.** `--f-serif` (EB Garamond) for page and section titles and pull-quote copy; `--f-sans` (DM Sans) for everything else. There is no monospace face — uppercase micro-labels are DM Sans 700 uppercase with `letter-spacing: 0.08em`.

**Sizes come from tokens, never raw px.**

| Token | Mobile (=<480px) | Desktop (>=1440px) | Used for |
| --- | --- | --- | --- |
| `--fs-display` | 36px | 76px | full-bleed hero |
| `--fs-h1` | 36px | 56px | page title |
| `--fs-h2` | 26px | 40px | section titles |
| `--fs-h3` | 20px | 24px | card and third-level titles |
| `--fs-lede` | 18px | 20px | intro paragraphs |
| `--fs-body` | 16px | 16px | body copy |
| `--fs-small` | 14px | 14px | captions, meta, form controls |
| `--fs-small-serif` | 15px | 15px | EB Garamond at small sizes |
| `--fs-micro` | 12px | 12px | uppercase labels, tags, counts |

The top five are `clamp()` values that interpolate across the viewport, so tablet widths need no separate `@media` override. Only add a breakpoint font-size when a specific layout actually demands it.

**12px is the floor.** Nothing ships smaller. EB Garamond and uppercase-with-letter-spacing both read smaller than their nominal size, which is what `--fs-small-serif` and the 12px floor exist to absorb.

**Line heights are tokens too** — `--lh-display` 1.05, `--lh-heading` 1.15, `--lh-lede` 1.26, `--lh-title` 1.3, `--lh-body` 1.55. Never set a line-height in px; it breaks the fluid sizes.

**Heading gaps.** Section title to first content is `var(--space-300)` (24px); page or hero title to content is `var(--space-250)` (20px).

**Section rhythm.** A full-width colored section carries `var(--space-1000)` (80px) top and bottom padding, so its heading never sits flush against the band's edge. The page hero's bottom padding is `var(--space-600)` (48px) — shorter than 80px because the section below supplies its own.

## Components

- **Hero** (`.events-hero`): a serif, accent-purple `.events-hero__title` ("Events") and a plain sans lede filling the section's full padded width. Previously had a `.eyebrow` ("Penn MEDIATED") above the title and a `max-width: 760px` cap on the lede, both matching `about`'s Mission Statement block / `team-leadership`'s hero — both removed 2026-08-29 as a sitewide decision (see "No eyebrow" below); the lede's cap went with it since the section comment already said it should fill the full width.
- **Seminar section** (`.seminar`): full-bleed brand-gradient backdrop — the same treatment as `about`'s Orbital section and `team-leadership`'s Core Team block, so this, the page's primary content block, reads as the lead chapter. Heading and body copy are white for contrast on the gradient (matching `.orbital__title`/`.orbital__lead`'s pattern), with body-copy links in white/underlined rather than the usual red, for the same reason.
- **Seminar copy + featured schedule PDF** (`.seminar__copy` / `.seminar__pdf` / `.seminar__pdf-link` / `.seminar__pdf-cta`): `.seminar__inner` is the same two-column grid as `.past-events__grid` below it (`repeat(2, 1fr)`, `--space-600` gap, top-aligned), rather than a bespoke centered flex composition — the copy column is capped at `max-width: 560px` (matching `.event-card__desc`'s reading measure) *while it shares the row with `.seminar__pdf`*; that cap is lifted (`max-width: none`) once `.seminar__inner` stacks to one column under 900px, so the copy fills the full single-column width instead of leaving a block of empty space on the right (fixed 2026-08-31 — previously the cap carried through the stack unconditionally). The PDF card stays capped narrower still, at `max-width: 360px`, at every width — it's a document thumbnail, not reading copy, so it isn't meant to stretch — and left-aligns within its column rather than filling it edge-to-edge or centering under 900px (that centering used to visibly knock it out of alignment with the left-aligned copy above it once the section stacked to one column). The PDF card itself is a white tile floating on the gradient, unconstrained by the 4:3 aspect ratio `about`'s school-block/center-block/partner-card tiles use since this is a full document page rather than a logo — showing a rendered preview of the actual schedule PDF, with a text CTA beneath it to download the file directly. No `.card-arrow` badge on the tile itself (removed 2026-08-31 — the badge read as redundant next to the explicit "Download the Full Schedule (PDF)" CTA directly below it); `.card-arrow` is still used elsewhere on the page (`.event-card__media`). Replaced an earlier hand-transcribed 8-row schedule list and a logistics callout box, both removed once the real PDF was available to feature instead.
- **Past events** (`.past-events` / `.event-card`): a two-column grid (stacks to one column under 900px) of `.event-card`s. `.past-events__grid` uses `align-items: stretch` (changed from `start` on 2026-08-31) so both cards in a row match the height of the taller one, rather than each hugging its own content and leaving the boxes visibly different lengths — unlike `.seminar__inner` above it, which stays `start`-aligned since its two children (a paragraph and a document thumbnail) aren't meant to look like matching cards. Each `.event-card` is now a bordered white box (`padding: var(--space-800)`, a hairline `rgba(13,13,12,0.08)` border, and a soft `0 20px 60px rgba(13,13,12,0.08)` shadow) — the same "card floating on the page" treatment as `home`'s `.about-center__card` (padding drops to `var(--space-400)` under 900px, matching that card's own mobile step). Inside: a small red date tag, title, and body copy, matching the type scale of `grant-card`/`news-card` elsewhere in the design system. `.event-card__desc` is capped at `max-width: 560px` (added 2026-08-29) so its lines don't run the full ~616px column width — a comfortable reading measure, matching `.seminar__copy`'s own cap — *while the grid is two columns*; under 900px, once `.past-events__grid` stacks to one column, the cap lifts (`max-width: none`, fixed 2026-08-31) so the copy fills the now much wider single column instead of leaving empty space on its right. The Media Fragmentation card uses `.event-card__media` (a 16:9 tile showing the real event photo, linked out to the `@PennMediated` YouTube channel — same "image tile + card-arrow badge + hover lift" language as `about`'s school-block/center-block/partner-card tiles), capped at the same `max-width: 560px` as `.event-card__desc` above 900px; that cap also lifts under 900px now (fixed 2026-08-31 — it used to carry through the single-column stack specifically to stop the tile from ballooning to the full container width, but that traded a full-bleed image for a large empty gap on the right of the card, which read worse), followed by a `.event-card__links` row using the same "dark text, subtle underline, turns red only on hover" plain-text-link pattern as `home`'s `.about-center__link` — a long-arrow (`⟶`) after the text instead of the `.card-arrow` badge, and red/orange only appearing as a hover state rather than a permanent border (changed 2026-08-31; previously a bordered white-fill pill with an always-visible red outline). The kickoff card's two captioned photos (`.event-card__photo-grid`) stay a 2-up grid at every width (never collapsing to one column, even on narrow screens) and are capped at the same `max-width: 560px` as `.event-card__desc` above 900px, so the pair's right edge lines up with the body copy above it; that cap lifts together with `.event-card__desc`'s under 900px (fixed 2026-08-31) so the alignment holds at the new, wider single-column width too, instead of leaving the photo pair narrower than the paragraph above it. (Briefly changed to a single stacked column on 2026-08-29 over concern that side-by-side read too small at 560px; reverted 2026-08-31 — stacking dropped the second photo underneath the first on narrow screens and blew each one up disproportionately, unbalancing it against the single-image Media Fragmentation card next to it. At 560px each photo is ~272px, which reads fine paired with its caption.)
- **External-link arrow badge** (`.card-arrow`): copied verbatim from `about/styles.css`, used on the `.event-card__media` image tile. `.seminar__pdf-link` used to carry one too but had it removed (2026-08-31, see above). `.event-card__links` does *not* use it either — that link uses a plain inline `⟶` character instead, matching `home`'s `.about-center__link`.
## Images and video

This applies to every image, GIF and video added to any Penn MEDIATED repo. It is written to be followed directly — by a person or by a Claude session — without further instruction.

### The one rule that is never optional

**Every `<img>` and `<video>` carries explicit `width` and `height` attributes, holding the file's real intrinsic pixel dimensions.**

```html
<img src="assets/example.webp" width="640" height="334" alt="…">
```

They do not set the display size — CSS does. They give the browser the aspect ratio *before* the file downloads, so it reserves a correctly shaped box instead of collapsing to nothing and shoving everything below it down the page as each file lands. That shift is measured by search engines (Cumulative Layout Shift) and is worse for a reader, who loses their place or clicks a link that just moved.

Every repo has a global `img, video { max-width: 100%; height: auto; display: block; }` reset, so the CSS keeps winning and the attributes only ever contribute the ratio. **Never guess the numbers** — read them off the file.

### Pick the format by what the file is

| Content | Format | Never use |
| --- | --- | --- |
| Photo, screenshot, artwork | **WebP**, quality 88 | PNG or JPEG at full camera resolution |
| Logo, wordmark, icon | **SVG** if you have it, else WebP | — |
| Anything that moves | **MP4** (H.264) + a WebP poster | **GIF, ever** |

GIF is the big one. It has no interframe compression, so a screen recording is roughly ten times the size it needs to be: `research-compendium.gif` was 11.3MB for 290 frames; the identical recording as H.264 is 1.2MB.

### Size it to the box it displays in, not to what you were sent

Find the CSS box the image renders into, then export at **2×** that width for retina. Anything beyond that is bytes the browser downloads and immediately throws away. (`gni-membership.png` was 7992px wide, rendering into a 319px box — a 470KB file doing a 33KB job.)

In this repo:

| Where | CSS box at 1440px | Export at |
| --- | --- | --- |
| Event card media (`.event-card__media-img`) | 486×273, cropped | ~972px wide |
| Photo-pair tile (`.event-card__photo-frame img`) | 235×176, cropped | ~470px wide |
| Seminar PDF preview (`.seminar__pdf-img`) | 344px wide | ~688px |

If you are adding an image somewhere not listed, measure the box first (`getBoundingClientRect().width` in the browser, at a 1440px viewport) and double it.

### Commands

Stills — resize and convert in one pass:

```python
from PIL import Image
TARGET = 640                      # 2x the CSS box
im = Image.open('source.png')
w, h = im.size
if w > TARGET:
    im = im.resize((TARGET, round(h * TARGET / w)), Image.LANCZOS)
im.save('out.webp', quality=88, method=6)
print(im.size)                    # <- these are the width/height attributes
```

Animation — MP4 plus a poster frame:

```bash
ffmpeg -i source.gif -movflags +faststart -pix_fmt yuv420p \
       -vf "scale=1280:-2:flags=lanczos" -crf 24 out.mp4
ffmpeg -i source.gif -frames:v 1 -vf "scale=1280:-2:flags=lanczos" poster.png
python3 -c "from PIL import Image; Image.open('poster.png').convert('RGB').save('out-poster.webp', quality=80, method=6)"
ffprobe -v error -show_entries stream=width,height -of default=nw=1 out.mp4
```

`-crf 24` is a good default; raise it toward 30 for a smaller file, lower it toward 20 for a sharper one. `-pix_fmt yuv420p` is required for Safari and iOS.

### Markup for video

```html
<video src="assets/name.mp4" poster="assets/name-poster.webp" width="1280" height="622"
       autoplay muted loop playsinline preload="metadata" aria-label="…"></video>
```

Each attribute earns its place: `muted` is what permits autoplay at all, `playsinline` stops iOS opening it fullscreen, `poster` means the slot is never empty while the video loads, and `aria-label` replaces `alt` (a `<video>` has no `alt`).

CSS cannot stop autoplay, so **a page with video needs the reduced-motion script** at the end of `<body>`. If the page already has one, leave it alone; if you are adding the first video to a page, add it:

```html
<script>
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
    document.querySelectorAll('video[autoplay]').forEach(function (v) {
      v.autoplay = false; v.pause(); v.currentTime = 0; v.removeAttribute('loop');
    });
  }
</script>
```

Also check the CSS: any rule that sizes or crops an image needs to name `video` too, or the video slot will not match the image slot it replaced (`.card__image img` becomes `.card__image img, .card__image video`).

### Before you call it done

- [ ] File is WebP, SVG or MP4 — no GIF, no full-resolution PNG or JPEG
- [ ] Its width is about 2× the CSS box it renders into
- [ ] `width`/`height` attributes match the file's real dimensions
- [ ] Real `alt` text (or `aria-label` on a video) that describes the image; empty `alt=""` only if it is purely decorative
- [ ] Lives in this repo's `assets/`, not hotlinked from another site
- [ ] Page opened in a browser at 1440px and ~400px — nothing overflows, nothing jumps on load
- [ ] Originals are not committed alongside the optimised file; git history is the backup

Do not commit an unoptimised original "just in case" — the previous commit already holds it, and a duplicate in the working tree also ships to the server.

## Hyperlinks

One taxonomy, five categories, shared by every page repo. Pick the category by what the link *is*, not by which repo you happen to be editing.

**1. In-text links** — embedded mid-sentence in flowing prose.

| ground | text | underline | hover |
| --- | --- | --- | --- |
| white / light | `--c-red-dark` | none | fade to `opacity: 0.7` |
| colour / gradient | `--c-white` | `border-bottom: 1px solid rgba(255, 255, 255, 0.5)` | fade to `opacity: 0.7` |

Both grounds use `font-weight: 500` and `transition: opacity 0.15s`, and both fade rather than change hue. On a white ground **colour is the affordance** — no underline; the underline is category 2's job. On a coloured ground the red is invisible, so the link goes white and takes the hairline rule instead. Where an underline is used it is a `border-bottom`, never `text-decoration`.

#### Why interactive red is `--c-red-dark`, not `--c-red`

`--c-red-dark` (`#df3611`) is the closing stop of `--c-gradient`, promoted to a token of its own and declared in all twelve repos.

`--c-red` (`#f03d1f`) measures roughly **3.9:1** against white — under the 4.5:1 WCAG AA threshold for body text, and the same 3.9:1 applies to white text sitting on a `--c-red` fill. `--c-red-dark` measures about **4.5:1** either way and clears it. The two are near-indistinguishable at text sizes, so this is a contrast fix, not a visual change.

**The rule: anything you click is `--c-red-dark`.** Links and buttons take it wherever they would otherwise be red-orange — as text colour, as a box fill, as a hover or active state, and on the markers inside them (disclosure chevrons and their labels). It applies in every category and every state.

**`--c-red` stays the brand accent for everything you don't click**: section headings, eyebrow and metadata labels, tag and pill backgrounds, accent bars and card borders, full-width colour bands, the `.card-arrow` hover gradient, and focus rings. These are either large text, non-text UI at the 3:1 threshold, or sit on a tinted rather than white ground.

The one deliberate hold-out is red link text on a **dark** ground (`home`'s `.footer__email`), where the darker red would *reduce* contrast rather than improve it. That link has a separate outstanding issue — on a dark ground the standard is white text with an opacity fade, not red at all.

**2. Independent links** — a standalone text link that isn't inside a sentence ("Learn More About the Center", "Download the Full Schedule"). Unlike category 1 these carry the underline and are set in the body colour, so they read as a control rather than as emphasis inside a sentence:

| ground | text | underline | hover |
| --- | --- | --- | --- |
| white / light | `--c-dark`, `font-weight: 600` | `border-bottom: 1px solid rgba(13, 13, 12, 0.35)` | text and underline both turn `--c-red-dark` (`transition: color 0.15s, border-color 0.15s`) |
| colour / gradient | `--c-white`, `font-weight: 600` | `border-bottom: 1px solid rgba(255, 255, 255, 0.5)` | fade to `opacity: 0.7` |

Plus a **thin arrow** `⟶` after the text. Use `⟶` (`&#10230;`), not the `↗` badge from category 4.

**3. Document buttons** — an independent link that opens a document (a PDF, a report). A filled button box, not text:

| ground | box | text |
| --- | --- | --- |
| white / light | `--c-red-dark` | `--c-white` |
| colour / gradient | `--c-white` | `--c-dark` |

Hover is **movement, not colour** — a lift or nudge. Do not darken or recolour the box.

**4. Links to another web page** — this site or an external one. The containing box carries the shared `.card-arrow`: a 26px dark circle with a white `↗`, in the box's top corner. On hover the arrow scales slightly and its background becomes a sliding purple-to-orange gradient (`@keyframes card-arrow-slide`), and the box itself animates. No separate text button — the whole box is the link.

**Exception:** a link to a research paper is category 2, not this — thin arrow, no badge.

**5. Hyperlinked headings** — a heading that is itself a link (a post title, a card title). Sits in the body colour and shifts to `--c-red-dark` on hover (or fades, on a coloured ground), with **no arrow and no underline**.

### Dropdowns and disclosures

A dropdown, `<details>` block or expand/collapse control uses one affordance sitewide: a **chevron SVG** (`M2 5l5 5 5-5`, 13×13, `--c-red-dark` stroke, `stroke-width: 1.8`) beside a `--c-red-dark` label at `--fs-small`, rotating `180deg` on open with `transition: transform 0.25s`. See `llm-civic-discourse`'s "Full summary & details" toggle for the reference implementation.

Never leave the marker to the browser — style `<select>` with `appearance: none` and supply the chevron, and hide the native `<summary>` marker. The `↗` circle badge is category 4's language and does not belong on a disclosure control.

- **"Featuring" names** (`.event-card__featuring` / `.event-card__featuring span`): a small uppercase `--f-sans` eyebrow label ("FEATURING", 11px/700) above the speaker names. The names themselves (`span`) are `--f-sans` too, at the same 11px size as the label (600 weight, sentence case, no letter-spacing) — switched from `--f-serif` at 16px on 2026-08-31, first to match the label's family (sitewide convention: serif is reserved for headlines, quotes, and the "MEDIATED" wordmark; everything else is sans), then to match its size as well.
- **No eyebrow above hero/section headings** — sitewide convention as of 2026-08-29. This page's `.eyebrow` ("Penn MEDIATED") was removed along with its CSS rule; don't reintroduce one here or on any other repo.
- **Scroll-triggered reveal**: one-shot only (no `.reveal--toggle`), same choice as `team-leadership` — this is a long, bottom-heavy content page, and `about`'s README documents a scroll-jitter "spasming" bug when a toggle-based reveal section sits at the very bottom of the page. One-shot avoids that.
- **`overflow-anchor: none`** (on `html`/`body`): same fix as `team-leadership`, for the same reason — prevents the browser's scroll-anchoring from nudging the viewport (and cropping the hero title) when web fonts swap in a beat after first paint.

## Keeping in sync

If you change a shared token or component pattern here, check whether `about` (and by extension `home`/`grants`/`team-leadership`) should get the same change, and vice versa — these repos duplicate CSS rather than sharing a stylesheet, so consistency is a discipline, not something enforced automatically.
