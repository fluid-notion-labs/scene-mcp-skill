# Authoring patterns

## Layout discipline

Pick a convention before placing anything and state it in node names:

- **Architecture**: building footprint on XZ, Y up; wall runs named
  `Wall North`, `Wall West`…; storeys as groups (`Level 0`, `Level 1`).
- **Diagrams**: a vertical wall in the XY plane (thin depth on Z) reads best
  from the default camera; flow left→right in columns, top→bottom in rows.
  Put all diagram nodes at the same Z so nothing occludes.

Spacing: leave ≥0.6 m (or ~25% of node size) between nodes — labels and
hover targets need room.

## Diagram node

```
add_primitive {name:"cx (CLI)", kind:"box", dimensions:[2.6, 1.2, 0.12],
               position:[-3.8, 2.0, 0]}
set_part_info {id, part_id:"diag.cx-cli", label:"cx", category:"diagram-node",
               material:{kind:"timber", spec:"diagram: contextual pool"},
               notes:"what this service does"}
```

- Thin boxes (`depth` 0.12) so the largest face carries the label.
- Colour-code by category using `material.kind` — the viewer tints each
  family differently (glass = pale, timber = warm, masonry = terracotta,
  membrane = dark, metal-sheet = blue-grey). Use `membrane` for
  planned/not-yet-built items and say so in the label, e.g. `devforge (planned)`.
- Labels render on the part's largest face: for a wall-style diagram that is
  the face toward the camera. Text is dark on light materials, yellow on dark.

## Diagram edge (arrow)

An edge is a thin box oriented along the direction between two nodes:

1. `dx = x2-x1`, `dy = y2-y1`, `len = sqrt(dx²+dy²) - shrink` (shrink ≈ node
   half-widths + gap, typically 1.2–1.7 each side).
2. Dimensions `[len, 0.09, 0.05]` (thin in Z so it doesn't pop out of the wall).
3. Position: midpoint of A and B.
4. Rotation: pure roll about Z, since the bar lies in the XY plane:
   `angle = atan2(dy, dx)`, quaternion `[0, 0, sin(angle/2), cos(angle/2)]`.
5. `set_part_info` with `label` naming the relationship ("MCP stdio",
   "→ structured result"). Direction goes in the label text until edge
   arrowheads land in the tool surface.

For edges in the XZ plane (floor plans), rotate about Y instead:
quaternion `[0, sin(-angle/2), 0, cos(angle/2)]` with `angle = atan2(dz, dx)`.

## Repetition

Repeated elements (studs at 450 centres, window mullions) are best added as
a loop: same `part_id`, computed positions, under one group. The BOM then
reports quantity correctly and `delete`/`reparent` on the group moves them
all.

## Verifying as you go

- After each batch of changes: `get_tree {depth: 2}` to confirm structure.
- Before declaring done: `bill_of_materials` to confirm part aggregation.
- Visually: check `get_viewer_logs` for `SSE changed` → refetch → no parse
  errors; ask the user to look at the viewer. Do not iterate blind.
