# Materials and bills of materials

## Material vocabulary

`material = {kind, spec, treatment?}`:

- **kind** — one of the family enum: `timber`, `steel`, `metal-sheet`,
  `fibre-cement`, `plasterboard`, `wood-panel`, `insulation`, `membrane`,
  `concrete`, `masonry`, `aac`, `plastic`, `glass`, `other`. The kind is
  also the viewer's tint (each family renders a distinct muted color).
- **spec** — the as-sold designation, e.g. `MGP10`, `0.42 BMT Colorbond`,
  `R2.5 glasswool`, or for diagrams a free tag like `diagram: contextual pool`.
- **treatment** — finish, e.g. `H2F`, `galvanised`, `fire-rated`.

Choose the kind by what the thing *is*, not how it should look — the viewer
palette is derived from it. For diagram categories, pick the family whose
tint reads best and say so in the spec.

## Parts and the BOM

`bill_of_materials` aggregates by `part_id` over a subtree (or the whole
scene): total quantity (summed from per-node `quantity`), unit, material
string, node count. Two rules make it work:

1. **One `part_id` per real-world part type.** Fifty studs all get
   `stud-90x45-mgp10`; the BOM reports quantity 50, not 50 lines.
2. **`quantity` is per-node count.** A node representing 3 identical items
   has `quantity: 3`. Defaults to 1.

`nominal_dimensions` and `unit` (`each`, `lm`, `m2`, `sheet`) make the BOM
purchasable rather than merely countable.

## Round-trip guarantees

Part info (including `label`) and stable ids live in node `extras`, so an
export → import round trip preserves names, hierarchy, parts, ids, and
parametric dimensions. Foreign GLBs import as opaque meshes: geometry and
hierarchy survive; their materials/textures do not — re-tag with part info.
