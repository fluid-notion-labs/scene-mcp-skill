# Viewer, live sync, and observation

## The moving parts

- **scene-mcp** (stdio MCP server) — holds the authoritative scene. With
  `SCENE_API_URL` set (e.g. `http://127.0.0.1:8787/api/scene`), every
  mutating call pushes the scene to scene-api. Without it, pushes are
  skipped harmlessly.
- **scene-api** — HTTP server on `127.0.0.1:8787`: serves the current scene
  as GLB, SSE events, photo upload, log relay, and the viewer itself at
  `/viewer/`. Start it with `cargo run -p scene-api`.
- **viewer** — static three.js page; "Live" mode subscribes to SSE and
  refetches the GLB on every change.

If scene-api isn't running, tell the user: they will see nothing live.

## The observation loop

The browser's console is relayed to scene-api and readable from your side:

- `get_viewer_logs` — returns recent entries: `SSE connect`, `SSE changed
  (version N) → refetching GLB`, `live scene — N root node(s)`, parse
  errors, photo uploads. If the user reports "nothing happened", check
  these before anything else.
- Chain to check: MCP push → `scene replaced` (api log) → `SSE changed`
  (browser) → `scene.glb served` → parse. A break anywhere shows in
  `get_viewer_logs` or scene-api's stderr.

## Photos

The user can upload reference photos in the viewer (📷 button, or drag & drop
images). These are how you see the real site:

1. `get_viewer_photos` — list (id, name, size).
2. `get_viewer_photo {id}` — the actual image; study it.
3. `set_photo_camera {id, position, target, fov?}` — your guess of where the
   photo was taken from. It renders as a camera frustum with the photo
   attached, so the user can verify your guess visually. Iterating on this
   is how you align the model with reality.

## What the user sees

- Scene tree panel (bottom-left): your node names, part labels, group
  hierarchy. Meaningful names are visible UI, not internal detail.
- Settings panel (☰): ambient light slider, directional toggle.
- Info panel: clicking a part shows name, `scene_id`, `part_id`, label,
  notes.
- Videos panel: user can upload videos; use them with `scene-frames`
  (see generation.md) for reconstruction.
