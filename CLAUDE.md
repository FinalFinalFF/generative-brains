# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Generative artwork generator for the UM Institute for Mental Fitness ("Generative Brains") — a FinalFinal client project. It produces grid-based blob patterns in the style of `braingrid.png`: circles on a regular grid that fuse into connected, neuron-like clusters. `examples.png` shows the broader aesthetic family (Interface festival-style marks). `PHILOSOPHY.md` documents the design intent ("Synaptic Tessellation").

## Running

No build, no dependencies. `index.html` is fully self-contained (p5.js from CDN):

```
open index.html
```

`?seed=N` in the URL loads a specific variation. Same seed + same parameters always reproduces the identical composition. Export via Download PNG (canvas renders at pixelDensity 2, so exports are 2400px wide) or Download SVG — true vector output from `buildSVG()`, which shares `blobGeometry()` with the canvas renderer: cell rects and diagonal necks are exact paths, and fillet cut circles become a per-blob `<mask>` so they cross-cut fillet squares exactly like the canvas erase pass.

## Architecture

Everything lives in the inline `<script>` of `index.html`, in two stages:

**1. Blob growth (`buildGrid`)** — a `rows × cols` grid of blob IDs (0 = empty). Distinct objects are grown one at a time from random empty cells via seeded random walks (size uniform 1..`blobSize`) until `density` of the grid is occupied. Each growth step extends from the walk tip with probability `snakiness` (snaky tubes) or from any cell of the blob (clumpy masses), and moves diagonally with probability `diagonal` (producing skinny-neck fusions). Diagonal steps are refused when the two between-cells hold the same other blob (`emptyDiagonals`), which would create two necks crossing at one corner. Different blobs freely occupy adjacent cells — this is what makes the composition read as many packed objects rather than one merged mass, since fusion requires a matching blob ID. Seeded through p5's `randomSeed`; every parameter change re-runs `initializeSystem()` so results stay deterministic.

**2. Renderer (`render`)** — composites one offscreen layer per blob onto the canvas. Per blob:

1. Each of its cells is a rect that extends to the cell boundary on sides where the neighbor is the *same blob* (fusing contours; overlapped by 0.5px to prevent antialiasing seams) and is inset by the gap `g` otherwise — neighboring objects sit tangent with a `2g` channel between them. A corner gets radius `R` **only when both of its orthogonal neighbors are not-this-blob** — this alone makes isolated cells circles, runs capsules, and bend shoulders round.
2. Diagonal necks (`drawNeck`): for every same-blob pair touching only diagonally (neither between-cell is the same blob, and the between-cells aren't one shared other blob), the two circles fuse via a metaball pinch — an ink quad between the tangent chords, minus two side "fillet circles" externally tangent to both main circles, sized so the waist half-width is `neckWidth × (s/2 − g)`. The erase is clipped to the quad because the fillet circles are large (often > half a cell) and would otherwise nick nearby ink.
3. Concave fillets: for every not-this-blob cell corner wrapped by three same-blob neighbors (both orthogonal + the diagonal), an ink square of size `rf` goes into that corner, then quarter-circles (radius `rf`, centered `rf` diagonally inward) are removed with `layer.erase()`. Order is load-bearing: rects → necks → all squares → all cuts; overlapping cuts form the cusps between arms and the concave holes inside enclosed cells.

The per-blob layering (not a single canvas) is required: fillet cuts would otherwise erase the ink of a different object sitting in the wrapped cell — e.g., a dot nested in another blob's notch.

Key geometry: arm width = `s − 2g`; max convex radius `R = s/2 − g`; max fillet radius `rf = s/2 + g` (at which enclosed single-cell holes become perfect circles — the default 0.65 gives concave-diamond holes).

## Conventions

- The sidebar UI (seed navigation, sliders, Anthropic-branded styling) comes from the `algorithmic-art` skill template — keep its structure when adding parameters: a `control-group` div, an entry in `params`/`defaultParams`, and the `resetParameters` sync list.
- `cols`/`rows`/`seed` are the only integer params (`intParams`); everything else formats to 2 decimals.
- Canvas width is fixed at 1200; height derives from rows/cols so cells stay square.
