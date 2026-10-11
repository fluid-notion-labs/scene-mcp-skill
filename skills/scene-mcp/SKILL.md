---
name: scene-mcp
description: Build and edit 3D scenes, architectural models, and diagrams through the scene-mcp MCP server — parametric primitives with real dimensions, part metadata, materials, stable IDs, GLB import/export, and a live browser viewer the agent can observe. Use when authoring 3D content, creating architecture diagrams in 3D, generating massing models from photos, or debugging why a scene doesn't look right in the viewer.
license: MIT
metadata:
  author: fluid-notion-labs
  version: "0.2.0"
  domain: 3d
  triggers: scene-mcp, 3D scene, GLB, glTF, architectural model, 3D diagram, massing, bill of materials, photo camera, viewer
  role: specialist
  scope: implementation
  output-format: code
  related-skills: mcp-developer
---

# scene-mcp

Build 3D scenes through typed MCP tools: groups, parametric primitives with
real dimensions, part metadata, and a live browser viewer that updates as you
work. Everything is a GLB tree — portable, inspectable, round-trip safe.

## Core Workflow

1. **Start** — `scene_new {path}` for a fresh scene, or `scene_open {path}` to
   continue one. If `scene-api` is running, every change live-updates the
   browser viewer (set up before starting, see references/viewer.md).
2. **Structure first** — `create_group` for assemblies (walls, floors,
   diagram groups); build inside them. Name things properly — names show in
   the viewer's scene tree and hover tooltips.
3. **Add geometry** — `add_primitive` (box/cylinder/plane, metres, Y-up).
   Dimensions are baked into the mesh: keep node scale at identity on
   parametric nodes (the server rejects scale on them — resize via
   dimensions instead). Groups and imported models may be scaled.
4. **Tag everything** — `set_part_info`: `part_id` (stable identity),
   `label` (short text rendered on the part in the viewer — diagrams need
   this), `material` (kind + spec), `notes` (hover detail). See
   references/tools.md for the full field list.
5. **Connect things** — for diagrams, edges are thin bars with a rotation:
   compute the quaternion for the direction, add a box sized to the span.
   See the diagram pattern in references/patterns.md.
6. **Verify** — `get_tree` / `get_node` for structure, `bill_of_materials`
   for parts, and the viewer for looks: `get_viewer_logs` shows what the
   browser saw (SSE connects, GLB fetches, errors). If a photo of the real
   site exists, `get_viewer_photos` retrieves it.
7. **Persist** — `export_glb` / `scene_save`. GLB is the only contract.

## Rules that keep you out of trouble

- **Metres, Y-up.** All positions, dimensions, and camera poses.
- **Labels are data, not textures.** Set `label` in part info; the viewer
  renders it on the part's largest face. Don't build text from geometry.
- **Never assume scale on generated models.** Foreign/generated GLBs come
  in at arbitrary size. Measure with `get_node` (mesh summary gives
  vertex/triangle counts; use `scene-gen inspect <file.glb>` for physical
  dimensions) and rescale to a known real-world measurement.
- **The viewer is your eyes.** After nontrivial changes, check
  `get_viewer_logs` for errors, and ask the user to look. Photos uploaded
  by the user (`get_viewer_photos`) show what the site actually looks like;
  `set_photo_camera` lets you guess where each photo was taken from.
- **Group-and-tag beats one-off geometry.** Repeated elements (studs,
  windows) become named nodes with the same `part_id` so the BOM aggregates.

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Full tool surface | `references/tools.md` | Exact parameters, all 25+ tools |
| Diagram authoring | `references/patterns.md` | Flowcharts, architecture maps, layout math |
| Viewer & live sync | `references/viewer.md` | scene-api, SSE, logs, photos, cameras |
| Generating from photos | `references/generation.md` | scene-gen CLI, Meshy/Tripo, scale normalization |
| Materials & BOM | `references/materials-bom.md` | Material kinds, part info, aggregation |
