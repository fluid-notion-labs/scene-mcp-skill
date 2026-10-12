# Generating models from photos (scene-gen)

The `scene-gen` CLI turns a few photos of a real object or building into a
textured **massing model** — shape and surface, but no metric scale and no
measurable geometry. It is the fast alternative to SfM/photogrammetry
(`scene-sfm`), which gives measured poses and points but takes much longer.

## What it's good for

- Massing / context models: "a building roughly like this, at roughly the
  right size" placed in a scene.
- Props and set dressing.
- NOT for: construction-grade dimensions, as-built documentation, anything
  that must be measured. Use real measurements and parametric primitives
  for that, or SfM for measured reconstruction.

## Prep the photo first

Raw photos usually have clutter (cars, bins, vegetation) and excess margin.
Use the MCP prep tools before generating — see references/tools.md
("Image prep") for parameters:

```
prep_image {image: "site.jpg", target: "the building",
            remove: ["car", "rubbish bin"], output: "tmp/prep"}
```

Then generate from the cropped result: `scene-gen single tmp/prep/site-prep.jpg`.
For manual control, `detect_objects` gives bounding boxes and `inpaint_image`
removes regions from a hand-built mask.

## Usage

```bash
# backend auto-picked from available keys (TRIPO_API_KEY, MESHY_API_KEY)
scene-gen probe

# single photo
scene-gen single photo.jpg --out building.glb

# multi-view (2-4 images; semantic order front, left, back, right)
scene-gen massing front.jpg left.jpg back.jpg right.jpg --out building.glb

# with a known real-world measurement — reports the scale to apply
scene-gen inspect building.glb --footprint 10x12
```

Keys come from `--api-key`, the environment, or `env/.env` in the project.

## Scale normalization

Generated models have arbitrary size. With one known measurement (a wall
length, the building footprint), `--footprint WxD` computes non-uniform
scale factors `[sx, sy, sz]` that make the model's X/Z footprint match
reality (height scales by the geometric mean of the two factors). Apply
after import:

```
import_glb {path}
set_transform {id, scale: [sx, sy, sz]}
```

For a boxy building this is surprisingly accurate; for organic shapes use a
uniform scale from the most reliable measurement.

## Pipeline with video

For a walk-around video, extract frames first (scene-frames), pick the
sharpest, widest-separated views, and feed those:

```bash
cargo run -p scene-frames -- export video.mp4 frames/ --keep 12   # blur-ranked
scene-gen massing frames/f001.jpg frames/f030.jpg frames/f060.jpg --out massing.glb
```

## Cost and backends

- **Meshy**: ~30 credits per textured model (~$0.60 at Pro rates); inline
  base64 images; endpoint `/openapi/v1/{,multi-}image-to-3d`.
- **Tripo V3**: ~20–30 credits (~$0.20–0.30); file upload + multiview with
  4 slots [front, left, back, right]; V2 retired Nov 2026.
- **Self-hosted** (Hunyuan3D, TRELLIS on RunPod): planned backend, zero
  marginal cost.

Generated GLBs import as opaque geometry; textures are not preserved through
the scene pipeline — set part-info materials for consistent tinting.
