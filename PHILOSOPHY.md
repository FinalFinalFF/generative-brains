# Synaptic Tessellation

*An algorithmic philosophy for the Generative Brains project — UM Institute for Mental Fitness*

## The Movement

Synaptic Tessellation is the aesthetics of connection rendered discrete. It begins with the humblest of structures — a rectangular grid of cells, each one a binary decision: present or absent, firing or silent. From this austere substrate, richness emerges not from the cells themselves but from their *adjacencies*. When two occupied cells touch, they do not merely sit side by side; they fuse, their boundaries dissolving into a single continuous contour. When an occupied cell stands alone, it resolves into a perfect circle — a neuron waiting for its network. The artwork is never drawn; it is *negotiated*, cell by cell, between presence and absence.

The mathematics of the movement lives in the corner. Every convex corner of the emergent form is rounded to the full half-width of its cell, so that isolated cells become circles and linear runs become capsules. Every concave corner — the inner elbow where a form bends around an empty cell — receives a fillet: a quarter-arc of negative space carved back into the ink. It is this concave fillet, tuned with painstaking care, that transforms a crude pixel-map into something organic, tensile, almost wet. Where four arms of ink enclose a single empty cell, the four fillets meet and the void crystallizes into a four-pointed star — an astroid of breath inside the mass. These cusps and stars are not decorations; they are the inevitable consequence of a geometry meticulously derived, the kind of construction that reads as effortless precisely because every radius was reasoned through by a master of the craft.

Occupancy is a population, not a field. The composition is grown one organism at a time: each blob begins as a single cell dropped into open space and extends itself by seeded random walk — sometimes snaking from its tip like a growing axon, sometimes thickening from its body like a soma — until it reaches its allotted size. Crucially, each organism carries an identity, and only cells of the same identity fuse. Different organisms may press into adjacent cells, tangent and jostling, yet they never merge; the grid fills toward its target density as a crowd of distinct individuals, not a single coalescing mass. The balance between snaking and clumping is the composition's temperament: all tube and it turns wiry; all mass and it turns bulbous. The mastery is in the blend, refined through countless iterations until every seed yields a population that feels deliberate.

The conceptual seed is neural. These are synapses in schematic: nodes, processes, junctions, and the essential gaps between them — because in a brain, as in this geometry, the gap is functional. The synaptic cleft is where the signal happens. So the algorithm honors its gaps: unconnected forms never quite touch, separated by a thin channel of ground that gives the composition its electric tension. Someone who has stared at neuronal micrographs will feel the reference in their spine; everyone else will simply see forms that seem alive.

Each seed is a different brain. The algorithm is fixed — a single meticulously crafted system, the product of deep computational expertise — but the seed space is infinite, and every integer summons a new topology of connection and isolation. This is the movement's final commitment: process over product, system over artifact. The designer does not compose the artwork; the designer tunes the physics and then explores the space of minds it can generate.

## Algorithmic Commitments

1. **Grid as substrate** — a boolean occupancy field on a regular grid; all form derives from adjacency.
2. **Corner calculus** — per-corner convex radii (full half-width when both orthogonal neighbors are empty) and concave fillets (where an empty cell's corner is wrapped by three occupied neighbors).
3. **Grown, not thresholded** — each blob is a distinct organism grown by seeded random walk; identity determines fusion, so packed neighbors stay separate objects.
4. **The sacred gap** — disconnected forms are inset from their cell boundaries; connection closes the gap, isolation preserves it.
5. **Seeded determinism** — identical seed and parameters must always reproduce the identical composition.
