# OMS-Configurator

Custom seating OMS configurator — a single, responsive page (`index.html`)
that works on both desktop and mobile, no separate mobile link required
(`mobile.html` just redirects to `index.html` for old links).

`index.html` is fully self-contained: every layer image, all vinyl /
airmesh / platilon / leather swatches, the vendored jsPDF library, the
Eqwal Ability logo/circle backdrop and the title font are embedded
directly in the file as data URIs. You can open it straight from disk
(double-click it, or drag it into a browser tab) with nothing else
alongside it, or serve it from any static file host / GitHub Pages —
both work identically.

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
  - **Shell** (Frontal panel, Outside shell) — the two most visible outer
    surfaces — is **Vinyl only**, same 15-color palette as Supportec, no
    sub-tabs.
  - **Headrest shell** and **Footrest shell** are also **Vinyl only**.
  - **Headrest upholstery** (the inside) offers **Airmesh, Platilon or
    Leather** — no Vinyl — as three sub-tabs of colors (the
    `.subtabs`/`.sub-pane` pattern also used by Footrest upholstery).
  - **Footrest upholstery** offers **Vinyl or Bare foam** (no cover,
    natural finish) — the same "Bare foam" concept as Supportec, paired
    with Vinyl instead of Platilon/Airmesh.
  - **Upholstery tab** — Back, Back lateral, Seating, Abduction wedge —
    each offers **Leather, Platilon or Airmesh**, no Vinyl, as three
    sub-tabs of colors per zone (instead of Supportec's stacked
    Platilon/Airmesh sections).
- Vinyl (15), Platilon (4) and Airmesh (7) swatches are the exact same
  palette as Supportec/Janton (same names, codes and hex values — this
  is shared Eqwal Ability material data, not re-derived). **Leather (18)**
  is new for OMS: names and codes come from the "Eqwal Ability - Leather"
  reference sheet; hex values were sampled from the swatch photos on that
  sheet, the same approach used for the other three material arrays.
- Tabs: **Shell** (Frontal panel + Outside shell, Vinyl only) →
  **Headrest** (optional, toggle switch; Vinyl shell + Airmesh/Platilon/
  Leather upholstery) → **Footrest** (optional, toggle switch; Vinyl
  shell + Vinyl/Bare foam upholstery) → **Upholstery** (four collapsible
  sections — Back, Back lateral, Seating, Abduction wedge — each
  Leather/Platilon/Airmesh).
- **View order** unlocks once both shell panels and all four upholstery
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
Airmesh/Leather swatches live in the `VINYL` / `PLATILON` / `AIRMESH` /
`LEATHER` arrays near the top of the inline script; which materials apply
to which zone is driven by the individual `buildShellTab` /
`buildHeadrestFinish` / `buildFootrestFinish` / `buildAccordion`
functions.

## Fonts

- Body text uses **Noto Sans**, loaded from Google Fonts (falls back to
  the system sans-serif if offline).
- The title ("Customize your OMS seating.") uses the licensed **Bogue
  Semibold Italic**, embedded in the `@font-face` rule in `index.html`
  (same asset used by the AFO, Cheneau, Janton and Supportec
  configurators).
