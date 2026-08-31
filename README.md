# Penn MEDIATED — Events

The events page for the [Center on Media, Technology and Democracy](https://infodem.upenn.edu) — the Fall 2026 seminar schedule and past public events. Static HTML/CSS, no build step. Deploys directly to https://infodem.upenn.edu/events/ (or the new site's equivalent URL) — see "Deployment" below.

- `index.html` — page markup
- `styles.css` — all styling (design tokens live at the top in `:root`)
- `assets/` — the Fall 2026 schedule PDF and its rendered preview image, plus event photos (the Media Fragmentation panel tile, and the two Information & Democracy kickoff photos)

## Deployment

This repo deploys straight to the live site — no GitHub Pages hosting step, no WordPress iframe embed. Same mechanism as `about`, `grants`, and `team-leadership`:

- A clone of this repo lives on the department's web server (eniac) at the path WordPress resolves `/events/` to. The server's `.htaccess` defers to real files/directories on disk before handing a request to WordPress, so this repo's own files are what actually serve the live URL — there's no WordPress Page involved at that URL anymore.
- Deploy is triggered by a GitHub webhook (push to `main` → WordPress REST endpoint → `git pull`) once that infrastructure is live — see the top-level `Website` folder's migration notes if you're setting this up before that's wired up.
- To undo a live mistake: `git revert` the bad commit and push it, same as any other change. **Don't** `git reset --hard` + force-push — the deploy mechanism expects a normal fast-forward `git pull`.
- The server only ever pulls; it never pushes back to GitHub. Edits should always originate here (in GitHub), not by hand-editing files directly on the server.

Full setup process (SSH access, deploy keys) is documented in `eniac-github-ssh-setup.md` at the top level of the `Website` folder.

This repo's design system is copied from [`about`](https://github.com/PennMEDIATED/about) — see that repo's README for the canonical spacing/color/type tokens and component conventions. Don't redefine a token or component pattern here that already exists there; pull the value from `about` instead so the pages don't drift apart. `--pad-x` here is responsive (32px under 900px, 20px under 480px), same as `home`/`grants`/`team-leadership`.

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

## Components

- **Hero** (`.events-hero`): a serif, accent-purple `.events-hero__title` ("Events") and a plain sans lede filling the section's full padded width. Previously had a `.eyebrow` ("Penn MEDIATED") above the title and a `max-width: 760px` cap on the lede, both matching `about`'s Mission Statement block / `team-leadership`'s hero — both removed 2026-08-29 as a sitewide decision (see "No eyebrow" below); the lede's cap went with it since the section comment already said it should fill the full width.
- **Seminar section** (`.seminar`): full-bleed brand-gradient backdrop — the same treatment as `about`'s Orbital section and `team-leadership`'s Core Team block, so this, the page's primary content block, reads as the lead chapter. Heading and body copy are white for contrast on the gradient (matching `.orbital__title`/`.orbital__lead`'s pattern), with body-copy links in white/underlined rather than the usual red, for the same reason.
- **Seminar copy + featured schedule PDF** (`.seminar__copy` / `.seminar__pdf` / `.seminar__pdf-link` / `.seminar__pdf-cta`): a copy column (heading + intro, capped at `max-width: 560px` for a tighter reading measure) and a fixed-width PDF card (`width: min(34vw, 400px)`), vertically centered against each other and centered as a pair within the section (`justify-content: center`, not pinned to the container's edges) — unlike `home`'s `.research-cta__inner`, which space-betweens its copy and image out to the edges; here the two columns read as one balanced, evenly-margined composition instead. Stacks to one column under 900px. The PDF card itself is a white tile floating on the gradient — same "card-arrow badge, hover lift" language as `about`'s school-block/center-block/partner-card tiles, just unconstrained by their 4:3 aspect ratio since this is a full document page rather than a logo — showing a rendered preview of the actual schedule PDF, with a text CTA beneath it to download the file directly. Replaced an earlier hand-transcribed 8-row schedule list and a logistics callout box, both removed once the real PDF was available to feature instead.
- **Past events** (`.past-events` / `.event-card`): a two-column grid (stacks to one column under 900px) of `.event-card`s — a small red date tag, title, and body copy, matching the type scale of `grant-card`/`news-card` elsewhere in the design system. `.event-card__desc` is capped at `max-width: 560px` (added 2026-08-29) so its lines don't run the full ~616px column width — a comfortable reading measure, matching `.seminar__copy`'s own cap. The Media Fragmentation card uses `.event-card__media` (a 16:9 tile showing the real event photo, linked out to the `@PennMediated` YouTube channel — same "image tile + card-arrow badge + hover lift" language as `about`'s school-block/center-block/partner-card tiles), followed by a `.event-card__links` row of bordered link containers (white fill, 1px `--c-red` border, hover lift — same tile language, just pill-sized and with a brand-red outline instead of the tiles' usual hairline gray) rather than free-floating underlined text. The kickoff card's two captioned photos (`.event-card__photo-grid`) stay a 2-up grid at every width (never collapsing to one column, even on narrow screens) and are capped at the same `max-width: 560px` as `.event-card__desc` so the pair's right edge lines up with the body copy above it. (Briefly changed to a single stacked column on 2026-08-29 over concern that side-by-side read too small at 560px; reverted 2026-08-31 — stacking dropped the second photo underneath the first on narrow screens and blew each one up disproportionately, unbalancing it against the single-image Media Fragmentation card next to it. At 560px each photo is ~272px, which reads fine paired with its caption.)
- **External-link arrow badge** (`.card-arrow`): copied verbatim from `about/styles.css`, plus a smaller inline variant inside the `.event-card__links` bordered containers (20px instead of 26px, vertically centered against the link text instead of pinned to a card corner).
- **Hyperlinks in body copy** (`.event-card__caption a`): inline links inside flowing prose use one shared treatment sitewide: `color: var(--c-red)`, `font-weight: 500`, no underline, `opacity: 0.7` on hover, no color change. This is the reference implementation the rest of the design system copies — see `.post-excerpt a` / `.modal-body a` (`blog`), `.intro__body a` (`data`/`grants`), and the bio/card-desc link rules in `our-team-faculty`. The one exception on this page itself is `.seminar__body a`, which sits on the purple gradient backdrop and uses white + underline instead, for contrast reasons specific to that surface — don't treat it as a second pattern to choose from elsewhere.
- **No eyebrow above hero/section headings** — sitewide convention as of 2026-08-29. This page's `.eyebrow` ("Penn MEDIATED") was removed along with its CSS rule; don't reintroduce one here or on any other repo.
- **Scroll-triggered reveal**: one-shot only (no `.reveal--toggle`), same choice as `team-leadership` — this is a long, bottom-heavy content page, and `about`'s README documents a scroll-jitter "spasming" bug when a toggle-based reveal section sits at the very bottom of the page. One-shot avoids that.
- **`overflow-anchor: none`** (on `html`/`body`): same fix as `team-leadership`, for the same reason — prevents the browser's scroll-anchoring from nudging the viewport (and cropping the hero title) when web fonts swap in a beat after first paint.

## Keeping in sync

If you change a shared token or component pattern here, check whether `about` (and by extension `home`/`grants`/`team-leadership`) should get the same change, and vice versa — these repos duplicate CSS rather than sharing a stylesheet, so consistency is a discipline, not something enforced automatically.
