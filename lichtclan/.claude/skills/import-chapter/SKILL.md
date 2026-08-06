---
name: import-chapter
description: Import a new chapter of "Die Chronik des Lichtclans" from photos of Tabea's handwritten notebook pages into the static site. Use when the user asks to add/import a new chapter, gives a folder of notebook photos for the Lichtclan site, or says things like "add chapter N", "Kapitel N importieren", "hier sind die Fotos für das nächste Kapitel".
---

# Import a Lichtclan chapter

Turns photos of handwritten notebook pages into a new `kapitel-N.html` page on
this site: transcribe → lightly correct → extract illustrations → build the
page → wire up navigation. Follow the steps in order; don't skip the
verification step.

## 0. Groundwork

- Find the next chapter number `N`: highest existing `kapitel-*.html` + 1.
- Find the source photos. If the user didn't give a path, ask — don't guess a
  Downloads subfolder name.
- Photos are usually named `PXL_*.jpg` and are **not necessarily in reading
  order** — sort by filename timestamp, but always confirm order by content
  (page numbers are hand-written in a corner of most pages, decorated with a
  little sunburst).

## 1. Transcribe

Read each photo directly with the Read tool (vision) and transcribe the
handwritten German text, page by page, in reading order.

- **Rotated spreads**: sometimes the child wrote across a two-page spread
  with the notebook turned, so the photo shows sideways text. If lines look
  vertical/rotated in the image, don't try to read it rotated — fix it first:
  ```
  sips -r -90 <source.jpg> --out <scratchpad>/rotated.jpg
  ```
  (try `-90` or `90` depending on which way it's tilted), then Read the
  rotated copy. This is far more reliable than reading rotated text directly.
- Note the chapter title if the notebook page has one (usually decorated,
  top of the first page).
- Cross-check any line you're unsure of by re-reading that specific photo —
  don't guess at illegible words; look again.
- Watch for a page's text continuing directly into the next page's photo
  (mid-sentence) — stitch across the page break, don't add punctuation that
  isn't there.

## 2. Light copy-edit

Apply only **basic grammar and spelling corrections** — the user has been
explicit that the text should not change much beyond that. In practice this
means:

- Fix unambiguous spelling slips (e.g. child wrote "da" for "du", doubled
  letters missing like "Brennesel" → "Brennnessel").
- Add/fix quotation marks to the site's convention: „…" (German low-high
  quotes) around dialogue.
- Fix obvious case/article errors (e.g. "an die Seite" → "an der Seite" where
  it's clearly a slip, not a style choice).
- Do **not** rewrite sentences, smooth out the kid's voice, change word
  order, or "improve" phrasing that is just simple/informal. Keep run-on
  dialogue tags, keep the sentence rhythm.

## 3. Extract illustrations

The notebook has hand-drawn, colored-pencil doodles in the margins alongside
the text. Find them and crop them out as separate image assets — do not
skip this even if it's tedious.

1. Create `assets/kapitel-N/`.
2. For each drawing, crop from the (possibly rotated, see step 1) full-res
   photo with ImageMagick:
   ```
   magick <source.jpg> -crop <W>x<H>+<X>+<Y> <scratchpad>/crop.jpg
   ```
   Read the crop back to check it, and iterate the box — crop tight around
   the drawing, excluding as much surrounding handwriting as reasonably
   possible (a little incidental text/paper grain in frame is fine and
   matches the existing style; a name label crossing the whole image is not).
3. Copy final crops into `assets/kapitel-N/` with descriptive, lowercase
   filenames (character name if it's a character portrait, otherwise a short
   scene description) — no spaces, match the pattern used in
   `assets/kapitel-1/` (`mouse.jpg`, `tigercat.jpg`, etc.).
4. Make a **badge**: a tight square crop (~280×280) on one illustration's
   most recognizable detail (usually a face). This becomes
   `assets/kapitel-N/badge.jpg`, used both in the chapter header and on the
   index card.
5. Reuse `assets/kapitel-1/pawtrail.jpg` for scene-break dividers — it's a
   shared motif across chapters, don't duplicate it into the new folder.

## 4. Build the chapter page

Copy the structure of the most recent `kapitel-*.html` (currently
`kapitel-2.html`) — same header markup, `.dropcap` on the first paragraph,
`.scene-break` dividers, `.chapter-end` with "Fortsetzung folgt …", same nav
pattern. Update: title, meta description, badge image, kicker ("Kapitel N"),
chapter title, byline, body text, and the images.

**Never mention "Warrior Cats" anywhere** (title, alt text, captions, meta
description, commit message) — this is an original story, copyright issue
if the source inspiration is named. Only use the story's own invented names.

**Placing illustrations — avoid float squeeze**: the site's default doodle
style (`.doodle--left` / `.doodle--right`, `tilt`) floats a portrait-oriented
image beside body text, and works well when there's a long paragraph on
either side of a single image. It breaks down when several character
portraits land close together around short back-and-forth dialogue (e.g. a
naming-ceremony scene): opposing floats squeeze the paragraph between them
into a near-unreadable narrow column. If that shape of scene comes up again,
group the portraits instead of floating them individually:

```html
<div class="doodle-row">
  <figure class="doodle"><img src="..." alt="..." /><figcaption>Name</figcaption></figure>
  <figure class="doodle"><img src="..." alt="..." /><figcaption>Name</figcaption></figure>
</div>
```

`.doodle-row` (added to `style.css` for Kapitel 2) lays these out as a small
flex gallery instead of floating them. Use plain `.doodle` figures (no
`--left`/`--right`) inside it. Default to individual floated doodles for
normal scenes; reach for `.doodle-row` only when floats would actually
collide.

## 5. Wire up navigation

- Previous chapter (`kapitel-(N-1).html`): replace its disabled
  `<span class="disabled">Kapitel N (bald) →</span>` with a real
  `<a href="kapitel-N.html">Kapitel N →</a>`.
- New chapter's nav: `← Kapitel (N-1)` link back, disabled
  `Kapitel (N+1) (bald) →` placeholder forward.
- `index.html`: turn the current "Kapitel N — Folgt bald …" placeholder card
  into a real `<a class="chapter-card">` (badge, kicker, title, one/two
  sentence teaser summarizing the chapter), and add a fresh disabled
  placeholder card for `Kapitel N+1`.

## 6. Verify before calling it done

Don't eyeball the HTML and assume it's fine — render it:

```
python3 -m http.server <port> --directory <path-to-lichtclan>
```

then load `index.html`, the previous chapter, and the new chapter in the
Chrome tool and scroll through. Specifically check:

- The new chapter reads cleanly at desktop width — no squeezed text columns
  next to floated images (see step 4).
- Index card and both chapters' nav links go to the right place.
- **CSS edits don't show up on reload** the way you'd expect — Chrome caches
  `style.css` aggressively across plain navigations in the same tab. After
  editing `style.css`, hard-reload (`cmd+shift+r`) or you'll debug against a
  stale stylesheet.
- If a floated/flex image row looks stuck at the wrong width, check for the
  flex min-content trap: a flex item containing an `<img>` won't shrink
  below the image's intrinsic size unless you set `min-width: 0` on it
  explicitly — `width` alone isn't enough.

Kill the http.server and close any scratch browser tabs when done.

## 7. Commit

Only commit/push if the user asked for it (or asks after reviewing). Follow
the repo's existing commit style (see `git log` — short, e.g.
`feat(lichtclan): Kapitel N`).
