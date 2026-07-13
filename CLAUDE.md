# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Generative artwork generator for the UM Institute for Mental Fitness ("Generative Brains") — a FinalFinal client project. It produces grid-based blob patterns in the style of `braingrid.png`: circles on a regular grid that fuse into connected, neuron-like clusters. `examples.png` shows the broader aesthetic family (Interface festival-style marks). `PHILOSOPHY.md` documents the design intent ("Synaptic Tessellation").

## Running

No build, no dependencies. `index.html` is fully self-contained (p5.js from CDN):

```
open index.html
```

The tool is also published via GitHub Pages at https://finalfinalff.github.io/generative-brains/ (pushing to `main` deploys).

`?seed=N` in the URL loads a specific variation. Same seed + same parameters always reproduces the identical composition. Export via Download PNG (canvas renders at pixelDensity 2, so exports are 2400px wide) or Download SVG — true vector output from `buildSVG()`, which shares `blobGeometry()` with the canvas renderer. Each blob is flattened into a single compound `<path>` (holes included) by boolean ops in paper.js (`blobToSinglePathD`: union ink shapes, subtract cut circles; arcs become beziers). If paper.js fails to load, export falls back to grouped shapes with a per-blob `<mask>`.

## Architecture

Everything lives in the inline `<script>` of `index.html`, in two stages:

**1. Blob growth (`buildGrid`)** — a `rows × cols` grid of blob IDs (0 = empty). Distinct objects are grown one at a time from random empty cells via seeded random walks (size uniform 1..`blobSize`) until `density` of the grid is occupied. Each growth step extends from the walk tip with probability `snakiness` (snaky tubes) or from any cell of the blob (clumpy masses), and moves diagonally with probability `diagonal` (producing skinny-neck fusions). Diagonal steps are refused when the two between-cells hold the same other blob (`emptyDiagonals`), which would create two necks crossing at one corner. Different blobs freely occupy adjacent cells — this is what makes the composition read as many packed objects rather than one merged mass, since fusion requires a matching blob ID. Seeded through p5's `randomSeed`; every parameter change re-runs `initializeSystem()` so results stay deterministic.

**Shape guide (`sampleGuideField`/`cellWeight`/`weightedPick`)** — an optional uploaded image (Shape Guide section) acts as a density field, not a hard mask: the image is contain-fit downsampled to one darkness value per cell (`guideField`), auto-levels stretched to the full 0..1 range (so a non-black subject can still reach full density). Downsampling is feature-preserving, not mean-luminance — plain averaging can't see features narrower than a cell, so coarse grids would paint over them. Each cell is supersampled 4×4 and three rules apply, in order: (1) **interior light** — light that can't be reached from the image border through comfortably-wide light (erode light mask by ~1.25 cells → flood-fill from border → dilate back; whatever light remains is interior to the artwork) zeroes any cell it covers ≥20% of, making channels/pools/holes inviolable at any grid resolution while background-facing silhouette cells keep soft mean weights (the ragged organic edge); (2) **line crossings** — a cell substantially covered by a tone that also flanks it on both sides of an axis (left+right or top+bottom neighbor means) snaps to that tone, which is what lets dark line art on white paper stay inked (neighborhood-average tests fail at junctions, where context stops being one-sided — hence the flank test); (3) otherwise the cell keeps its mean. Fine grids (35+ rows) barely exercise these rules: channels there are ≥1 cell wide and go fully white, which the weight floor forbids on its own. Every random cell choice in `buildGrid` (blob seeding and each growth step) is a weighted pick, `weight = (1 − guideInfluence × (1 − darkness))³`, snapped to 0 below 0.02 — dark areas grow dense, mid-tones sparse, effectively-white cells forbidden. The hard floor matters: picks are relative among options, so without it a blob whose dark neighborhood is full would spill into white cells as the least-bad option (ink outside the shape, budget stolen from inside). Forbidden cells also suppress diagonal necks — `emptyDiagonals` refuses growth steps and `blobGeometry` skips necks whose between-cells include a zero-weight cell — because neck ink renders across the between-cells and would bridge over a protected white channel (a blob can wrap into diagonal self-adjacency without ever stepping diagonally, so the geometry check is required, not just the growth check). Density budgeting is in weight units on both sides: the target is `density × Σweights`, and a filled cell consumes its own weight (not 1.0) — so low-weight edge cells can't drain budget contributed by the dark interior, and Density means "how full the dark region gets" at any grid size. The random-consumption order in `buildGrid` is load-bearing: with no guide loaded, `weightedPick` consumes exactly one `random()` and picks uniformly, and the size draw stays before the seed pick — this keeps all pre-guide seeds reproducing their original compositions. `guideImg`/`onionOpacity` are runtime-only state (not in `params`, not serialized); renderer and SVG export are guide-agnostic since the guide only shapes the grid. The Onion Skin slider overlays the guide image (`#onion-skin`, a DOM `<img>` positioned by `updateOnionSkin` at the sampler's exact contain-fit placement) for visual registration — it never appears in PNG/SVG exports.

**2. Renderer (`render`)** — composites one offscreen layer per blob onto the canvas. Per blob:

1. Each of its cells is a rect that extends to the cell boundary on sides where the neighbor is the *same blob* (fusing contours; overlapped by 0.5px to prevent antialiasing seams) and is inset by the gap `g` otherwise — neighboring objects sit tangent with a `2g` channel between them. A corner gets radius `R` **only when both of its orthogonal neighbors are not-this-blob** — this alone makes isolated cells circles, runs capsules, and bend shoulders round.
2. Diagonal necks (`drawNeck`): for every same-blob pair touching only diagonally (neither between-cell is the same blob, and the between-cells aren't one shared other blob), the two circles fuse via a metaball pinch — an ink quad between the tangent chords, minus two side "fillet circles" externally tangent to both main circles, sized so the waist half-width is `neckWidth × (s/2 − g)`. The erase is clipped to the quad because the fillet circles are large (often > half a cell) and would otherwise nick nearby ink.
3. Concave fillets: for every not-this-blob cell corner wrapped by three same-blob neighbors (both orthogonal + the diagonal), an ink square of size `rf` goes into that corner, then quarter-circles (radius `rf`, centered `rf` diagonally inward) are removed with `layer.erase()`. Order is load-bearing: rects → necks → all squares → all cuts; overlapping cuts form the cusps between arms and the concave holes inside enclosed cells.

The per-blob layering (not a single canvas) is required: fillet cuts would otherwise erase the ink of a different object sitting in the wrapped cell — e.g., a dot nested in another blob's notch.

Key geometry: arm width = `s − 2g`; max convex radius `R = s/2 − g`; max fillet radius `rf = s/2 + g` (at which enclosed single-cell holes become perfect circles — the default 0.65 gives concave-diamond holes).

**Color (`assignBlobColors`)** — `params.palette` holds the selected swatch colors (empty ⇒ every blob uses `inkColor` on `paperColor`; swatches matching `paperColor` are filtered out). With a multi-color palette, blobs are greedy graph-colored so no two touching blobs (orthogonal *or* diagonal adjacency) share a color: each blob's starting palette index comes from `blobHue[id]`, a seeded random fixed at growth time — so per-blob colors are deterministic for a given seed, and reordering swatches doesn't reshuffle the composition. Color controls live in their own swatch UI (`renderSwatches`/`setPaperSwatch`/`swapColors`), not the slider system.

## Conventions

- The sidebar UI (seed navigation, sliders, Anthropic-branded styling) comes from the `algorithmic-art` skill template — keep its structure when adding parameters: a `control-group` div, an entry in `params`/`defaultParams`, and the `resetParameters` sync list.
- `cols`/`rows`/`seed`/`blobSize` are the integer params (`intParams`); other slider params format to 2 decimals. `palette`/`inkColor`/`paperColor` are not sliders and bypass that machinery.
- Canvas width is fixed at 1200; height derives from rows/cols so cells stay square.
