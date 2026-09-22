# OMS-Configurator

Custom seating OMS configurator — a single, responsive page (`index.html`)
that works on both desktop and mobile, no separate mobile link required
(`mobile.html` just redirects to `index.html` for old links).

`index.html` is fully self-contained: every layer image, all vinyl /
airmesh / platilon / leather / wrapping swatches, the vendored jsPDF
library, the Eqwal Ability logo/circle backdrop and the title font are
embedded directly in the file as data URIs. You can open it straight
from disk (double-click it, or drag it into a browser tab) with nothing
else alongside it, or serve it from any static file host / GitHub
Pages — both work identically.

## How it's built

- Plain HTML/CSS/JS, no build step, no dependencies at runtime other than
  Google Fonts (Noto Sans body text) loaded over the network — everything
  else works offline.
- Same layout system, render engine and stacked-footrest-canvas approach
  as the Supportec configurator (its own README documents the reasoning
  in full) — white background, Noto Sans body copy, the licensed Bogue
  Semibold Italic title font, the Eqwal Ability logo top-left, and the
  soft grey circle backdrop behind the seat preview.
- The studio hand-off was again a 2752×1536 canvas per layer; every layer
  was cropped to a shared box, `(601, 3, 2192, 1522)` (left, top, right,
  bottom) on the original canvas, giving a 1591×1519 working canvas —
  tight enough to bound the chair, headrest and footrest across every
  combination of zones and add-ons, without cutting off the two side
  wings of the back (the widest element) or the footrest (the
  bottom-right-most). `#seat-canvas`'s CSS `aspect-ratio` is set to
  `1591/1519` to match.
- Layer images (bare seat, frontal panel, outside shell, back, back
  lateral, seating, abduction wedge, headrest shell/upholstery, footrest
  shell/upholstery) are drawn back to front exactly as on Supportec:
  "complete" (the bare seat, no headrest/footrest) always drawn first,
  each zone tinted with a canvas `multiply` blend against its own artwork
  so shading/highlights always show through, headrest/footrest drawn
  additively on top only when their toggle is on.
- **Materials differ from Supportec, by zone:**
  - **"Shell"-type zones — Outside shell (Shell tab), Headrest shell and
    Footrest shell** — all share the same three-way material choice:
    **Vinyl, Leather or Wrapping** (`SHELL_MATERIALS`, applied via the
    shared `buildShellMaterials()` helper), each as its own sub-tab of
    swatches. Frontal panel used to be a fourth shell-type zone but has
    moved to the Upholstery tab (see below) — the Shell tab now holds
    only the Outside shell.
  - **Wrapping** is a new, fourth material family: printed transfer
    patterns (Dino, Leopard, Lamas, Licorne, Pandas, Jungle, Espace,
    Citron, Flamants, Tigres, Ferrari), not flat colors. Its swatches
    are tiled fabric-scan photos rather than hex chips, and selecting
    one fills the zone with a repeating `canvas` pattern (`createPattern`
    against a small tile image, drawn through the same `source-in` +
    `multiply` mask as every color) instead of a solid `fillStyle`.
  - **Headrest upholstery** (the inside) offers **Airmesh, Platilon or
    Leather** — no Vinyl — as three sub-tabs of colors (the
    `.subtabs`/`.sub-pane` pattern also used everywhere else multiple
    materials apply to one zone).
  - **Footrest upholstery** offers **Vinyl or Bare foam**. Bare foam is
    now rendered **black** (tinted like any other color, via the normal
    `multiply` pipeline) rather than left untinted/white — it's still a
    single swatch with no color list, just a black one instead of white.
  - **Upholstery tab** — now five sections: **Frontal panel** (moved
    here from the Shell tab), Back, Back lateral, Seating, Abduction
    wedge — each offers **Leather, Platilon or Airmesh**, no Vinyl, as
    three sub-tabs of colors per zone.
- Vinyl (15), Platilon (4), Airmesh (7) and Leather (18) swatches are
  the same arrays as before (Vinyl/Platilon/Airmesh shared with
  Supportec/Janton; Leather sampled from the "Eqwal Ability - Leather"
  reference sheet). **Wrapping (11)** is new: Dino 172080, Leopard
  173705, Lamas 172085, Licorne 172081, Pandas 172083, Jungle 172082,
  Espace 172084, Citron 172078, Flamants 172079, Tigres 172086, Ferrari
  172630 — names and reference codes as given by Eqwal Ability (the
  codes are the studio's real reference numbers, unlike every other
  material array here, which use made-up sequential codes).
  Each swatch's `img` is a small square crop from a clean, label-free
  area of that pattern's reference sheet — chosen by an automated search
  for the lowest-"whiteness" (i.e. most fully covered by print, no gaps
  or stray label text) square in a safe region of the page — then made
  seamlessly tileable via 2×2 mirror-tiling (the crop plus horizontal-,
  vertical- and both-flipped copies), so every edge matches its neighbor
  exactly and `canvas`'s `createPattern('repeat')` shows no visible seam
  or gap between repeats. The mirrored tile is reused directly as both
  the round swatch preview (CSS `background-image`) and the canvas
  pattern source, base64-encoded as JPEG (patterned photos compress far
  smaller as JPEG than PNG, and don't need alpha).
- Tabs: **Shell** (Outside shell: Vinyl/Leather/Wrapping) →
  **Headrest** (optional, toggle switch; Vinyl/Leather/Wrapping shell +
  Airmesh/Platilon/Leather upholstery) → **Footrest** (optional, toggle
  switch; Vinyl/Leather/Wrapping shell + Vinyl/Bare foam upholstery) →
  **Upholstery** (five collapsible sections — Frontal panel, Back, Back
  lateral, Seating, Abduction wedge — each Leather/Platilon/Airmesh).
- **View order** unlocks once the shell finish and all five upholstery
  sections are chosen (and the headrest/footrest finish too, if that
  add-on is switched on). It opens an order-review screen with the full
  breakdown and a **Save as PDF** button that renders the current preview
  image plus the full selection list to a PDF, entirely client-side.

## Updating an asset

Every layer must stay pixel-aligned to the same 1591×1519 canvas. If the
studio sends a fresh export, it'll be on their original 2752×1536 canvas —
crop it to the box `(601, 3, 2192, 1522)` (left, top, right, bottom) first,
the same box every current layer was cropped to, then base64-encode the
result and paste it into the matching entry in the `LAYER_SRC` object in
`index.html`. If a new zone's content falls outside that box, the box
needs to grow to fit it — on every layer, `#seat-canvas`'s `aspect-ratio`,
and the JS `LAYER_SRC` loader all assume one shared canvas size, and a
mismatch between layers will visibly misalign them. Vinyl/Platilon/
Airmesh/Leather/Wrapping swatches live in the `VINYL` / `PLATILON` /
`AIRMESH` / `LEATHER` / `WRAPPING` arrays near the top of the inline
script; which materials apply to which zone is driven by
`SHELL_MATERIALS` (used by every shell-type zone via
`buildShellMaterials()`) and the individual `buildShellTab` /
`buildHeadrestFinish` / `buildFootrestFinish` / `buildAccordion`
functions. A `WRAPPING` entry's `img` has no particular size requirement
(`canvas`'s pattern tiling doesn't care) but must be seamless — a plain
crop will show a visible seam/gap at every repeat — so build a new one
the same way as the current eleven: crop a clean, label-free square,
then 2×2 mirror-tile it (original + h-flip + v-flip + both-flip) before
base64-encoding it into a new array entry with the pattern's real
reference code.

## Fonts

- Body text uses **Noto Sans**, loaded from Google Fonts (falls back to
  the system sans-serif if offline).
- The title ("Customize your OMS seating.") uses the licensed **Bogue
  Semibold Italic**, embedded in the `@font-face` rule in `index.html`
  (same asset used by the AFO, Cheneau, Janton and Supportec
  configurators).
